# Java Course Notes

A structured guide to core Java concepts, from how the JVM works to object-oriented programming.

---

## Topics

| # | Topic | Description |
|---|-------|-------------|
| 1 | [How Java Runs Everywhere](docs/01-how-java-works.md) | WORA, JVM, JRE, JDK, bytecode, and JIT compilation |
| 2 | [Data Types](docs/02-data-types.md) | Primitives, reference types, type casting, and `var` |
| 3 | [Operators and Expressions](docs/03-operators.md) | Arithmetic, logical, bitwise, ternary, and precedence |
| 4 | [Control Flow](docs/04-control-flow.md) | if/else, switch, for, while, break, and continue |
| 5 | [Methods](docs/05-methods.md) | Defining methods, overloading, varargs, and recursion |
| 6 | [Object-Oriented Programming](docs/06-oop.md) | Classes, encapsulation, inheritance, polymorphism, and abstraction |
| 7 | [Exception Handling](docs/07-exception-handling.md) | try/catch/finally, custom exceptions, and best practices |
| 8 | [Collections and Generics](docs/08-collections-and-generics.md) | Generics, List, Set, Map, Queue, and choosing the right collection |
| 9 | [Functional Programming and Streams](docs/09-functional-programming-and-streams.md) | Lambdas, functional interfaces, method references, Stream API, and Optional |
| 10 | [Concurrency and Multithreading](docs/10-concurrency-and-multithreading.md) | Threads, synchronization, executors, CompletableFuture, and concurrent collections |
| 11 | [I/O and File Handling](docs/11-io-and-file-handling.md) | Paths, Files, byte/character streams, Scanner, serialization, and best practices |
| 12 | [Design Patterns](docs/12-design-patterns.md) | Creational, structural, and behavioral patterns with Java examples |
| 13 | [Testing](docs/13-testing.md) | JUnit 5, assertions, parameterized tests, Mockito, and best practices |
| 14 | [Build Tools and Dependency Management](docs/14-build-tools.md) | Maven, Gradle, dependency scopes, transitive deps, and conflict resolution |
| 15 | [Modern Java Features](docs/15-modern-java-features.md) | Text blocks, records, sealed classes, pattern matching, and virtual threads (Java 9–21) |
| 16 | [Reflection and Annotations](docs/16-reflection-and-annotations.md) | Inspecting classes at runtime, custom annotations, and annotation-driven frameworks |
| 17 | [JVM Internals](docs/17-jvm-internals.md) | Heap, GC algorithms, JIT compilation, memory leaks, and diagnostic tools |
| 18 | [Networking and HTTP](docs/18-networking-and-http.md) | HttpClient, REST requests, JSON with Jackson, TCP/UDP sockets, and WebSockets |
| 19 | [Enums](docs/19-enums.md) | Enum constants, fields, methods, EnumSet, EnumMap, and enum-based patterns |
| 20 | [Date and Time API](docs/20-date-and-time.md) | LocalDate, LocalTime, ZonedDateTime, Instant, Duration, Period, and DateTimeFormatter |
| 21 | [Regular Expressions](docs/21-regular-expressions.md) | Pattern, Matcher, groups, flags, and common regex patterns |
| 22 | [JDBC and Database Access](docs/22-jdbc-and-databases.md) | Connecting to databases, PreparedStatement, transactions, HikariCP, and the DAO pattern |
| 23 | [Logging](docs/23-logging.md) | SLF4J, Logback, log levels, MDC, structured JSON logging, and best practices |
| 24 | [Java Module System](docs/24-java-module-system.md) | module-info.java, requires, exports, opens, uses/provides, and jlink |
| 25 | [Security](docs/25-security.md) | Hashing, AES/RSA encryption, SecureRandom, TLS, password hashing, and secure coding |
| 26 | [Internationalization](docs/26-internationalization.md) | Locale, ResourceBundle, MessageFormat, NumberFormat, and locale-aware formatting |
| 27 | [Strings](docs/27-strings.md) | String methods, StringBuilder, formatting, text blocks, and character utilities |
| 28 | [Interfaces and Abstract Classes](docs/28-interfaces-and-abstract-classes.md) | Contracts, default methods, abstract classes, sealed interfaces, and when to use each |
| 29 | [Comparable and Comparator](docs/29-comparable-and-comparator.md) | Natural ordering, custom ordering, Comparator chains, and sorting with streams |
| 30 | [JPA and Hibernate](docs/30-jpa-and-hibernate.md) | Entity mapping, JPQL, relationships, fetch strategies, cascades, and the N+1 problem |
| 31 | [Spring Boot](docs/31-spring-boot.md) | Dependency injection, REST controllers, Spring Data JPA, validation, and testing |
| 32 | [XML Processing](docs/32-xml-processing.md) | DOM, SAX, StAX, JAXB, and XPath |
| 33 | [Reactive Programming](docs/33-reactive-programming.md) | Project Reactor, Mono, Flux, operators, backpressure, and Spring WebFlux |
| 34 | [Structured Concurrency](docs/34-structured-concurrency.md) | StructuredTaskScope, ShutdownOnFailure, ShutdownOnSuccess, and virtual threads |
| 35 | [Inner Classes](docs/35-inner-classes.md) | Non-static inner, static nested, local, and anonymous classes |
| 36 | [Pattern Matching](docs/36-pattern-matching.md) | instanceof patterns, switch type patterns, guarded patterns, and record deconstruction |
| 37 | [JSON with Jackson](docs/37-json-with-jackson.md) | ObjectMapper, serialization, deserialization, annotations, JsonNode, and Spring Boot config |
| 38 | [Performance and Profiling](docs/38-performance-and-profiling.md) | JMH benchmarking, JFR, async-profiler, heap analysis, GC tuning, and common pitfalls |
| 39 | [Spring Security](docs/39-spring-security.md) | Authentication, JWT, SecurityFilterChain, method security, and CORS |
| 40 | [Docker and Java](docs/40-docker-and-java.md) | Dockerfiles, multi-stage builds, JVM container flags, Compose, and GraalVM native image |
| 41 | [Microservices Patterns](docs/41-microservices-patterns.md) | Circuit breaker, retry, bulkhead, API gateway, service discovery, Feign, and Saga |
| 42 | [Message Queues and Kafka](docs/42-message-queues-and-kafka.md) | Producers, consumers, consumer groups, error handling, dead-letter topics, and RabbitMQ |
| 43 | [Caching](docs/43-caching.md) | Spring Cache, Caffeine, Redis, cache-aside, TTL, and cache eviction |
| 44 | [SOLID Principles](docs/44-solid-principles.md) | SRP, OCP, LSP, ISP, DIP — with violations and fixes |
| 45 | [REST API Design](docs/45-rest-api-design.md) | Resource naming, status codes, pagination, versioning, error responses, and OpenAPI |
| 46 | [Optional](docs/46-optional.md) | Creating, transforming, filtering, chaining, and anti-patterns |
| 47 | [Integration Testing](docs/47-integration-testing.md) | @SpringBootTest, MockMvc, Testcontainers, slice tests, and @ServiceConnection |
| 48 | [Database Migrations](docs/48-database-migrations.md) | Flyway, Liquibase, versioned migrations, and testing schema changes |
| 49 | [NoSQL and MongoDB](docs/49-nosql-and-mongodb.md) | Document mapping, Spring Data MongoDB, MongoTemplate, aggregation, and transactions |
| 50 | [Monitoring and Actuator](docs/50-monitoring-and-actuator.md) | Spring Boot Actuator, Micrometer, custom metrics, Prometheus, and Grafana |
| 51 | [CI/CD with GitHub Actions](docs/51-ci-cd.md) | Build, test, Docker build/push, deploy workflows, and quality gates |
| 52 | [Kubernetes](docs/52-kubernetes.md) | Deployments, Services, Ingress, ConfigMaps, Secrets, HPA, and kubectl |
| 53 | [Domain-Driven Design](docs/53-domain-driven-design.md) | Entities, value objects, aggregates, repositories, domain events, and bounded contexts |
| 54 | [Event Sourcing and CQRS](docs/54-event-sourcing-and-cqrs.md) | Event store, aggregate replay, projections, command handlers, and read models |
| 55 | [GraphQL](docs/55-graphql.md) | Schema SDL, @QueryMapping, @MutationMapping, @BatchMapping, subscriptions, and testing |
| 56 | [Scheduling](docs/56-scheduling.md) | @Scheduled, cron expressions, async tasks, dynamic scheduling, and Quartz |
