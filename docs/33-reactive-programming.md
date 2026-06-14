# Reactive Programming

[← Back to README](../README.md)

---

**Reactive programming** models data as asynchronous streams and processes them without blocking threads. Instead of "call a method and wait", you compose a pipeline of operators and subscribe to results when they arrive.

Java's reactive ecosystem is built on the **Reactive Streams** specification (`java.util.concurrent.Flow` in the JDK) and implemented by **Project Reactor** (`Mono` / `Flux`), which is the foundation of Spring WebFlux.

```mermaid
flowchart LR
    pub["Publisher\n(produces data)"]
    sub["Subscriber\n(consumes data)"]
    op["Operators\nmap, filter,\nflatMap, zip"]

    pub -->|stream| op -->|transformed stream| sub
    sub -->|backpressure: request(n)| pub
```

---

## Core Concepts

| Concept | Meaning |
|---------|---------|
| **Publisher** | Source of data — emits items, errors, or completion |
| **Subscriber** | Consumes items from a Publisher |
| **Subscription** | Contract between Publisher and Subscriber; controls demand |
| **Backpressure** | Subscriber signals how many items it can handle (prevents overflow) |
| **Mono** | Publisher of 0 or 1 item |
| **Flux** | Publisher of 0 to N items |

---

## Project Reactor — Maven

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.6.7</version>
</dependency>

<!-- for testing -->
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <version>3.6.7</version>
    <scope>test</scope>
</dependency>
```

---

## Mono — 0 or 1 Item

```java
import reactor.core.publisher.Mono;

// create
Mono<String> hello    = Mono.just("Hello");
Mono<String> empty    = Mono.empty();
Mono<String> error    = Mono.error(new RuntimeException("oops"));
Mono<String> deferred = Mono.fromSupplier(() -> expensiveCall());

// transform
Mono<Integer> length = hello
    .map(String::length)         // 5
    .filter(n -> n > 3);         // passes — 5 > 3

// subscribe
hello.subscribe(
    value -> System.out.println("Got: " + value),   // onNext
    err   -> System.err.println("Error: " + err),   // onError
    ()    -> System.out.println("Done")              // onComplete
);

// block (use only in tests or main — never in a reactive pipeline)
String result = hello.block();
```

---

## Flux — 0 to N Items

```java
import reactor.core.publisher.Flux;

// create
Flux<Integer> numbers = Flux.just(1, 2, 3, 4, 5);
Flux<Integer> range   = Flux.range(1, 10);         // 1..10
Flux<String>  fromList= Flux.fromIterable(List.of("a", "b", "c"));
Flux<Long>    ticks   = Flux.interval(Duration.ofMillis(500));  // 0, 1, 2, ... every 500ms

// subscribe
numbers.subscribe(System.out::println);  // 1 2 3 4 5

// transform with operators
Flux<String> result = Flux.range(1, 5)
    .filter(n  -> n % 2 == 0)           // 2, 4
    .map(n     -> "Item " + n)           // "Item 2", "Item 4"
    .take(3);                            // at most 3 (here: 2)

result.subscribe(System.out::println);
// Item 2
// Item 4
```

---

## Key Operators

### Transforming

```java
// map — transform each item (1-to-1)
Flux.just("hello", "world")
    .map(String::toUpperCase)
    .subscribe(System.out::println);  // HELLO, WORLD

// flatMap — transform each item into a Publisher, then merge (async)
Flux.just("user1", "user2")
    .flatMap(id -> fetchUserAsync(id))   // returns Mono<User>
    .subscribe(user -> System.out.println(user.getName()));

// concatMap — like flatMap but preserves order
Flux.just("a", "b", "c")
    .concatMap(s -> Mono.just(s + "!").delayElement(Duration.ofMillis(100)))
    .subscribe(System.out::println);  // a!, b!, c! in order
```

### Filtering

```java
Flux.range(1, 10)
    .filter(n -> n % 2 == 0)    // 2,4,6,8,10
    .take(3)                     // 2,4,6
    .skip(1)                     // 4,6
    .subscribe(System.out::println);
```

### Combining

```java
// merge — interleave two Flux (by arrival time)
Flux<String> merged = Flux.merge(
    Flux.just("A", "B").delayElements(Duration.ofMillis(100)),
    Flux.just("1", "2").delayElements(Duration.ofMillis(150))
);

// zip — pair elements from multiple sources
Flux.zip(Flux.just("Alice", "Bob"), Flux.just(30, 25))
    .subscribe(tuple -> System.out.println(tuple.getT1() + " is " + tuple.getT2()));
// Alice is 30
// Bob is 25

// concat — append second after first completes
Flux<String> all = Flux.concat(Flux.just("A", "B"), Flux.just("C", "D"));
```

### Collecting

```java
// collect into a List
Mono<List<Integer>> list = Flux.range(1, 5).collectList();

// group by
Flux.just("apple", "avocado", "banana", "blueberry")
    .groupBy(s -> s.charAt(0))
    .flatMap(group -> group.collectList()
        .map(items -> group.key() + ": " + items))
    .subscribe(System.out::println);
// a: [apple, avocado]
// b: [banana, blueberry]

// reduce
Mono<Integer> sum = Flux.range(1, 5).reduce(0, Integer::sum);
```

---

## Error Handling

```java
Flux.just(1, 2, 0, 3)
    .map(n -> 10 / n)

    // replace error with fallback value
    .onErrorReturn(-1)

    // switch to fallback publisher on error
    .onErrorResume(e -> Flux.just(0, 0))

    // continue after error, skipping the bad item
    // .onErrorContinue((e, item) -> log.warn("Skipped {}", item))

    // retry on error
    .retry(3)

    .subscribe(
        System.out::println,
        err -> System.err.println("Unhandled: " + err)
    );
```

---

## Schedulers — Threading

By default Reactor is single-threaded. Use `Schedulers` to control execution:

```java
import reactor.core.scheduler.Schedulers;

Flux.range(1, 5)
    .publishOn(Schedulers.parallel())     // downstream runs on parallel pool
    .map(n -> n * 2)
    .subscribeOn(Schedulers.boundedElastic())  // subscription/source on elastic pool
    .subscribe(System.out::println);

// common schedulers
Schedulers.single()          // single reusable thread
Schedulers.parallel()        // CPU-bound work (thread count = CPU cores)
Schedulers.boundedElastic()  // I/O-bound work (grows as needed, bounded)
Schedulers.immediate()       // current thread (default)
```

---

## Backpressure

```java
// subscriber controls how many items it receives at a time
Flux.range(1, 1000)
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(10);  // request first 10
        }

        @Override
        protected void hookOnNext(Integer value) {
            System.out.println(value);
            if (value % 10 == 0) {
                request(10);  // request next batch
            }
        }
    });
```

---

## Testing with StepVerifier

```java
import reactor.test.StepVerifier;

@Test
void testFlux() {
    Flux<Integer> flux = Flux.range(1, 3);

    StepVerifier.create(flux)
        .expectNext(1)
        .expectNext(2)
        .expectNext(3)
        .expectComplete()
        .verify();
}

@Test
void testMono() {
    Mono<String> mono = Mono.just("hello").map(String::toUpperCase);

    StepVerifier.create(mono)
        .expectNext("HELLO")
        .verifyComplete();
}

@Test
void testError() {
    Mono<String> mono = Mono.error(new IllegalArgumentException("bad"));

    StepVerifier.create(mono)
        .expectErrorMatches(e -> e instanceof IllegalArgumentException)
        .verify();
}
```

---

## Spring WebFlux

Spring WebFlux is the reactive web framework — it replaces blocking Spring MVC with a non-blocking stack.

```xml
<!-- instead of spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserRepository repo;  // extends ReactiveCrudRepository

    public UserController(UserRepository repo) { this.repo = repo; }

    @GetMapping
    public Flux<User> getAll() {
        return repo.findAll();  // returns Flux
    }

    @GetMapping("/{id}")
    public Mono<ResponseEntity<User>> getById(@PathVariable String id) {
        return repo.findById(id)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<User> create(@RequestBody Mono<User> user) {
        return user.flatMap(repo::save);
    }
}
```

---

## Reactive vs Blocking — When to Use Which

| Reactive | Blocking (Spring MVC) |
|----------|-----------------------|
| Many concurrent connections with I/O waits | Moderate load, simple CRUD |
| Streaming data (SSE, WebSocket) | Request/response pattern |
| Microservices calling other services | Relational DB with JDBC (reactive drivers needed for reactive) |
| Low latency under high concurrency | Easier to reason about, more library support |

> Don't use reactive just because it sounds modern. If your bottleneck isn't I/O concurrency, the extra complexity isn't worth it.

---

## Reactive Programming Summary

| Concept | Type / Method |
|---------|--------------|
| 0 or 1 item | `Mono<T>` |
| 0..N items | `Flux<T>` |
| Transform 1-to-1 | `.map()` |
| Transform 1-to-many (async) | `.flatMap()` |
| Filter | `.filter()` |
| Combine streams | `.merge()`, `.zip()`, `.concat()` |
| Error fallback | `.onErrorReturn()`, `.onErrorResume()` |
| Retry | `.retry(n)` |
| Switch thread | `.publishOn()`, `.subscribeOn()` |
| Collect | `.collectList()`, `.reduce()` |
| Test | `StepVerifier.create(...)` |
| Spring reactive web | `spring-boot-starter-webflux` |

---

[← Back to README](../README.md)
