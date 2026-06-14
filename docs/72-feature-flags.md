# Feature Flags

[← Back to README](../README.md)

---

**Feature flags** (also called feature toggles) let you deploy code to production without immediately exposing it to users. They decouple deployment from release, enabling gradual rollouts, A/B testing, kill switches, and safe experimentation.

```mermaid
flowchart LR
    deploy["Deploy code\n(flag OFF)"]
    test["Test in prod\n(flag ON for QA)"]
    rollout["Gradual rollout\n(10% → 50% → 100%)"]
    done["Full release"]

    deploy --> test --> rollout --> done
```

---

## Types of Feature Flags

| Type | Lifespan | Use Case |
|------|----------|----------|
| **Release toggle** | Days–weeks | Hide incomplete features during development |
| **Experiment toggle** | Days–weeks | A/B testing, measure impact |
| **Ops toggle** | Long-lived | Kill switch for performance-sensitive code paths |
| **Permission toggle** | Long-lived | Beta access, paid-tier features |

---

## Togglz — Lightweight In-Process Flags

### Maven Dependency

```xml
<dependency>
    <groupId>org.togglz</groupId>
    <artifactId>togglz-spring-boot-starter</artifactId>
    <version>4.4.0</version>
</dependency>
<!-- Optional: web console to toggle flags at runtime -->
<dependency>
    <groupId>org.togglz</groupId>
    <artifactId>togglz-console</artifactId>
    <version>4.4.0</version>
</dependency>
```

### Define Flags

```java
// Define all flags in one enum
public enum Features implements Feature {

    @Label("New checkout flow")
    NEW_CHECKOUT,

    @Label("AI product recommendations")
    AI_RECOMMENDATIONS,

    @Label("Express shipping option")
    EXPRESS_SHIPPING;

    public boolean isActive() {
        return FeatureContext.getFeatureManager().isActive(this);
    }
}
```

### Configuration

```java
@Configuration
public class TogglzConfig implements TogglzConfig {

    @Override
    public Class<? extends Feature> getFeatureClass() {
        return Features.class;
    }

    @Override
    public StateRepository getStateRepository() {
        // In-memory (dev) — all flags enabled
        return new InMemoryStateRepository();

        // File-based (staging)
        // return new FileBasedStateRepository(new File("/config/features.properties"));

        // JPA-backed (prod) — persists across restarts
        // return new JpaStateRepository(entityManagerFactory);
    }

    @Override
    public UserProvider getUserProvider() {
        // Spring Security user for user-scoped flags
        return new SpringSecurityUserProvider("ADMIN");
    }
}
```

### Using Flags in Code

```java
@Service
public class CheckoutService {

    public CheckoutResult checkout(Cart cart) {
        if (Features.NEW_CHECKOUT.isActive()) {
            return newCheckoutFlow(cart);
        }
        return legacyCheckoutFlow(cart);
    }
}
```

```java
// In a Spring MVC controller
@GetMapping("/recommendations")
public List<Product> getRecommendations(@AuthenticationPrincipal User user) {
    if (!Features.AI_RECOMMENDATIONS.isActive()) {
        return popularProducts();  // fallback
    }
    return aiRecommendationService.recommend(user.getId());
}
```

### Web Console

```yaml
# application.yml — expose the Togglz admin console
togglz:
  console:
    enabled: true
    path: /togglz-console   # → http://localhost:8080/togglz-console
    secured: true           # requires ADMIN role
```

---

## Unleash — Feature Flag Service

Unleash is a self-hosted feature flag platform with advanced targeting (user ID, gradual rollout %).

### Maven Dependency

```xml
<dependency>
    <groupId>io.getunleash</groupId>
    <artifactId>unleash-client-java</artifactId>
    <version>9.2.3</version>
</dependency>
```

### Running Unleash Locally

```yaml
# compose.yml
services:
  unleash:
    image: unleashorg/unleash-server:latest
    ports:
      - "4242:4242"
    environment:
      DATABASE_URL: postgres://unleash:unleash@postgres/unleash
      INIT_CLIENT_API_TOKENS: default:development.unleash-insecure-api-token
    depends_on:
      - postgres

  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: unleash
      POSTGRES_PASSWORD: unleash
      POSTGRES_DB: unleash
```

### Configuration

```java
@Bean
public Unleash unleash() {
    UnleashConfig config = UnleashConfig.builder()
        .appName("order-service")
        .instanceId("order-service-1")
        .unleashAPI("http://localhost:4242/api")
        .apiKey("default:development.unleash-insecure-api-token")
        .build();
    return new DefaultUnleash(config);
}
```

### Using Unleash Flags

```java
@Service
public class ShippingService {

    private final Unleash unleash;

    public ShippingService(Unleash unleash) {
        this.unleash = unleash;
    }

    public List<ShippingOption> getOptions(String userId) {
        // Simple on/off flag
        if (unleash.isEnabled("express-shipping")) {
            return extendedOptions();
        }
        return standardOptions();
    }

    public String getCheckoutVariant(String userId) {
        // A/B test — variant toggle
        Variant variant = unleash.getVariant("checkout-ab-test",
            new UnleashContext.Builder().userId(userId).build());

        return switch (variant.getName()) {
            case "new-flow" -> "new";
            case "legacy"   -> "legacy";
            default         -> "control";
        };
    }
}
```

### Gradual Rollout

In the Unleash UI, set the activation strategy to **Gradual rollout**:
- **Rollout %**: `20` — only 20% of users see the feature
- **Grouping ID**: `userId` — consistent per user (same user always gets same variant)

---

## Testing with Feature Flags

```java
@SpringBootTest
class CheckoutServiceTest {

    @MockBean
    Unleash unleash;  // or FeatureManager for Togglz

    @Autowired
    CheckoutService checkoutService;

    @Test
    void newCheckoutFlowWhenFlagEnabled() {
        when(unleash.isEnabled("new-checkout")).thenReturn(true);

        CheckoutResult result = checkoutService.checkout(testCart());

        assertThat(result.flowUsed()).isEqualTo("new");
    }

    @Test
    void legacyCheckoutFlowWhenFlagDisabled() {
        when(unleash.isEnabled("new-checkout")).thenReturn(false);

        CheckoutResult result = checkoutService.checkout(testCart());

        assertThat(result.flowUsed()).isEqualTo("legacy");
    }
}
```

---

## Best Practices

- **Set a removal date** when creating a flag — long-lived release flags become permanent tech debt.
- **Never nest flags** — `if (flagA && flagB)` becomes exponentially hard to test.
- **Use flags at the boundary** (controller / use case), not scattered through domain logic.
- **Default to disabled** — flags should be `false` when the service cannot reach the flag server.
- **Clean up flags** — delete the enum entry, the code branch, and the flag from the system after full rollout.

---

## Feature Flags Summary

| Tool | Hosting | Strengths |
|------|---------|-----------|
| Togglz | In-process | Simple setup, no external service, JPA persistence |
| Unleash | Self-hosted server | Gradual rollout, targeting, A/B variants, audit log |
| LaunchDarkly | SaaS | Enterprise features, real-time push, analytics |

| Pattern | Detail |
|---------|--------|
| Kill switch | Set flag OFF instantly to disable broken code |
| Gradual rollout | Enable for 10% → 50% → 100% of users |
| A/B test | Two variants measured for conversion/performance |
| Permission toggle | Only specific users/roles see the flag as enabled |

---

[← Back to README](../README.md)
