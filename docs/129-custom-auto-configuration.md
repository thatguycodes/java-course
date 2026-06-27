# Spring Boot Custom Auto-Configuration

[← Back to README](../README.md)

---

Spring Boot's magic comes from **auto-configuration** — `@AutoConfiguration` classes that register beans only when certain conditions are met. Writing your own lets you package reusable infrastructure (audit logging, multi-tenancy, health checks) as a drop-in library that configures itself when added to a project.

```mermaid
flowchart LR
    jar["Your Library JAR\n(spring-boot-starter-audit)"]
    factories["META-INF/spring/\norg.springframework.boot.\nautoconfigurate.AutoConfiguration.imports"]
    cond["@ConditionalOnClass\n@ConditionalOnProperty\n@ConditionalOnMissingBean"]
    beans["Auto-registered Beans\n(AuditService, AuditRepository)"]

    jar --> factories --> cond --> beans
```

---

## Project Structure

```
my-audit-starter/
├── src/main/java/com/example/audit/
│   ├── AuditService.java
│   ├── AuditRepository.java
│   ├── AuditProperties.java
│   └── AuditAutoConfiguration.java
└── src/main/resources/
    └── META-INF/spring/
        └── org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

---

## Configuration Properties

```java
@ConfigurationProperties(prefix = "app.audit")
@Validated
public class AuditProperties {

    /** Whether audit logging is enabled */
    private boolean enabled = true;

    /** Topic or table name for audit events */
    private String destination = "audit-log";

    @NotNull
    private RetentionPolicy retention = new RetentionPolicy();

    @Getter @Setter
    public static class RetentionPolicy {
        private Duration period = Duration.ofDays(90);
        private boolean autoCleanup = true;
    }

    // getters, setters
}
```

---

## Auto-Configuration Class

```java
@AutoConfiguration
@ConditionalOnClass(AuditService.class)   // only if AuditService is on the classpath
@ConditionalOnProperty(
    prefix = "app.audit",
    name = "enabled",
    havingValue = "true",
    matchIfMissing = true)               // enabled by default
@EnableConfigurationProperties(AuditProperties.class)
public class AuditAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean             // user can override by declaring their own
    public AuditRepository auditRepository(AuditProperties props) {
        return new DefaultAuditRepository(props.getDestination());
    }

    @Bean
    @ConditionalOnMissingBean
    public AuditService auditService(AuditRepository repo, AuditProperties props) {
        return new AuditService(repo, props);
    }

    @Bean
    @ConditionalOnClass(name = "org.springframework.web.servlet.DispatcherServlet")
    @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
    public AuditWebMvcConfigurer auditMvcConfigurer(AuditService auditService) {
        return new AuditWebMvcConfigurer(auditService);   // only in MVC apps
    }
}
```

---

## Registering the Auto-Configuration

Spring Boot 2.7+ uses imports file instead of `spring.factories`:

```
# src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.example.audit.AuditAutoConfiguration
```

For Spring Boot 2.x (legacy):
```properties
# src/main/resources/META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.audit.AuditAutoConfiguration
```

---

## Conditional Annotations

```java
@AutoConfiguration
public class MyAutoConfiguration {

    // Class presence on classpath
    @ConditionalOnClass(KafkaTemplate.class)
    @Bean public KafkaAuditPublisher kafkaPublisher() { ... }

    // Class ABSENCE (provide default when lib not present)
    @ConditionalOnMissingClass("io.micrometer.core.instrument.MeterRegistry")
    @Bean public NoOpMetrics noOpMetrics() { ... }

    // Bean presence / absence
    @ConditionalOnBean(DataSource.class)
    @Bean public JdbcAuditRepository jdbcRepo(DataSource ds) { ... }

    @ConditionalOnMissingBean(AuditRepository.class)
    @Bean public InMemoryAuditRepository inMemoryRepo() { ... }

    // Property value
    @ConditionalOnProperty(name = "app.audit.backend", havingValue = "kafka")
    @Bean public KafkaAuditPublisher kafka() { ... }

    @ConditionalOnProperty(name = "app.audit.backend", havingValue = "jdbc",
                           matchIfMissing = true)
    @Bean public JdbcAuditRepository jdbc() { ... }

    // Application type
    @ConditionalOnWebApplication(type = Type.SERVLET)
    @Bean public AuditFilter servletFilter() { ... }

    @ConditionalOnWebApplication(type = Type.REACTIVE)
    @Bean public AuditWebFilter reactiveFilter() { ... }

    // Expression
    @ConditionalOnExpression("${app.audit.enabled} and ${app.metrics.enabled}")
    @Bean public MetricAuditBridge bridge() { ... }

    // Custom condition
    @Conditional(ProductionEnvironmentCondition.class)
    @Bean public ProdAuditLogger prodLogger() { ... }
}
```

---

## Custom Condition

```java
public class ProductionEnvironmentCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnvironment();
        String[] profiles = env.getActiveProfiles();
        return Arrays.asList(profiles).contains("prod");
    }
}
```

---

## Auto-Configuration Ordering

```java
// Run after Spring's DataSource auto-configuration
@AutoConfiguration(after = DataSourceAutoConfiguration.class)
public class AuditAutoConfiguration { ... }

// Run before your security config
@AutoConfiguration(before = SecurityAutoConfiguration.class)
public class AuditAutoConfiguration { ... }

// Named ordering (multiple)
@AutoConfiguration(
    after  = { DataSourceAutoConfiguration.class, JpaAutoConfiguration.class },
    before = { SecurityAutoConfiguration.class })
public class AuditAutoConfiguration { ... }
```

---

## Configuration Metadata (IDE Hints)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

This generates `META-INF/spring-configuration-metadata.json` at build time, enabling IDE auto-complete for your `app.audit.*` properties.

Add additional hints manually:

```json
// src/main/resources/META-INF/additional-spring-configuration-metadata.json
{
  "hints": [
    {
      "name": "app.audit.backend",
      "values": [
        { "value": "kafka",  "description": "Publish audit events to a Kafka topic" },
        { "value": "jdbc",   "description": "Store audit events in a relational DB" },
        { "value": "memory", "description": "In-memory store (dev only)" }
      ]
    }
  ]
}
```

---

## Creating a Starter POM

A **starter** is a convenience POM that pulls in your auto-configuration plus its dependencies:

```xml
<!-- audit-spring-boot-starter/pom.xml -->
<artifactId>audit-spring-boot-starter</artifactId>

<dependencies>
    <!-- The auto-configuration module -->
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>audit-spring-boot-autoconfigure</artifactId>
        <version>${project.version}</version>
    </dependency>

    <!-- Transitive dependencies users always need -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
</dependencies>
```

---

## Testing Auto-Configuration

```java
class AuditAutoConfigurationTest {

    private final ApplicationContextRunner runner = new ApplicationContextRunner()
        .withConfiguration(AutoConfigurations.of(AuditAutoConfiguration.class));

    @Test
    void autoConfiguresServiceAndRepository() {
        runner.run(ctx -> {
            assertThat(ctx).hasSingleBean(AuditService.class);
            assertThat(ctx).hasSingleBean(AuditRepository.class);
        });
    }

    @Test
    void backoffWhenDisabled() {
        runner.withPropertyValues("app.audit.enabled=false")
            .run(ctx -> assertThat(ctx).doesNotHaveBean(AuditService.class));
    }

    @Test
    void userDefinedBeanOverridesAutoConfigured() {
        runner.withUserConfiguration(UserAuditConfig.class)
            .run(ctx -> {
                assertThat(ctx).hasSingleBean(AuditRepository.class);
                assertThat(ctx.getBean(AuditRepository.class))
                    .isInstanceOf(CustomAuditRepository.class);
            });
    }

    @Configuration
    static class UserAuditConfig {
        @Bean
        AuditRepository auditRepository() {
            return new CustomAuditRepository();
        }
    }
}
```

---

## Custom Auto-Configuration Summary

| Concept | Detail |
|---------|--------|
| `@AutoConfiguration` | Marks a class as auto-configurable; replaces `@Configuration` for auto-config |
| `.AutoConfiguration.imports` | Lists auto-configuration classes to load (Boot 2.7+) |
| `@ConditionalOnClass` | Activate only when a class is on the classpath |
| `@ConditionalOnMissingBean` | Register default bean only if user hasn't defined one |
| `@ConditionalOnProperty` | Toggle feature via property; `matchIfMissing=true` = on by default |
| `@EnableConfigurationProperties` | Bind `@ConfigurationProperties` class and expose it as a bean |
| `after` / `before` ordering | Ensure correct bean dependencies across auto-configs |
| `ApplicationContextRunner` | Test auto-configurations in isolation without starting a full context |
| Configuration processor | Generates IDE auto-complete metadata from `@ConfigurationProperties` |
| Starter POM | Zero-dependency entry point — user adds one dep and gets everything |

---

[← Back to README](../README.md)
