# HTTP Clients

[← Back to README](../README.md)

---

Spring Boot provides three ways to make HTTP calls to other services: the modern **`RestClient`** (synchronous, fluent, Spring Boot 3.2+), **`WebClient`** (reactive, non-blocking), and declarative **Feign** clients (interface-based, minimal boilerplate). Choose based on whether your stack is blocking or reactive, and how much boilerplate you want to write.

---

## RestClient (Spring Boot 3.2+)

`RestClient` replaces `RestTemplate` with a fluent, readable API. It is synchronous and blocks the calling thread.

### Setup

```java
@Bean
public RestClient orderServiceClient(RestClient.Builder builder) {
    return builder
        .baseUrl("http://order-service")
        .defaultHeader("Accept", "application/json")
        .defaultStatusHandler(
            HttpStatusCode::is4xxClientError,
            (req, res) -> { throw new ClientException(res.getStatusCode()); })
        .defaultStatusHandler(
            HttpStatusCode::is5xxServerError,
            (req, res) -> { throw new ServerException(res.getStatusCode()); })
        .build();
}
```

### GET

```java
@Service
public class OrderClient {

    private final RestClient restClient;

    // Simple object
    public Order getOrder(UUID id) {
        return restClient.get()
            .uri("/api/orders/{id}", id)
            .retrieve()
            .body(Order.class);
    }

    // Generic / parameterized type
    public List<Order> listOrders(String customerId) {
        return restClient.get()
            .uri("/api/orders?customerId={cid}", customerId)
            .retrieve()
            .body(new ParameterizedTypeReference<List<Order>>() {});
    }

    // Full response (status + headers + body)
    public ResponseEntity<Order> getOrderWithMeta(UUID id) {
        return restClient.get()
            .uri("/api/orders/{id}", id)
            .retrieve()
            .toEntity(Order.class);
    }
}
```

### POST / PUT / DELETE

```java
// POST
public Order createOrder(CreateOrderRequest request) {
    return restClient.post()
        .uri("/api/orders")
        .contentType(MediaType.APPLICATION_JSON)
        .body(request)
        .retrieve()
        .body(Order.class);
}

// PUT
public Order updateOrder(UUID id, UpdateOrderRequest request) {
    return restClient.put()
        .uri("/api/orders/{id}", id)
        .body(request)
        .retrieve()
        .body(Order.class);
}

// DELETE
public void deleteOrder(UUID id) {
    restClient.delete()
        .uri("/api/orders/{id}", id)
        .retrieve()
        .toBodilessEntity();
}
```

### Request Interceptors

```java
@Bean
public RestClient restClient(RestClient.Builder builder) {
    return builder
        .baseUrl("http://api.example.com")
        .requestInterceptor((request, body, execution) -> {
            request.getHeaders().set("Authorization", "Bearer " + tokenService.getToken());
            request.getHeaders().set("X-Request-ID", UUID.randomUUID().toString());
            return execution.execute(request, body);
        })
        .build();
}
```

---

## WebClient (Reactive)

`WebClient` is the reactive, non-blocking HTTP client. Use it in WebFlux apps or when you need to make many concurrent requests.

### Setup

```java
@Bean
public WebClient inventoryWebClient(WebClient.Builder builder) {
    return builder
        .baseUrl("http://inventory-service")
        .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
        .filter(ExchangeFilterFunctions.basicAuthentication("user", "pass"))
        .filter(logRequest())
        .build();
}

private ExchangeFilterFunction logRequest() {
    return ExchangeFilterFunction.ofRequestProcessor(req -> {
        log.debug("Request: {} {}", req.method(), req.url());
        return Mono.just(req);
    });
}
```

### GET (reactive)

```java
@Service
public class InventoryClient {

    private final WebClient webClient;

    public Mono<StockLevel> getStock(String productId) {
        return webClient.get()
            .uri("/api/inventory/{id}", productId)
            .retrieve()
            .onStatus(HttpStatusCode::is4xxClientError,
                res -> Mono.error(new ProductNotFoundException(productId)))
            .bodyToMono(StockLevel.class);
    }

    public Flux<StockLevel> getAllStock() {
        return webClient.get()
            .uri("/api/inventory")
            .retrieve()
            .bodyToFlux(StockLevel.class);
    }
}
```

### Parallel Calls

```java
// Make two calls simultaneously and combine results
public Mono<OrderDetail> getOrderDetail(UUID orderId) {
    Mono<Order>    order    = orderClient.getOrder(orderId);
    Mono<Customer> customer = customerClient.getCustomer(orderId);

    return Mono.zip(order, customer)
        .map(tuple -> new OrderDetail(tuple.getT1(), tuple.getT2()));
}
```

### Calling Synchronously from a Blocking Context

```java
// Only do this in tests or non-reactive services — blocks the thread
StockLevel stock = webClient.get()
    .uri("/api/inventory/{id}", productId)
    .retrieve()
    .bodyToMono(StockLevel.class)
    .block();
```

---

## Feign — Declarative HTTP Clients

Feign generates an HTTP client from an annotated interface. No implementation code — just declare the contract.

### Maven Dependency

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients
public class Application { ... }
```

### Define the Client

```java
@FeignClient(name = "product-service", url = "${services.product.url}")
public interface ProductClient {

    @GetMapping("/api/products/{id}")
    ProductResponse getProduct(@PathVariable("id") UUID id);

    @GetMapping("/api/products")
    Page<ProductResponse> listProducts(
        @RequestParam(required = false) String category,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size);

    @PostMapping("/api/products")
    ProductResponse createProduct(@RequestBody CreateProductRequest request);

    @PutMapping("/api/products/{id}")
    ProductResponse updateProduct(
        @PathVariable("id") UUID id,
        @RequestBody UpdateProductRequest request);

    @DeleteMapping("/api/products/{id}")
    void deleteProduct(@PathVariable("id") UUID id);
}
```

### Use It Like Any Bean

```java
@Service
public class OrderService {

    private final ProductClient productClient;

    public Order placeOrder(PlaceOrderCommand cmd) {
        // Feign makes the HTTP call transparently
        ProductResponse product = productClient.getProduct(cmd.productId());
        // ...
    }
}
```

### Feign Configuration

```java
@Configuration
public class FeignConfig {

    // Auth header on every request
    @Bean
    public RequestInterceptor authInterceptor(TokenService tokenService) {
        return template ->
            template.header("Authorization", "Bearer " + tokenService.getServiceToken());
    }

    // Custom error decoder
    @Bean
    public ErrorDecoder errorDecoder() {
        return (methodKey, response) -> switch (response.status()) {
            case 404 -> new ResourceNotFoundException("Not found: " + methodKey);
            case 429 -> new RateLimitException("Rate limited by " + methodKey);
            default  -> new FeignException.InternalServerError(
                "Server error", response.request(), null, null);
        };
    }
}

@FeignClient(name = "product-service", configuration = FeignConfig.class)
public interface ProductClient { ... }
```

### Feign with Retry

```yaml
feign:
  client:
    config:
      product-service:
        connect-timeout: 2000
        read-timeout: 5000
        logger-level: FULL
  circuitbreaker:
    enabled: true   # integrates with Resilience4j
```

---

## Timeouts and Connection Pools

### RestClient via HttpClient

```java
@Bean
public RestClient restClient(RestClient.Builder builder) {
    HttpClient httpClient = HttpClient.newBuilder()
        .connectTimeout(Duration.ofSeconds(2))
        .build();

    ClientHttpRequestFactory factory =
        new JdkClientHttpRequestFactory(httpClient);

    return builder
        .requestFactory(factory)
        .build();
}
```

### WebClient via Reactor Netty

```java
@Bean
public WebClient webClient(WebClient.Builder builder) {
    HttpClient httpClient = HttpClient.create()
        .responseTimeout(Duration.ofSeconds(5))
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 2_000);

    return builder
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
}
```

---

## HTTP Clients Summary

| | `RestClient` | `WebClient` | Feign |
|---|---|---|---|
| Paradigm | Blocking | Reactive / non-blocking | Blocking (declarative) |
| Boilerplate | Low | Medium | Minimal |
| Best for | MVC services | WebFlux / many concurrent calls | Microservice-to-service |
| Retry / CB | Manual / Spring Retry | Reactor operators | Resilience4j integration |
| Spring Boot | 3.2+ | 2.x+ | Spring Cloud |

| Feature | API |
|---------|-----|
| Base URL | `builder.baseUrl(...)` |
| Default headers | `builder.defaultHeader(...)` |
| Error handling | `defaultStatusHandler(...)` / `onStatus(...)` |
| Request interceptor | `requestInterceptor(...)` / `filter(...)` |
| Parameterized type | `new ParameterizedTypeReference<List<T>>(){}` |
| Parallel (WebClient) | `Mono.zip(mono1, mono2)` |
| Feign enable | `@EnableFeignClients` + `@FeignClient` |

---

[← Back to README](../README.md)
