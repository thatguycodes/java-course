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
| 57 | [gRPC](docs/57-grpc.md) | Protocol Buffers, service definitions, unary and streaming RPCs, and Spring Boot integration |
| 58 | [OAuth2 and OIDC](docs/58-oauth2-and-oidc.md) | OAuth2 grant types, Spring OAuth2 Client, resource server JWT validation, and Keycloak |
| 59 | [Spring Batch](docs/59-spring-batch.md) | Jobs, steps, chunk processing, readers/writers, fault tolerance, and partitioning |
| 60 | [File Storage (S3 / MinIO)](docs/60-file-storage.md) | AWS SDK v2, Spring Cloud AWS, upload/download, presigned URLs, and local MinIO |
| 61 | [WebSockets](docs/61-websockets.md) | STOMP over WebSocket, @MessageMapping, SimpMessagingTemplate, and auth |
| 62 | [Email with Spring Mail](docs/62-email.md) | JavaMailSender, HTML emails, Thymeleaf templates, attachments, and Mailpit |
| 63 | [Test-Driven Development](docs/63-tdd.md) | Red-Green-Refactor, three rules of TDD, AAA pattern, triangulation, and outside-in TDD |
| 64 | [Load Testing](docs/64-load-testing.md) | Gatling and k6 scenarios, virtual users, percentile thresholds, and CI integration |
| 65 | [Contract Testing](docs/65-contract-testing.md) | Pact consumer/provider contracts, Pact Broker, can-i-deploy, and Spring Cloud Contract |
| 66 | [Hexagonal Architecture](docs/66-hexagonal-architecture.md) | Ports & adapters, driving/driven ports, domain isolation, and adapter wiring |
| 67 | [Distributed Tracing](docs/67-distributed-tracing.md) | OpenTelemetry, Micrometer Tracing, Zipkin, Grafana Tempo, and custom spans |
| 68 | [Server-Sent Events (SSE)](docs/68-server-sent-events.md) | SseEmitter, reactive Flux SSE, auto-reconnect, and authenticated event streams |
| 69 | [Elasticsearch](docs/69-elasticsearch.md) | Spring Data Elasticsearch, full-text search, aggregations, and autocomplete |
| 70 | [Property-Based Testing](docs/70-property-based-testing.md) | jqwik, generators, shrinking, stateful action sequences, and combining with examples |
| 71 | [BDD with Cucumber](docs/71-bdd-with-cucumber.md) | Gherkin feature files, step definitions, data tables, hooks, and Spring integration |
| 72 | [Feature Flags](docs/72-feature-flags.md) | Togglz, Unleash, gradual rollout, A/B variants, and kill switches |
| 73 | [Transactional Outbox Pattern](docs/73-transactional-outbox.md) | Dual-write problem, outbox table, polling publisher, and Debezium CDC |
| 74 | [Rate Limiting](docs/74-rate-limiting.md) | Bucket4j, token bucket, distributed Redis limits, and Spring Cloud Gateway |
| 75 | [Clean Architecture](docs/75-clean-architecture.md) | Dependency rule, entities, use cases, interface adapters, and framework ring |
| 76 | [OpenAPI / Swagger](docs/76-openapi-swagger.md) | springdoc-openapi, @Operation, @Schema, Swagger UI, and code generation |
| 77 | [Java Memory Model](docs/77-java-memory-model.md) | Happens-before, volatile, atomics, synchronized, and memory reordering |
| 78 | [Event-Driven Architecture](docs/78-event-driven-architecture.md) | Choreography vs orchestration, idempotency, sagas, and schema registry |
| 79 | [Spring Cloud Config](docs/79-spring-cloud-config.md) | Config server, Git backend, @RefreshScope, Cloud Bus, and encryption |
| 80 | [Mutation Testing (PIT)](docs/80-mutation-testing.md) | Mutant operators, mutation score, surviving mutants, and CI gating |
| 81 | [Secrets Management](docs/81-secrets-management.md) | HashiCorp Vault, AWS Secrets Manager, dynamic credentials, and K8s secrets |
| 82 | [Deployment Strategies](docs/82-deployment-strategies.md) | Blue-green, canary, rolling updates, readiness probes, and zero-downtime DB migrations |
| 83 | [CORS and CSRF](docs/83-cors-and-csrf.md) | Same-origin policy, preflight, Spring Security CORS config, CSRF tokens, and SameSite |
| 84 | [RabbitMQ Deep Dive](docs/84-rabbitmq.md) | Exchanges, bindings, DLQ, competing consumers, fanout, topic routing, and RPC |
| 85 | [Records and Sealed Classes](docs/85-records-and-sealed-classes.md) | Compact constructors, withers, sealed permits, exhaustive switch, and domain errors |
| 86 | [Spring AOP](docs/86-spring-aop.md) | Aspects, pointcuts, @Around/@Before/@After, custom annotations, and AOP limitations |
| 87 | [Bean Validation](docs/87-bean-validation.md) | Jakarta Validation, custom constraints, cross-field validation, groups, and error responses |
| 88 | [Redis Deep Dive](docs/88-redis-deep-dive.md) | Strings, hashes, lists, sets, sorted sets, pub/sub, streams, Lua scripts, distributed locks |
| 89 | [Spring Retry](docs/89-spring-retry.md) | @Retryable, exponential backoff, jitter, @Recover fallbacks, and RetryTemplate |
| 90 | [Advanced Generics](docs/90-advanced-generics.md) | Wildcards, bounds, PECS, type erasure, super-type tokens, and recursive bounds |
| 91 | [HTTP Clients](docs/91-http-clients.md) | RestClient, WebClient, Feign, interceptors, timeouts, and parallel calls |
| 92 | [API Pagination](docs/92-api-pagination.md) | Offset, keyset, cursor pagination, Pageable, Slice, and Link headers |
| 93 | [Webhooks](docs/93-webhooks.md) | Registration, HMAC signing, delivery, retry with backoff, and idempotency |
| 94 | [Multi-Tenancy](docs/94-multi-tenancy.md) | Discriminator column, schema-per-tenant, separate databases, and tenant routing |
| 95 | [Java ServiceLoader (SPI)](docs/95-service-loader.md) | Service interfaces, META-INF/services, module-info provides/uses, and lazy loading |
| 96 | [Lombok](docs/96-lombok.md) | @Getter/@Setter, @Builder, @Data, @Value, @Slf4j, @NonNull, and JPA entity caveats |
| 97 | [MapStruct](docs/97-mapstruct.md) | @Mapper, @Mapping, nested mappers, @MappingTarget, null strategies, and Lombok ordering |
| 98 | [Code Quality Tools](docs/98-code-quality.md) | Checkstyle, SpotBugs, PMD, JaCoCo coverage gates, and SonarQube integration |
| 99 | [OWASP Top 10 for Java](docs/99-owasp-top-10.md) | Injection, broken access control, XSS, insecure deserialization, and SSRF — with fixes |
| 100 | [GraalVM Native Image](docs/100-graalvm-native.md) | AOT compilation, Spring AOT hints, RuntimeHintsRegistrar, native Docker, and limitations |
| 101 | [Log Aggregation](docs/101-log-aggregation.md) | Loki + Promtail, ELK stack, LogQL queries, log-trace correlation, and Kubernetes Fluent Bit |
| 102 | [HikariCP & Connection Pool Tuning](docs/102-hikaricp.md) | Pool sizing formula, timeout settings, Micrometer metrics, leak detection, and multi-datasource |
| 103 | [Java Anti-Patterns](docs/103-anti-patterns.md) | Anemic model, God class, service locator, primitive obsession, N+1, broad @Transactional |
| 104 | [Spring HATEOAS](docs/104-spring-hateoas.md) | EntityModel, CollectionModel, PagedModel, LinkBuilder, affordances, and HAL responses |
| 105 | [Database Indexing & Query Optimisation](docs/105-database-indexing.md) | B-tree, partial, covering, GIN indexes, EXPLAIN ANALYZE, N+1 fixes, and index maintenance |
| 106 | [Resilience4j](docs/106-resilience4j.md) | Circuit Breaker, Retry, Rate Limiter, Bulkhead, TimeLimiter, and Micrometer metrics |
| 107 | [Spring Cloud Gateway](docs/107-spring-cloud-gateway.md) | Routing, predicates, filters, rate limiting, circuit breaker, and canary deployments |
| 108 | [R2DBC — Reactive Relational Database](docs/108-r2dbc.md) | ReactiveCrudRepository, DatabaseClient, transactions, and Testcontainers integration |
| 109 | [Spring Authorization Server](docs/109-spring-authorization-server.md) | OAuth2/OIDC server, RegisteredClient, PKCE, token customisation, and persistent storage |
| 110 | [Spring Modulith](docs/110-spring-modulith.md) | Application modules, boundary enforcement, @ApplicationModuleListener, and module tests |
| 111 | [Virtual Threads Deep Dive](docs/111-virtual-threads.md) | Project Loom, mounting/unmounting, pinning, ScopedValue, and StructuredTaskScope |
| 112 | [Kafka Streams](docs/112-kafka-streams.md) | KStream, KTable, windowed aggregations, stream-table joins, and TopologyTestDriver |
| 113 | [Advanced Stream Collectors](docs/113-advanced-stream-collectors.md) | groupingBy, teeing, partitioningBy, flatMapping, custom Collector, and parallel streams |
| 114 | [Spring REST Docs](docs/114-spring-rest-docs.md) | Test-driven API docs, AsciiDoc snippets, request/response fields, and OpenAPI export |
| 115 | [Saga Pattern Deep Dive](docs/115-saga-pattern.md) | Choreography vs orchestration, compensating transactions, idempotency, and recovery |
| 116 | [Spring Data JPA Projections & Specifications](docs/116-jpa-projections-specifications.md) | Interface/DTO projections, dynamic projections, Specification<T>, and Querydsl |
| 117 | [JPA Advanced — Inheritance, Embeddables & Auditing](docs/117-jpa-advanced.md) | SINGLE_TABLE/JOINED, @Embedded, @ElementCollection, @CreatedDate, @Version, 2LC |
| 118 | [Functional Web (WebFlux.fn)](docs/118-functional-web.md) | RouterFunction, HandlerFunction, HandlerFilterFunction, predicates, and no-server testing |
| 119 | [Reactive Security](docs/119-reactive-security.md) | SecurityWebFilterChain, ReactiveUserDetailsService, ReactiveJwtDecoder, method security |
| 120 | [Spring Cloud Service Discovery](docs/120-service-discovery.md) | Eureka, Consul, @LoadBalanced, DiscoveryClient, and Kubernetes DNS |
| 121 | [Kafka Advanced](docs/121-kafka-advanced.md) | Exactly-once semantics, transactions, partition strategies, and consumer lag monitoring |
| 122 | [Distributed Locking](docs/122-distributed-locking.md) | Redisson RLock, Redis SETNX, PostgreSQL advisory locks, ShedLock, and fencing tokens |
| 123 | [Spring AI](docs/123-spring-ai.md) | ChatClient, prompt templates, structured output, function calling, RAG, and vector stores |
| 124 | [jOOQ](docs/124-jooq.md) | Type-safe SQL DSL, code generation, joins, CTEs, window functions, and batch operations |
| 125 | [Micrometer Observation API](docs/125-micrometer-observation.md) | Observation, low/high cardinality keys, @Observed, ObservationHandler, unified signals |
| 126 | [Apache Avro & Schema Registry](docs/126-avro-schema-registry.md) | Avro schemas, code generation, schema evolution, compatibility modes, and Confluent Registry |
| 127 | [Spring Data Cassandra](docs/127-spring-data-cassandra.md) | Wide-column NoSQL, partition/clustering keys, reactive repository, TTL, and consistency levels |
| 128 | [CompletableFuture Deep Dive](docs/128-completablefuture.md) | thenCompose, thenCombine, allOf, anyOf, timeouts, error recovery, and virtual thread comparison |
| 129 | [Spring Boot Custom Auto-Configuration](docs/129-custom-auto-configuration.md) | @AutoConfiguration, @Conditional annotations, custom starters, metadata, and testing |
| 130 | [Java NIO & Asynchronous I/O](docs/130-java-nio.md) | Buffers, channels, Selector, AsynchronousFileChannel, WatchService, and NIO.2 Files API |
| 131 | [Service Mesh & Istio](docs/131-service-mesh-istio.md) | Sidecar proxy, mTLS, VirtualService, DestinationRule, circuit breaking, and fault injection |
| 132 | [Axon Framework](docs/132-axon-framework.md) | CQRS/ES with Axon, aggregates, projections, sagas, command/query/event gateways |
| 133 | [Database Sharding & Partitioning](docs/133-database-sharding.md) | Hash/range sharding, consistent hashing, AbstractRoutingDataSource, and PostgreSQL partitioning |
| 134 | [Spring Integration](docs/134-spring-integration.md) | Enterprise integration patterns, MessageChannel, transformers, routers, aggregators, Kafka adapter |
| 135 | [gRPC Advanced](docs/135-grpc-advanced.md) | Server/client/bidirectional streaming, interceptors, deadlines, status codes, and cancellation |
| 136 | [Java Concurrency Utilities](docs/136-concurrency-utilities.md) | Semaphore, CountDownLatch, CyclicBarrier, Phaser, StampedLock, and Exchanger |
| 137 | [Spring Shell & CLI Tools](docs/137-spring-shell.md) | @ShellComponent, @ShellMethod, completion, availability conditions, and ANSI output |
| 138 | [Hibernate Envers](docs/138-hibernate-envers.md) | @Audited, custom revision entity, AuditReader queries, and RevisionRepository |
| 139 | [Spring Data REST](docs/139-spring-data-rest.md) | Auto-expose repos as HAL endpoints, projections, event handlers, validators, and resource processors |
| 140 | [Jackson Advanced](docs/140-jackson-advanced.md) | Custom serializers/deserializers, mixins, polymorphic types, @JsonView, and modules |
| 141 | [GC Tuning & JVM Diagnostics](docs/141-gc-tuning.md) | G1/ZGC/Shenandoah flags, GC log analysis, heap dumps, jcmd, and async-profiler flamegraphs |
| 142 | [Zero-Downtime Database Migrations](docs/142-zero-downtime-migrations.md) | Expand/contract pattern, safe NOT NULL columns, CONCURRENTLY indexes, and batched backfills |
| 143 | [Spring Web Services (SOAP)](docs/143-spring-web-services.md) | Contract-first WSDL/XSD, @Endpoint, JAXB marshalling, WebServiceTemplate, and WS-Security |
| 144 | [Java Agents & Bytecode Instrumentation](docs/144-java-agents.md) | premain/agentmain, ClassFileTransformer, ByteBuddy, dynamic attach, and OpenTelemetry agent |
| 145 | [Reactive Caching](docs/145-reactive-caching.md) | CacheMono/CacheFlux, Caffeine, ReactiveRedisTemplate, Mono.cache(), and cache invalidation |
| 146 | [Spring Events & ApplicationEventPublisher](docs/146-spring-events.md) | @EventListener, @TransactionalEventListener, async events, ordering, and event chaining |
| 147 | [Spring Data JDBC](docs/147-spring-data-jdbc.md) | Aggregate roots, @MappedCollection, @Embedded, JdbcClient, and lifecycle callbacks |
| 148 | [JMS & Spring JMS](docs/148-jms-spring-jms.md) | JmsTemplate, @JmsListener, topics vs queues, dead letter queues, and JMS transactions |
| 149 | [Reactive Testing — StepVerifier & WebTestClient](docs/149-reactive-testing.md) | StepVerifier, virtual time, TestPublisher, WebTestClient, and SSE stream assertions |
| 150 | [Spring Boot @ConfigurationProperties](docs/150-configuration-properties.md) | Typed config binding, nested objects, validation, records, and IDE metadata |
| 151 | [Spring Session](docs/151-spring-session.md) | Distributed sessions with Redis/JDBC, session events, security integration, and cookie config |
| 152 | [JVM Startup Optimization — AppCDS & CRaC](docs/152-jvm-startup-optimization.md) | Application CDS, AOT processing, Checkpoint/Restore, and startup time comparison |
| 153 | [Vavr — Functional Java](docs/153-vavr.md) | Option, Try, Either, Validated, Tuple, immutable collections, and pattern matching |
| 154 | [Apache Camel](docs/154-apache-camel.md) | Routes, Java DSL, content-based router, splitter/aggregator, Kafka, and error handling |
| 155 | [Java 22–25 New Language Features](docs/155-java-22-25-features.md) | Sequenced Collections, unnamed patterns, String Templates, Gatherers, FFM API, and Value Objects |
| 156 | [Spring Cloud Stream](docs/156-spring-cloud-stream.md) | Function/Consumer/Supplier beans, Kafka/RabbitMQ binders, StreamBridge, DLQ, and partitioning |
| 157 | [Spring MVC Exception Handling & Problem Details](docs/157-exception-handling-problem-details.md) | @ExceptionHandler, @ControllerAdvice, ProblemDetail RFC 9457, and validation error responses |
| 158 | [Testcontainers Deep Dive](docs/158-testcontainers-deep-dive.md) | @ServiceConnection, reusable containers, WireMock, LocalStack, and dynamic property source |
| 159 | [Java Flight Recorder (JFR) Deep Dive](docs/159-java-flight-recorder.md) | Custom JFR events, JFR streaming API, jcmd recording, JDK Mission Control, and continuous profiling |
| 160 | [Spring Cloud Function & Serverless](docs/160-spring-cloud-function-serverless.md) | Function beans, AWS Lambda adapter, routing, GraalVM native, and cold start strategies |
| 161 | [Kafka Connect & Debezium CDC](docs/161-kafka-connect-debezium.md) | Source/sink connectors, Debezium PostgreSQL CDC, SMTs, JDBC sink, and connector management |
| 162 | [Spring Security Method Security](docs/162-spring-security-method-security.md) | @PreAuthorize, @PostAuthorize, @PreFilter/@PostFilter, PermissionEvaluator, and reactive security |
| 163 | [Apache POI — Excel & Document Generation](docs/163-apache-poi.md) | XSSF workbooks, SXSSF streaming, reading uploads, Word docs, and HTTP file download |
| 164 | [OpenAPI Code Generation](docs/164-openapi-code-generation.md) | Contract-first REST, openapi-generator plugin, server stubs, client SDKs, and CI validation |
| 165 | [Read Replicas & Multi-DataSource](docs/165-read-replicas-multi-datasource.md) | AbstractRoutingDataSource, LazyConnectionDataSourceProxy, replica routing, and lag monitoring |
| 166 | [Spring Boot Testing Slices](docs/166-spring-boot-testing-slices.md) | @WebMvcTest, @DataJpaTest, @JsonTest, @RestClientTest, @DataMongoTest, and @WebFluxTest |
| 167 | [Helm Charts](docs/167-helm-charts.md) | Chart structure, values.yaml, templates, _helpers.tpl, HPA, and helm upgrade --install |
| 168 | [Spring Security OAuth2 Client](docs/168-spring-security-oauth2-client.md) | Client credentials, token relay, OAuth2AuthorizedClientManager, and reactive client |
| 169 | [OpenTelemetry Manual Instrumentation](docs/169-opentelemetry-manual-instrumentation.md) | Custom spans, baggage, OTel Metrics API, async context propagation, and @WithSpan |
| 170 | [Temporal Workflow Engine](docs/170-temporal-workflow.md) | @WorkflowInterface, @ActivityInterface, signals, queries, timers, and child workflows |
| 171 | [Spring Boot Profiles Deep Dive](docs/171-spring-boot-profiles.md) | @Profile, profile groups, multi-document YAML, @ConditionalOnProperty, and test profiles |
| 172 | [Spring MVC Interceptors & Argument Resolvers](docs/172-spring-mvc-interceptors-argument-resolvers.md) | HandlerInterceptor, @CurrentUser resolver, WebMvcConfigurer, and custom converters |
| 173 | [Java Memory Leak Analysis](docs/173-java-memory-leak-analysis.md) | Heap dumps, Eclipse MAT, dominator tree, OQL, and common leak patterns |
| 174 | [AsyncAPI Specification](docs/174-asyncapi.md) | Event-driven API contracts, channels, message schemas, Kafka bindings, and code generation |
| 175 | [Gradle Advanced](docs/175-gradle-advanced.md) | Multi-module builds, version catalogs, convention plugins, build cache, and custom tasks |
| 176 | [PDF Generation](docs/176-pdf-generation.md) | Apache PDFBox, OpenPDF, creating tables, filling forms, merging, and HTTP download |
| 177 | [JWT Deep Dive](docs/177-jwt-deep-dive.md) | JWT structure, RS256/ES256, JWKS endpoint, refresh tokens, rotation, and Redis revocation |
| 178 | [JVM Thread Dump Analysis](docs/178-thread-dump-analysis.md) | jstack, thread states, deadlock detection, ThreadMXBean, and common patterns |
| 179 | [UUID Strategies & Distributed IDs](docs/179-uuid-strategies.md) | UUID v4/v7, ULID, TSID, NanoID, and B-tree index fragmentation implications |
| 180 | [HTTP Caching — ETag & Cache-Control](docs/180-http-caching.md) | Cache-Control directives, ETag, conditional GET, ShallowEtagHeaderFilter, and conditional PUT |
| 181 | [Chaos Engineering with Spring](docs/181-chaos-engineering.md) | Chaos Monkey for Spring Boot, latency/exception/kill assaults, and CI resilience validation |
| 182 | [Reactor Kafka — Reactive Kafka Client](docs/182-reactor-kafka.md) | KafkaSender, KafkaReceiver, backpressure, manual offset commit, and exactly-once transactions |
| 183 | [API Idempotency Patterns](docs/183-api-idempotency.md) | Idempotency-Key header, Redis token store, conditional PUT with ETag, and client retry strategy |
| 184 | [JVM Ergonomics & Container Awareness](docs/184-jvm-ergonomics.md) | UseContainerSupport, MaxRAMPercentage, GC selection heuristics, and Kubernetes resource limits |
| 185 | [Apache Pulsar with Spring Pulsar](docs/185-apache-pulsar.md) | PulsarTemplate, @PulsarListener, subscription types, schema support, DLQ, and reactive client |
| 186 | [ArchUnit — Architecture Testing](docs/186-archunit.md) | Enforce layer rules, naming conventions, cycle detection, and custom conditions in tests |
| 187 | [Spring Data Neo4j](docs/187-spring-data-neo4j.md) | Graph database with @Node, @Relationship, Cypher queries, path traversal, and recommendations |
| 188 | [RSocket](docs/188-rsocket.md) | Request-response, fire-and-forget, request-stream, and bidirectional channel with Spring Boot |
| 189 | [LangChain4j — LLM Integration](docs/189-langchain4j.md) | @AiService, prompt templates, tool calling, RAG pipeline, structured output, and streaming |
| 190 | [Java Annotation Processing (APT)](docs/190-annotation-processing.md) | AbstractProcessor, Filer, Messager, code generation, and compile-time validation |
| 191 | [Quartz Scheduler Deep Dive](docs/191-quartz-scheduler.md) | JDBC JobStore, clustering, CronTrigger, JobDataMap, misfire handling, and job listeners |
| 192 | [WebAuthn & Passkeys](docs/192-webauthn-passkeys.md) | FIDO2 passwordless auth, Spring Security 6.3+ WebAuthn, registration and authentication flows |
| 193 | [Spring Security SAML2](docs/193-spring-security-saml2.md) | Enterprise SSO with Okta/Azure AD, SP metadata, attribute extraction, and dynamic registration |
| 194 | [Data Masking & GDPR Compliance](docs/194-data-masking-gdpr.md) | PII masking in logs, column encryption, right-to-erasure, pseudonymisation, and audit trails |
| 195 | [Flyway Advanced](docs/195-flyway-advanced.md) | Java migrations, callbacks, repair, baseline, dry-run, placeholders, and CI validation |
| 196 | [Spring Transaction Management Deep Dive](docs/196-spring-transactions.md) | Propagation levels, isolation, rollback rules, self-invocation pitfall, and TransactionTemplate |
| 197 | [@Async & Spring Task Execution](docs/197-async-spring-task-execution.md) | @EnableAsync, ThreadPoolTaskExecutor, CompletableFuture, fan-out, and async error handling |
| 198 | [Weak, Soft & Phantom References](docs/198-weak-soft-phantom-references.md) | SoftReference, WeakReference, PhantomReference, ReferenceQueue, WeakHashMap, and Cleaner |
| 199 | [Spring State Machine](docs/199-spring-state-machine.md) | States, transitions, guards, actions, JPA persistence, listeners, and StateMachineTestPlan |
| 200 | [Apache Commons & Google Guava](docs/200-commons-guava.md) | StringUtils, StopWatch, CollectionUtils, ImmutableList, Multimap, Table, Preconditions, RangeMap |
| 201 | [Java Process API](docs/201-java-process-api.md) | ProcessBuilder, Process, ProcessHandle, I/O redirection, pipeline, and subprocess management |
| 202 | [mTLS & Certificate Management](docs/202-mtls-certificates.md) | Mutual TLS, keystores, truststores, PKCS12, Spring Boot TLS config, and certificate rotation |
| 203 | [Spring LDAP](docs/203-spring-ldap.md) | LdapTemplate, @Entry ODM, LdapRepository, Spring Security LDAP auth, and Active Directory |
| 204 | [Micrometer Metrics Deep Dive](docs/204-micrometer-metrics.md) | Counter, Gauge, Timer, DistributionSummary, percentile histograms, SLO buckets, and MeterFilter |
| 205 | [QR Code & Barcode Generation](docs/205-qr-barcode-generation.md) | ZXing QR codes, Code128, EAN-13, PDF417, HTTP image responses, decoding, and TOTP setup |
| 206 | [JMH Microbenchmarking](docs/206-jmh-microbenchmarking.md) | @Benchmark, benchmark modes, @State, @Fork, Blackhole, @Param, and common JIT pitfalls |
| 207 | [Spring Cache Abstraction](docs/207-spring-cache-abstraction.md) | @Cacheable, @CacheEvict, @CachePut, @Caching, CacheManager, Redis TTL config, and KeyGenerator |
| 208 | [Java Image Processing](docs/208-java-image-processing.md) | BufferedImage, ImageIO, Graphics2D, scaling, cropping, watermarking, rotation, and Thumbnailator |
| 209 | [Strangler Fig & Anti-Corruption Layer](docs/209-strangler-fig.md) | Incremental legacy migration, facade routing, model translation, CDC sync, and canary rollout |
| 210 | [Apache Tika — Document Parsing](docs/210-apache-tika.md) | MIME detection, text and metadata extraction, Office/PDF/HTML parsers, and archive recursion |
| 211 | [DynamoDB with Spring](docs/211-dynamodb.md) | Enhanced client, @DynamoDbBean, GSI queries, batch operations, transactions, and converters |
| 212 | [Spring Data Elasticsearch Deep Dive](docs/212-spring-data-elasticsearch.md) | Full-text search, aggregations, highlights, nested objects, bulk indexing, and alias reindex |
| 213 | [Java Scripting API & GraalVM Polyglot](docs/213-java-scripting-graalvm.md) | JSR-223 ScriptEngine, GraalVM Context, bidirectional interop, rule engine, and sandboxing |
| 214 | [Continuous Profiling with async-profiler](docs/214-continuous-profiling.md) | CPU/alloc/wall/lock profiling, flame graphs, JFR, Pyroscope integration, and pitfall checklist |
| 215 | [Protocol Buffers Deep Dive](docs/215-protobuf-deep-dive.md) | Proto3 schema, field numbers, oneof/map, serialization, JSON interop, schema evolution, Kafka |
| 216 | [Spring Boot Admin](docs/216-spring-boot-admin.md) | Admin UI over Actuator, client registration, health dashboard, log-level management, notifications |
| 217 | [Java Money (JSR 354 / Moneta)](docs/217-java-money.md) | MonetaryAmount, CurrencyUnit, rounding, MonetaryOperator, Jackson/JPA integration, currency conversion |
| 218 | [OpenTelemetry Collector](docs/218-opentelemetry-collector.md) | Receivers, processors, exporters, tail sampling, attribute scrubbing, K8s DaemonSet deployment |
| 219 | [Spring Security HTTP Security Headers](docs/219-http-security-headers.md) | CSP, HSTS, X-Frame-Options, Permissions-Policy, COOP/COEP, nonce-based CSP for SPAs |
| 220 | [Spring Cloud Kubernetes](docs/220-spring-cloud-kubernetes.md) | ConfigMap/Secret injection, service discovery, RBAC, config reload, leader election, probes |
| 221 | [Lock-Free Concurrency](docs/221-lock-free-concurrency.md) | CAS, AtomicReference, LongAdder, VarHandle, ABA problem, Treiber Stack, Michael-Scott Queue |
| 222 | [Jasypt — Encrypted Properties](docs/222-jasypt-encrypted-properties.md) | ENC() wrapper, master key via env var, PBEWITHHMACSHA512ANDAES_256, CI/CD integration |
| 223 | [GraphQL Subscriptions](docs/223-graphql-subscriptions.md) | @SubscriptionMapping, Flux<T>, WebSocket and SSE transport, Kafka-backed streams, auth |
| 224 | [Java Serialization Security](docs/224-java-serialization-security.md) | Gadget chains, ObjectInputFilter allowlist, HMAC signing, safe alternatives (JSON/Protobuf) |
| 225 | [Kafka AdminClient](docs/225-kafka-admin-client.md) | Topic creation/deletion, partition management, consumer lag, offset reset, ACLs, config changes |
