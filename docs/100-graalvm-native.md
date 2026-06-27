# GraalVM Native Image

[← Back to README](../README.md)

---

**GraalVM Native Image** compiles Java ahead-of-time (AOT) into a standalone native binary. The result starts in milliseconds, uses a fraction of the memory of a JVM app, and ships as a single self-contained executable — ideal for serverless functions, CLI tools, and containers where cold-start time matters.

```mermaid
flowchart LR
    java["Java Source"]
    javac["javac"]
    bytecode["Bytecode (.class)"]
    native["native-image"]
    binary["Native Binary\n(no JVM needed)"]

    java --> javac --> bytecode --> native --> binary
```

---

## JVM vs Native Image

| | JVM (JIT) | Native Image (AOT) |
|---|---|---|
| Startup time | 1–5 s | 10–100 ms |
| Peak throughput | Higher (JIT optimises) | Slightly lower |
| Memory at startup | 200–500 MB | 20–60 MB |
| Binary size | JAR + JVM | 50–100 MB self-contained |
| Dynamic features | Full reflection, class loading | Restricted — needs config |
| Best for | Long-running services | Serverless, CLI, containers |

---

## Spring Boot Native Setup

Spring Boot 3.x has first-class native support via the `spring-boot-maven-plugin`.

### Maven Configuration

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.0</version>
</parent>

<!-- Native profile — activated with -Pnative -->
<profiles>
    <profile>
        <id>native</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <goals><goal>compile-no-fork</goal></goals>
                            <phase>package</phase>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

### Build

```bash
# Build native binary (requires GraalVM JDK or sdkman)
sdk install java 21.0.3-graal
mvn -Pnative package

# Result: target/order-service  (no JVM needed)
./target/order-service
# Started in 0.082s

# Or build a native Docker image (no GraalVM needed locally)
mvn -Pnative spring-boot:build-image
docker run --rm -p 8080:8080 order-service:0.0.1-SNAPSHOT
```

---

## Reflection Configuration

Native Image performs a **closed-world** analysis at build time. Reflection, serialization, and dynamic proxies used at runtime must be declared explicitly.

### Automatic with Spring AOT

Spring Boot 3 runs an AOT phase that auto-generates most hints:

```bash
mvn spring-boot:process-aot   # generates src/main/resources/META-INF/native-image/
```

This creates `reflect-config.json`, `resource-config.json`, and `proxy-config.json` automatically for Spring beans, JPA, Jackson, etc.

### Manual Hints for Custom Code

```java
// Programmatic — register in a RuntimeHintsRegistrar
@Component
public class MyRuntimeHints implements RuntimeHintsRegistrar {

    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        // Reflection
        hints.reflection().registerType(MyDynamicClass.class,
            MemberCategory.INVOKE_DECLARED_CONSTRUCTORS,
            MemberCategory.INVOKE_PUBLIC_METHODS);

        // Resources
        hints.resources().registerPattern("templates/*.html");
        hints.resources().registerPattern("db/migration/*.sql");

        // Serialization
        hints.serialization().registerType(MySerializableRecord.class);

        // JDK proxies
        hints.proxies().registerJdkProxy(MyService.class);
    }
}

@SpringBootApplication
@ImportRuntimeHints(MyRuntimeHints.class)
public class Application { ... }
```

### JSON Config Files (Fallback)

```json
// src/main/resources/META-INF/native-image/reflect-config.json
[
  {
    "name": "com.example.MyClass",
    "allDeclaredConstructors": true,
    "allPublicMethods": true,
    "allDeclaredFields": true
  }
]
```

---

## Testing Native Images

```bash
# Run native tests (compiles and runs tests as native binary)
mvn -Pnative test

# Integration test against a native binary
mvn -Pnative verify
```

### JUnit Native Test Annotation

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestPropertySource(properties = "spring.profiles.active=test")
class OrderControllerNativeTest {

    @Autowired TestRestTemplate restTemplate;

    @Test
    void placeOrderReturns201() {
        ResponseEntity<OrderResponse> response = restTemplate.postForEntity(
            "/api/orders", new PlaceOrderRequest("CUST-1", List.of()), OrderResponse.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
    }
}
```

---

## Native Image Limitations

| Feature | Status |
|---------|--------|
| `Class.forName()` with a variable | Needs reflection config |
| Java serialization (`ObjectInputStream`) | Needs serialization config; avoid if possible |
| Bytecode manipulation (ASM, ByteBuddy) | Not supported at runtime — use build-time alternatives |
| Dynamic class loading | Not supported |
| `java.lang.reflect.Proxy` | Needs proxy config |
| JVM attach API / agents | Not available |
| Finalizers | Deprecated — use `Cleaner` |

---

## Optimisation Flags

```bash
# Enable G1 GC (better throughput than default Serial GC)
native-image --gc=G1 ...

# Profile-guided optimisation (PGO) — collect profiles, then rebuild
native-image --pgo-instrument -jar app.jar    # step 1 — collect profiles
./app < workload                               # step 2 — run realistic traffic
native-image --pgo=default.iprof -jar app.jar # step 3 — rebuild with profiles

# Spring Boot build args
<buildArgs>
    <buildArg>--gc=G1</buildArg>
    <buildArg>-O2</buildArg>
    <buildArg>--initialize-at-build-time=org.slf4j</buildArg>
</buildArgs>
```

---

## Native in Docker (Buildpack)

```dockerfile
# Paketo Buildpack — no GraalVM needed on the build machine
FROM ubuntu:22.04 AS builder
# ... (handled automatically by spring-boot:build-image)

# Or manual multi-stage build
FROM ghcr.io/graalvm/native-image:21 AS builder
WORKDIR /app
COPY . .
RUN ./mvnw -Pnative -DskipTests package

FROM ubuntu:22.04
WORKDIR /app
COPY --from=builder /app/target/order-service .
EXPOSE 8080
ENTRYPOINT ["./order-service"]
```

```bash
# Buildpack approach — easiest
mvn -Pnative spring-boot:build-image \
  -Dspring-boot.build-image.imageName=order-service:native

docker run --rm -p 8080:8080 order-service:native
```

---

## GraalVM Native Image Summary

| Concept | Detail |
|---------|--------|
| AOT compilation | Entire app compiled to machine code at build time |
| Closed-world | All reachable code must be known at build time |
| Spring AOT | Auto-generates reflection/resource hints for Spring apps |
| `RuntimeHintsRegistrar` | Programmatic registration of reflection / resources / proxies |
| Build command | `mvn -Pnative package` or `spring-boot:build-image` |
| Startup | ~100 ms vs 2–5 s for JVM |
| Memory | ~50 MB vs 300 MB+ for JVM |
| Tradeoff | No JIT peak throughput; restricted dynamic features |
| Best for | Serverless (AWS Lambda, Azure Functions), CLI tools, containers |

---

[← Back to README](../README.md)
