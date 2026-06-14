# Spring Retry

[← Back to README](../README.md)

---

**Spring Retry** provides declarative and programmatic retry support for operations that may fail transiently — network calls, database timeouts, external API rate limits. It eliminates boilerplate retry loops and integrates cleanly with Spring Boot.

```mermaid
flowchart LR
    call["Method Call"]
    attempt["Attempt 1"]
    fail1["Failure"]
    backoff["Backoff\n(wait)"]
    attempt2["Attempt 2"]
    fail2["Failure"]
    recover["@Recover\n(fallback)"]
    success["Success"]

    call --> attempt --> fail1 --> backoff --> attempt2
    attempt2 --> success
    attempt2 --> fail2 --> recover
```

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
<!-- Spring Retry uses AOP under the hood -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableRetry
public class Application { ... }
```

---

## `@Retryable` — Basic Usage

```java
@Service
public class PaymentService {

    // Retry up to 3 times on any RuntimeException
    @Retryable(retryFor = RuntimeException.class, maxAttempts = 3)
    public PaymentResult charge(String customerId, BigDecimal amount) {
        return stripeClient.charge(customerId, amount);
    }
}
```

---

## Backoff Strategies

```java
@Service
public class ExternalApiClient {

    // Fixed delay — wait 2 s between each attempt
    @Retryable(
        retryFor = HttpServerErrorException.class,
        maxAttempts = 4,
        backoff = @Backoff(delay = 2_000))
    public ApiResponse callApi(ApiRequest request) { ... }

    // Exponential backoff — 1 s, 2 s, 4 s, 8 s (doubles each time)
    @Retryable(
        retryFor = {ConnectException.class, SocketTimeoutException.class},
        maxAttempts = 5,
        backoff = @Backoff(delay = 1_000, multiplier = 2.0, maxDelay = 30_000))
    public ApiResponse callWithExponentialBackoff(ApiRequest request) { ... }

    // Exponential backoff with jitter — avoids thundering herd
    @Retryable(
        retryFor = TransientException.class,
        maxAttempts = 4,
        backoff = @Backoff(delay = 1_000, multiplier = 2.0, random = true))
    public ApiResponse callWithJitter(ApiRequest request) { ... }
}
```

---

## `@Recover` — Fallback After All Retries Exhausted

```java
@Service
public class NotificationService {

    @Retryable(
        retryFor = MessagingException.class,
        maxAttempts = 3,
        backoff = @Backoff(delay = 1_000))
    public void sendEmail(String to, String subject, String body) {
        emailClient.send(to, subject, body);
    }

    // Called when all 3 attempts fail
    // Must have same return type and first arg is the exception
    @Recover
    public void recoverSendEmail(MessagingException ex,
                                  String to, String subject, String body) {
        log.error("Failed to send email to {} after retries: {}", to, ex.getMessage());
        // store in outbox / alert ops team
        failedEmailRepository.save(new FailedEmail(to, subject, body, ex.getMessage()));
    }
}
```

---

## Excluding Exceptions

```java
// Retry on all exceptions EXCEPT these
@Retryable(
    retryFor   = Exception.class,
    noRetryFor = {IllegalArgumentException.class, ValidationException.class},
    maxAttempts = 3,
    backoff    = @Backoff(delay = 500))
public Order placeOrder(PlaceOrderCommand cmd) { ... }
```

---

## Retry with Stateful Logic

Use `RetryCallback` to access context about the current attempt:

```java
@Retryable(
    retryFor = TransientDataAccessException.class,
    maxAttempts = 3,
    listeners = "retryLoggingListener")
public void importData(List<Record> records) {
    dataRepository.saveAll(records);
}

@Component("retryLoggingListener")
public class RetryLoggingListener extends RetryListenerSupport {

    @Override
    public <T, E extends Throwable> void onError(
            RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
        log.warn("Retry attempt {} failed: {}",
            context.getRetryCount(), throwable.getMessage());
    }

    @Override
    public <T, E extends Throwable> void close(
            RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
        if (throwable != null) {
            log.error("All {} retry attempts exhausted", context.getRetryCount());
        }
    }
}
```

---

## Programmatic Retry with `RetryTemplate`

When you need retry logic outside of Spring-managed beans:

```java
@Bean
public RetryTemplate retryTemplate() {
    return RetryTemplate.builder()
        .maxAttempts(5)
        .exponentialBackoff(1_000, 2.0, 30_000)
        .retryOn(TransientException.class)
        .notRetryOn(PermanentException.class)
        .withListener(new RetryLoggingListener())
        .build();
}

@Service
public class DataSyncService {

    private final RetryTemplate retryTemplate;

    public void syncToRemote(SyncPayload payload) {
        retryTemplate.execute(
            context -> {
                remoteClient.push(payload);
                return null;
            },
            context -> {
                // recovery callback
                log.error("Sync failed after {} attempts", context.getRetryCount());
                deadLetterQueue.enqueue(payload);
                return null;
            });
    }
}
```

---

## `@CircuitBreaker` — Open Circuit After Repeated Failures

Spring Retry also provides a simple circuit breaker:

```java
@Service
public class WeatherService {

    // Opens circuit after 3 failures; resets after 5 seconds
    @CircuitBreaker(maxAttempts = 3, openTimeout = 5_000, resetTimeout = 20_000)
    public WeatherData getWeather(String city) {
        return weatherApiClient.fetch(city);
    }

    @Recover
    public WeatherData fallback(Exception ex, String city) {
        return WeatherData.unavailable(city);
    }
}
```

For production circuit breakers with metrics and dashboards, use Resilience4j (covered in the Microservices Patterns doc).

---

## Combining with `@Transactional`

Retry must wrap the transaction, not the other way around. Order: Retry → Transaction → Method.

```java
// CORRECT — @Retryable is the outer proxy
@Retryable(retryFor = OptimisticLockingFailureException.class, maxAttempts = 3)
@Transactional
public void updateOrder(String orderId, UpdateCommand cmd) {
    Order order = repo.findById(orderId).orElseThrow();
    order.apply(cmd);
    repo.save(order);
}
```

If `@Transactional` wraps `@Retryable`, the transaction commits before retry kicks in — making retries meaningless.

---

## Testing Retried Methods

```java
@SpringBootTest
class PaymentServiceRetryTest {

    @MockBean
    StripeClient stripeClient;

    @Autowired
    PaymentService paymentService;

    @Test
    void retriesOnTransientFailure() {
        when(stripeClient.charge(any(), any()))
            .thenThrow(new TransientStripeException("timeout"))
            .thenThrow(new TransientStripeException("timeout"))
            .thenReturn(new PaymentResult(true, "txn_123"));

        PaymentResult result = paymentService.charge("cust-1", new BigDecimal("50.00"));

        assertThat(result.success()).isTrue();
        verify(stripeClient, times(3)).charge(any(), any());
    }

    @Test
    void callsRecoverAfterAllRetries() {
        when(stripeClient.charge(any(), any()))
            .thenThrow(new TransientStripeException("gateway down"));

        assertThatThrownBy(() ->
            paymentService.charge("cust-1", new BigDecimal("50.00")))
            .isInstanceOf(TransientStripeException.class);

        verify(stripeClient, times(3)).charge(any(), any());
    }
}
```

---

## Spring Retry Summary

| Feature | Annotation / API |
|---------|-----------------|
| Basic retry | `@Retryable(retryFor = ..., maxAttempts = N)` |
| Fixed backoff | `@Backoff(delay = 1000)` |
| Exponential backoff | `@Backoff(delay = 1000, multiplier = 2.0, maxDelay = 30000)` |
| Jitter | `@Backoff(random = true)` |
| Fallback | `@Recover` method — same return type, exception as first arg |
| Exclude exceptions | `noRetryFor = {IllegalArgumentException.class}` |
| Listener | `RetryListenerSupport` — `onError` / `close` hooks |
| Programmatic | `RetryTemplate.builder()...build()` |
| Circuit breaker | `@CircuitBreaker(maxAttempts, openTimeout, resetTimeout)` |
| Enable | `@EnableRetry` on a `@Configuration` class |

---

[← Back to README](../README.md)
