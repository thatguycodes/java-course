# Contract Testing

[← Back to README](../README.md)

---

**Contract testing** verifies that two services agree on an API contract — independently, without running both services at the same time. The consumer records what it expects; the provider verifies it can satisfy those expectations. This catches integration breakage before it reaches a staging environment.

```mermaid
flowchart LR
    consumer["Consumer\n(e.g. Order Service)"]
    pact["Pact Broker /\nPactFlow"]
    provider["Provider\n(e.g. Product Service)"]

    consumer -->|1 generate pact| pact
    pact -->|2 publish contract| provider
    provider -->|3 verify against running app| pact
    pact -->|4 can-i-deploy result| consumer & provider
```

---

## Pact (Consumer-Driven Contract Testing)

### Maven Dependencies

```xml
<!-- Consumer -->
<dependency>
    <groupId>au.com.dius.pact.consumer</groupId>
    <artifactId>junit5</artifactId>
    <version>4.6.14</version>
    <scope>test</scope>
</dependency>

<!-- Provider -->
<dependency>
    <groupId>au.com.dius.pact.provider</groupId>
    <artifactId>junit5spring</artifactId>
    <version>4.6.14</version>
    <scope>test</scope>
</dependency>
```

---

## Consumer Side — Writing the Contract

The consumer test defines what it sends and what it expects back. Pact records the interaction as a JSON pact file.

```java
// src/test/java/com/example/order/ProductClientPactTest.java
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "product-service", port = "8080")
class ProductClientPactTest {

    @Pact(consumer = "order-service")
    public RequestResponsePact getProductById(PactDslWithProvider builder) {
        return builder
            .given("product 42 exists")
            .uponReceiving("a request for product 42")
                .path("/api/products/42")
                .method("GET")
            .willRespondWith()
                .status(200)
                .headers(Map.of("Content-Type", "application/json"))
                .body(new PactDslJsonBody()
                    .numberType("id", 42)
                    .stringType("name", "Widget")
                    .decimalType("price", 9.99)
                    .booleanType("inStock", true))
            .toPact();
    }

    @Test
    @PactTestFor(pactMethod = "getProductById")
    void shouldFetchProduct(MockServer mockServer) {
        // point your HTTP client at the mock
        ProductClient client = new ProductClient(mockServer.getUrl());

        Product product = client.getById(42L);

        assertThat(product.id()).isEqualTo(42L);
        assertThat(product.name()).isEqualTo("Widget");
        assertThat(product.price()).isEqualByComparingTo("9.99");
    }
}
```

Running the test creates `target/pacts/order-service-product-service.json`.

### Matching Rules

```java
new PactDslJsonBody()
    .numberType("id")               // any number
    .stringType("name")             // any string
    .stringMatcher("email",
        "\\w+@\\w+\\.\\w+",
        "alice@example.com")        // regex match
    .decimalType("price")           // any decimal
    .datetime("createdAt",
        "yyyy-MM-dd'T'HH:mm:ss")   // ISO datetime
    .minArrayLike("tags", 1,        // array with at least 1 element
        new PactDslJsonBody()
            .stringType("label"))
```

### POST / Request Body

```java
@Pact(consumer = "order-service")
public RequestResponsePact createOrder(PactDslWithProvider builder) {
    return builder
        .given("product 42 exists and is in stock")
        .uponReceiving("a request to create an order")
            .path("/api/orders")
            .method("POST")
            .headers(Map.of("Content-Type", "application/json"))
            .body(new PactDslJsonBody()
                .numberType("productId", 42)
                .numberType("quantity", 2))
        .willRespondWith()
            .status(201)
            .headers(Map.of("Content-Type", "application/json"))
            .body(new PactDslJsonBody()
                .numberType("orderId")
                .stringMatcher("status", "PENDING|CONFIRMED", "PENDING"))
        .toPact();
}
```

---

## Provider Side — Verifying the Contract

The provider test loads the pact file and replays each interaction against the running app.

```java
// src/test/java/com/example/product/ProductServicePactVerificationTest.java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Provider("product-service")
@PactFolder("../order-service/target/pacts")  // load pact from consumer
class ProductServicePactVerificationTest {

    @LocalServerPort
    int port;

    @MockBean
    ProductRepository productRepository;

    @BeforeEach
    void setUp(PactVerificationContext context) {
        context.setTarget(new HttpTestTarget("localhost", port));
    }

    @TestTemplate
    @ExtendWith(PactVerificationInvocationContextProvider.class)
    void verifyPact(PactVerificationContext context) {
        context.verifyInteraction();
    }

    // State handlers — set up test data for each provider state
    @State("product 42 exists")
    void productExists() {
        when(productRepository.findById(42L))
            .thenReturn(Optional.of(new Product(42L, "Widget",
                new BigDecimal("9.99"), true)));
    }

    @State("product 42 exists and is in stock")
    void productExistsAndInStock() {
        when(productRepository.findById(42L))
            .thenReturn(Optional.of(new Product(42L, "Widget",
                new BigDecimal("9.99"), true)));
    }
}
```

---

## Pact Broker — Sharing Contracts

The Pact Broker stores and versions pact files so teams don't need to share them via the filesystem.

### Running a Pact Broker Locally

```yaml
# compose.yml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: pact
      POSTGRES_PASSWORD: pact
      POSTGRES_DB: pact
    volumes:
      - pact-db:/var/lib/postgresql/data

  pact-broker:
    image: pactfoundation/pact-broker:latest
    ports:
      - "9292:9292"
    environment:
      PACT_BROKER_DATABASE_URL: postgres://pact:pact@postgres/pact
      PACT_BROKER_LOG_LEVEL: INFO
    depends_on:
      - postgres

volumes:
  pact-db:
```

### Publishing Pacts

```xml
<!-- pom.xml — Pact Maven plugin -->
<plugin>
    <groupId>au.com.dius.pact.provider</groupId>
    <artifactId>maven</artifactId>
    <version>4.6.14</version>
    <configuration>
        <pactBrokerUrl>http://localhost:9292</pactBrokerUrl>
        <pactDirectory>target/pacts</pactDirectory>
        <projectVersion>${project.version}</projectVersion>
        <trimSnapshot>true</trimSnapshot>
    </configuration>
</plugin>
```

```bash
mvn pact:publish
```

### Verifying Against the Broker

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Provider("product-service")
@PactBroker(host = "localhost", port = "9292")  // load from broker
class ProductServicePactVerificationTest {
    // same as before
}
```

### can-i-deploy

```bash
# Check whether it's safe to deploy order-service v1.2.0 to prod
pact-broker can-i-deploy \
  --pacticipant order-service \
  --version 1.2.0 \
  --to-environment production \
  --broker-base-url http://localhost:9292
```

---

## Spring Cloud Contract

Spring Cloud Contract is an alternative that keeps contracts in the provider's repository as Groovy/YAML stubs. The consumer gets a stub JAR it runs locally.

### Provider — Define Contracts

```groovy
// src/test/resources/contracts/product/shouldReturnProduct.groovy
package contracts.product

import org.springframework.cloud.contract.spec.Contract

Contract.make {
    description "should return product by id"
    request {
        method GET()
        url "/api/products/42"
    }
    response {
        status OK()
        headers { contentType applicationJson() }
        body(
            id: 42,
            name: $(anyNonEmptyString()),
            price: $(anyPositiveDouble()),
            inStock: true
        )
    }
}
```

```xml
<!-- Provider pom.xml -->
<plugin>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-maven-plugin</artifactId>
    <version>4.1.4</version>
    <extensions>true</extensions>
    <configuration>
        <testFramework>JUNIT5</testFramework>
        <baseClassForTests>
            com.example.product.BaseContractTest
        </baseClassForTests>
    </configuration>
</plugin>
```

```java
// src/test/java/com/example/product/BaseContractTest.java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public abstract class BaseContractTest {

    @Autowired MockMvc mockMvc;

    @MockBean ProductRepository productRepository;

    @BeforeEach
    void setUp() {
        RestAssuredMockMvc.mockMvc(mockMvc);
        when(productRepository.findById(42L))
            .thenReturn(Optional.of(new Product(42L, "Widget",
                new BigDecimal("9.99"), true)));
    }
}
```

Running `mvn install` on the provider generates a stub JAR:
`product-service-0.0.1-SNAPSHOT-stubs.jar`

### Consumer — Use the Stub

```java
// Consumer test
@SpringBootTest
@AutoConfigureStubRunner(
    ids = "com.example:product-service:+:stubs:8080",
    stubsMode = StubRunnerProperties.StubsMode.LOCAL)
class OrderServiceIntegrationTest {

    @Autowired ProductClient productClient;

    @Test
    void fetchProductFromStub() {
        Product product = productClient.getById(42L);
        assertThat(product.name()).isEqualTo("Widget");
    }
}
```

---

## CI Integration

```yaml
# .github/workflows/contract-tests.yml
name: Contract Tests

on: [push, pull_request]

jobs:
  consumer:
    name: Consumer — generate pacts
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '21', distribution: 'temurin' }
      - run: mvn test -pl order-service
      - uses: actions/upload-artifact@v4
        with:
          name: pacts
          path: order-service/target/pacts/

  provider:
    name: Provider — verify pacts
    runs-on: ubuntu-latest
    needs: consumer
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '21', distribution: 'temurin' }
      - uses: actions/download-artifact@v4
        with: { name: pacts, path: order-service/target/pacts/ }
      - run: mvn test -pl product-service
```

---

## Contract Testing Summary

| Concept | Pact | Spring Cloud Contract |
|---------|------|-----------------------|
| Contract owned by | Consumer | Provider |
| Contract format | JSON pact file | Groovy / YAML |
| Stub delivery | Pact Broker | Maven stub JAR |
| Provider state | `@State` method | `BaseContractTest` setup |
| Deployment check | `can-i-deploy` CLI | N/A |

| Term | Meaning |
|------|---------|
| Consumer | Service that calls the API |
| Provider | Service that implements the API |
| Pact | JSON file recording interactions |
| Provider state | Precondition set up on the provider before verifying |
| can-i-deploy | Check that a version is compatible before releasing |
| Stub | Fake provider that replays recorded responses |

---

[← Back to README](../README.md)
