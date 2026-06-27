# Spring REST Docs

[← Back to README](../README.md)

---

**Spring REST Docs** generates API documentation from your test suite. Unlike Swagger/OpenAPI annotations scattered through production code, REST Docs captures the actual request and response from a running test — the documentation is always accurate because it fails to build if the documented behaviour doesn't match.

```mermaid
flowchart LR
    test["MockMvc / WebTestClient\ntest"]
    snippets["AsciiDoc Snippets\n(curl, request, response,\nrequest-fields, response-fields)"]
    asciidoc["hand-written .adoc\n(imports snippets)"]
    html["HTML / PDF\nAPI Documentation"]

    test -->|document()| snippets
    snippets --> asciidoc
    asciidoc -->|Asciidoctor Maven plugin| html
```

---

## Dependencies

```xml
<dependency>
    <groupId>org.springframework.restdocs</groupId>
    <artifactId>spring-restdocs-mockmvc</artifactId>
    <scope>test</scope>
</dependency>

<!-- Asciidoctor plugin to build HTML docs -->
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>3.0.0</version>
    <executions>
        <execution>
            <id>generate-docs</id>
            <phase>prepare-package</phase>
            <goals><goal>process-asciidoc</goal></goals>
            <configuration>
                <backend>html</backend>
                <doctype>book</doctype>
                <attributes>
                    <snippets>${project.build.directory}/generated-snippets</snippets>
                </attributes>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-asciidoctor</artifactId>
            <version>3.0.1</version>
        </dependency>
    </dependencies>
</plugin>
```

---

## Test Setup

```java
@WebMvcTest(OrderController.class)
@AutoConfigureRestDocs(outputDir = "target/generated-snippets")
class OrderControllerDocTest {

    @Autowired MockMvc mockMvc;
    @MockBean  OrderService orderService;

    @Test
    void documentGetOrder() throws Exception {
        UUID orderId = UUID.fromString("a1b2c3d4-e5f6-7890-abcd-ef1234567890");
        when(orderService.findById(orderId)).thenReturn(
            new OrderResponse(orderId, "CONFIRMED", new BigDecimal("49.99"), "cust-1"));

        mockMvc.perform(get("/api/orders/{id}", orderId)
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andDo(document("orders/get-by-id",          // snippet directory name
                pathParameters(
                    parameterWithName("id").description("The order UUID")),
                responseFields(
                    fieldWithPath("id").description("Unique order identifier"),
                    fieldWithPath("status").description("Order status: PENDING, CONFIRMED, SHIPPED, DELIVERED"),
                    fieldWithPath("total").description("Order total in USD"),
                    fieldWithPath("customerId").description("ID of the customer who placed the order"))));
    }
}
```

This generates snippets in `target/generated-snippets/orders/get-by-id/`:
- `curl-request.adoc`
- `http-request.adoc`
- `http-response.adoc`
- `path-parameters.adoc`
- `response-fields.adoc`

---

## Documenting Requests

```java
@Test
void documentPlaceOrder() throws Exception {
    when(orderService.placeOrder(any())).thenReturn(
        new OrderResponse(UUID.randomUUID(), "PENDING", new BigDecimal("99.99"), "cust-1"));

    mockMvc.perform(post("/api/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "customerId": "cust-1",
                        "items": [
                            { "productId": "prod-1", "quantity": 2, "unitPrice": 49.99 }
                        ]
                    }
                    """))
        .andExpect(status().isCreated())
        .andDo(document("orders/place",
            requestFields(
                fieldWithPath("customerId").description("Customer placing the order"),
                fieldWithPath("items").description("List of line items"),
                fieldWithPath("items[].productId").description("Product identifier"),
                fieldWithPath("items[].quantity").description("Number of units"),
                fieldWithPath("items[].unitPrice").description("Price per unit in USD")),
            responseFields(
                fieldWithPath("id").description("Generated order ID"),
                fieldWithPath("status").description("Initial status (always PENDING)"),
                fieldWithPath("total").description("Computed order total"),
                fieldWithPath("customerId").description("Customer who placed the order"))));
}
```

---

## Query Parameters and Request Headers

```java
@Test
void documentListOrders() throws Exception {
    when(orderService.findByCustomer(any(), any())).thenReturn(List.of());

    mockMvc.perform(get("/api/orders")
                .param("customerId", "cust-1")
                .param("status", "CONFIRMED")
                .param("page", "0")
                .param("size", "20")
                .header("Authorization", "Bearer eyJhbGci..."))
        .andExpect(status().isOk())
        .andDo(document("orders/list",
            queryParameters(
                parameterWithName("customerId").description("Filter by customer"),
                parameterWithName("status").description("Filter by status").optional(),
                parameterWithName("page").description("Page number (0-based)").optional(),
                parameterWithName("size").description("Page size (default 20)").optional()),
            requestHeaders(
                headerWithName("Authorization").description("Bearer JWT access token"))));
}
```

---

## Relaxed vs Strict Documentation

By default, every field in the response must be documented or the test fails.

```java
// Strict — ALL fields must be documented (default)
responseFields(
    fieldWithPath("id").description("..."),
    fieldWithPath("status").description("..."));

// Relaxed — only documented fields are checked; undocumented fields pass
relaxedResponseFields(
    fieldWithPath("id").description("..."));

// Ignore specific fields you don't want to document
responseFields(
    fieldWithPath("id").description("..."),
    fieldWithPath("_links").ignored());
```

---

## Custom Snippets

```java
// Custom snippet — document response headers
mockMvc.perform(post("/api/orders")...)
    .andExpect(status().isCreated())
    .andDo(document("orders/place",
        responseHeaders(
            headerWithName("Location").description("URL of the created order"))));
```

---

## AsciiDoc Source Files

Write a hand-authored AsciiDoc file that imports the generated snippets:

```asciidoc
// src/main/asciidoc/api-guide.adoc
= Order Service API
:toc:
:toclevels: 2

== Place an Order

POST `/api/orders`

=== Request Fields
include::{snippets}/orders/place/request-fields.adoc[]

=== Example Request
include::{snippets}/orders/place/curl-request.adoc[]

include::{snippets}/orders/place/http-request.adoc[]

=== Example Response
include::{snippets}/orders/place/http-response.adoc[]

== Get an Order

GET `/api/orders/{id}`

=== Path Parameters
include::{snippets}/orders/get-by-id/path-parameters.adoc[]

=== Response Fields
include::{snippets}/orders/get-by-id/response-fields.adoc[]

=== Example
include::{snippets}/orders/get-by-id/curl-request.adoc[]
include::{snippets}/orders/get-by-id/http-response.adoc[]
```

After `mvn package`, the HTML appears at `target/generated-docs/api-guide.html`.

---

## Serving Docs from the Application

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/docs/**")
            .addResourceLocations("classpath:/static/docs/");
    }
}
```

```xml
<!-- Copy generated HTML into the JAR -->
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <executions>
        <execution>
            <id>copy-docs</id>
            <phase>prepare-package</phase>
            <goals><goal>copy-resources</goal></goals>
            <configuration>
                <outputDirectory>
                    ${project.build.outputDirectory}/static/docs
                </outputDirectory>
                <resources>
                    <resource>
                        <directory>${project.build.directory}/generated-docs</directory>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Docs are then available at `http://localhost:8080/docs/api-guide.html`.

---

## OpenAPI Integration (springdoc + REST Docs)

Use `restdocs-api-spec` to generate an OpenAPI spec from REST Docs snippets:

```xml
<dependency>
    <groupId>com.epages</groupId>
    <artifactId>restdocs-api-spec-mockmvc</artifactId>
    <version>0.19.2</version>
    <scope>test</scope>
</dependency>
```

```java
import static com.epages.restdocs.apispec.MockMvcRestDocumentationWrapper.document;
import static com.epages.restdocs.apispec.ResourceDocumentation.resource;

mockMvc.perform(get("/api/orders/{id}", orderId))
    .andDo(document("get-order",
        resource(ResourceSnippetParameters.builder()
            .tag("Orders")
            .summary("Get order by ID")
            .pathParameters(parameterWithName("id").description("Order UUID"))
            .responseFields(fieldWithPath("id").description("Order ID"))
            .build())));
```

This generates `openapi3.yaml` that Swagger UI can serve.

---

## Spring REST Docs Summary

| Concept | Detail |
|---------|--------|
| `@AutoConfigureRestDocs` | Wires MockMvc with snippet output directory |
| `document("name", ...)` | Captures snippets for the named operation |
| `pathParameters(...)` | Documents `{variable}` path segments |
| `queryParameters(...)` | Documents `?param=value` query strings |
| `requestFields(...)` | Documents JSON request body fields |
| `responseFields(...)` | Documents JSON response body fields (strict by default) |
| `relaxedResponseFields(...)` | Only documented fields checked; extras pass |
| `requestHeaders(...)` | Documents expected request headers |
| AsciiDoc source | Hand-written doc that `include::` generated snippets |
| Asciidoctor plugin | Converts `.adoc` → `.html` during `prepare-package` |
| vs Swagger | REST Docs fails build on inaccurate docs; Swagger risks drift |

---

[← Back to README](../README.md)
