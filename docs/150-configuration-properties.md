# Spring Boot @ConfigurationProperties

[← Back to README](../README.md)

---

`@ConfigurationProperties` binds external configuration (YAML, environment variables, system properties) to typed Java beans. It is safer and more IDE-friendly than injecting individual values with `@Value` — it supports nested objects, lists, maps, validation, and relaxed binding out of the box.

```mermaid
flowchart LR
    yml["application.yml\napp.payment.timeout=5s"]
    binder["Spring Binder\n(relaxed binding)"]
    props["@ConfigurationProperties\nPaymentProperties"]
    svc["PaymentService\n(injected)"]

    yml --> binder --> props --> svc
```

---

## Defining a Properties Class

```java
@ConfigurationProperties(prefix = "app.payment")
@Validated
public class PaymentProperties {

    @NotNull
    private String provider;

    @NotNull
    @DurationUnit(ChronoUnit.SECONDS)
    private Duration timeout = Duration.ofSeconds(5);

    @Min(1) @Max(10)
    private int maxRetries = 3;

    private String apiKey;

    // Nested object
    private Webhook webhook = new Webhook();

    // List of allowed currencies
    private List<String> supportedCurrencies = List.of("USD", "EUR");

    // Map of fee rates by currency
    private Map<String, BigDecimal> feeRates = new HashMap<>();

    public static class Webhook {
        private String url;
        private String secret;
        private Duration connectTimeout = Duration.ofSeconds(2);
        // getters/setters
    }

    // getters/setters or use @Data / records (Spring Boot 3.0+)
}
```

```yaml
# application.yml
app:
  payment:
    provider: stripe
    timeout: 10s
    max-retries: 5            # kebab-case → maxRetries (relaxed binding)
    api-key: ${STRIPE_API_KEY}
    supported-currencies:
      - USD
      - EUR
      - GBP
    fee-rates:
      USD: 0.029
      EUR: 0.025
    webhook:
      url: https://api.example.com/webhooks/payment
      secret: ${WEBHOOK_SECRET}
      connect-timeout: 3s
```

---

## Registering Properties Classes

```java
// Option 1: @Component on the class itself
@Component
@ConfigurationProperties(prefix = "app.payment")
public class PaymentProperties { ... }

// Option 2: @EnableConfigurationProperties (preferred for library code)
@Configuration
@EnableConfigurationProperties(PaymentProperties.class)
public class PaymentConfig { }

// Option 3: @ConfigurationPropertiesScan (scan a package)
@SpringBootApplication
@ConfigurationPropertiesScan("com.example.config")
public class Application { }
```

---

## Injecting and Using Properties

```java
@Service
@RequiredArgsConstructor
public class PaymentService {

    private final PaymentProperties props;

    public PaymentResult charge(ChargeRequest request) {
        return paymentGateway.charge(
            request,
            props.getApiKey(),
            props.getTimeout(),
            props.getMaxRetries());
    }

    public boolean isCurrencySupported(String currency) {
        return props.getSupportedCurrencies().contains(currency);
    }

    public BigDecimal getFeeRate(String currency) {
        return props.getFeeRates().getOrDefault(currency, BigDecimal.ZERO);
    }
}
```

---

## Records as Properties (Spring Boot 3.0+)

Immutable configuration with constructor binding:

```java
@ConfigurationProperties(prefix = "app.database")
public record DatabaseProperties(
    @NotBlank String url,
    @NotBlank String username,
    @Min(1) @Max(100) int maxPoolSize,
    @DefaultValue("30s") Duration connectionTimeout,
    @DefaultValue("true") boolean sslEnabled
) {}
```

```yaml
app:
  database:
    url: jdbc:postgresql://localhost:5432/orders
    username: app
    max-pool-size: 20
    connection-timeout: 45s
```

---

## Validation

```java
@ConfigurationProperties(prefix = "app.cache")
@Validated
public class CacheProperties {

    @NotNull
    private String provider;       // caffeine | redis

    @Positive
    private int maxSize = 10_000;

    @DurationMin(seconds = 1)
    @DurationMax(minutes = 60)
    private Duration ttl = Duration.ofMinutes(5);

    @Pattern(regexp = "^[a-z][a-z0-9-]*$")
    private String keyPrefix = "app";
}
```

Validation errors fail fast at startup with a descriptive message — no runtime surprises.

---

## Profile-Specific Overrides

```yaml
# application.yml (defaults)
app:
  payment:
    provider: mock
    timeout: 5s

# application-prod.yml (production overrides)
app:
  payment:
    provider: stripe
    timeout: 10s
    api-key: ${STRIPE_API_KEY}
```

```bash
# Environment variables override YAML (highest precedence)
APP_PAYMENT_TIMEOUT=15s java -jar app.jar
```

---

## Testing @ConfigurationProperties

```java
// Unit test — bind properties directly
@Test
void properties_bindCorrectly() {
    PaymentProperties props = new PaymentProperties();
    props.setProvider("stripe");
    props.setTimeout(Duration.ofSeconds(10));
    assertThat(props.getTimeout()).isEqualTo(Duration.ofSeconds(10));
}

// Slice test — load only the config bean
@SpringBootTest
@EnableConfigurationProperties(PaymentProperties.class)
@TestPropertySource(properties = {
    "app.payment.provider=test-gateway",
    "app.payment.timeout=2s",
    "app.payment.max-retries=1"
})
class PaymentPropertiesTest {

    @Autowired PaymentProperties props;

    @Test
    void props_loadedCorrectly() {
        assertThat(props.getProvider()).isEqualTo("test-gateway");
        assertThat(props.getTimeout()).isEqualTo(Duration.ofSeconds(2));
        assertThat(props.getMaxRetries()).isEqualTo(1);
    }
}
```

---

## IDE Metadata (Auto-complete Support)

```xml
<!-- Generates META-INF/spring-configuration-metadata.json -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

After adding the processor, IDEs provide auto-complete and documentation hints in `application.yml`.

---

## @ConfigurationProperties Summary

| Concept | Detail |
|---------|--------|
| `@ConfigurationProperties(prefix)` | Bind all properties under `prefix` to this class |
| Relaxed binding | `kebab-case`, `UPPER_CASE`, `camelCase` all map to the same field |
| `@Validated` | Trigger JSR-303 validation at startup; fail fast on invalid config |
| `@DefaultValue` | Provide a default in constructor-binding (records) |
| `@DurationUnit` | Parse duration strings like `5s`, `2m`, `1h` to `java.time.Duration` |
| `@EnableConfigurationProperties` | Register a properties class that isn't annotated with `@Component` |
| `@ConfigurationPropertiesScan` | Scan a package for all `@ConfigurationProperties` classes |
| Records | Immutable config binding in Spring Boot 3+; constructor-bound |
| Environment variable override | `APP_PAYMENT_TIMEOUT=10s` overrides `app.payment.timeout` |
| Configuration processor | Add `spring-boot-configuration-processor` for IDE auto-complete in YAML |

---

[← Back to README](../README.md)
