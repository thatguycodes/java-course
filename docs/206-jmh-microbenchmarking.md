# JMH Microbenchmarking Deep Dive

[← Back to README](../README.md)

---

**JMH** (Java Microbenchmark Harness) is the official tool for writing reliable Java microbenchmarks. Writing a correct benchmark is harder than it looks — the JIT compiler inlines, unrolls, and eliminates "dead" code, making naive measurements wildly inaccurate. JMH handles JVM warmup, forked JVMs, and dead-code prevention via `Blackhole`, giving you trustworthy numbers.

```mermaid
flowchart LR
    bench["@Benchmark method"]
    fork["Forked JVM\n(clean classloader)"]
    warmup["Warmup iterations\n(JIT settles)"]
    measure["Measurement iterations\n(record results)"]
    report["Results\n(ops/s, ns/op, percentiles)"]

    bench --> fork --> warmup --> measure --> report
```

---

## Dependency (Maven)

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
    <scope>test</scope>
</dependency>
```

```xml
<!-- Maven plugin to run benchmarks -->
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>exec-maven-plugin</artifactId>
    <configuration>
        <mainClass>org.openjdk.jmh.Main</mainClass>
        <arguments>
            <argument>.*StringBenchmark.*</argument>
        </arguments>
    </configuration>
</plugin>
```

---

## Basic Benchmark

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Thread)
@Fork(value = 2, warmups = 1)
@Warmup(iterations = 5, time = 1)
@Measurement(iterations = 10, time = 1)
public class StringConcatBenchmark {

    private static final int N = 1000;

    @Benchmark
    public String stringPlus() {
        String result = "";
        for (int i = 0; i < N; i++) {
            result += i;            // creates N intermediate Strings
        }
        return result;
    }

    @Benchmark
    public String stringBuilder() {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < N; i++) {
            sb.append(i);
        }
        return sb.toString();
    }

    @Benchmark
    public String stringJoiner() {
        StringJoiner sj = new StringJoiner("");
        for (int i = 0; i < N; i++) {
            sj.add(Integer.toString(i));
        }
        return sj.toString();
    }
}
```

```bash
# Run with JMH runner (after mvn package)
java -jar target/benchmarks.jar StringConcatBenchmark

# Sample output:
# Benchmark                          Mode  Cnt      Score     Error  Units
# StringConcatBenchmark.stringPlus   avgt   20  42315.123 ± 812.4  ns/op
# StringConcatBenchmark.stringBuilder avgt  20    123.456 ±   2.1  ns/op
```

---

## Benchmark Modes

```java
// Throughput — operations per second (higher is better)
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
public long throughputMode() { ... }

// AverageTime — average time per operation (lower is better)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public long avgTimeMode() { ... }

// SampleTime — distribution of time per operation (gives percentiles)
@BenchmarkMode(Mode.SampleTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public long sampleTimeMode() { ... }

// SingleShotTime — one cold invocation (no warmup; measures startup cost)
@BenchmarkMode(Mode.SingleShotTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public long coldStartMode() { ... }

// All — run all modes
@BenchmarkMode(Mode.All)
public long allModes() { ... }
```

---

## State and Setup

```java
@State(Scope.Benchmark)   // one instance shared across all threads
public class SharedState {
    public List<Integer> numbers;

    @Setup(Level.Trial)   // once before all iterations
    public void setup() {
        numbers = IntStream.rangeClosed(1, 10_000)
            .boxed()
            .collect(Collectors.toList());
    }

    @TearDown(Level.Trial)
    public void tearDown() {
        numbers = null;
    }
}

@State(Scope.Thread)      // one instance per thread
public class ThreadState {
    public Random rng;

    @Setup(Level.Iteration)   // before each measurement iteration
    public void setup() {
        rng = new Random(42);
    }
}

// Inject state into benchmark methods
@Benchmark
public int findMax(SharedState shared, ThreadState thread) {
    int idx = thread.rng.nextInt(shared.numbers.size());
    return shared.numbers.get(idx);
}
```

---

## Blackhole — Preventing Dead Code Elimination

```java
// WRONG: JIT may eliminate computations whose result isn't used
@Benchmark
public void wrongBenchmark() {
    Math.sqrt(12345.6789);   // result discarded → JIT skips it
}

// RIGHT option 1: return the result
@Benchmark
public double returnResult() {
    return Math.sqrt(12345.6789);   // JMH consumes the return value
}

// RIGHT option 2: consume via Blackhole
@Benchmark
public void blackholeBenchmark(Blackhole bh) {
    bh.consume(Math.sqrt(12345.6789));
    bh.consume(Math.log(12345.6789));
    bh.consume(Integer.bitCount(0xDEADBEEF));
}
```

---

## Parameterized Benchmarks

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Benchmark)
public class CollectionBenchmark {

    @Param({"10", "100", "1000", "10000"})
    private int size;

    private List<Integer> arrayList;
    private List<Integer> linkedList;

    @Setup
    public void setup() {
        arrayList  = new ArrayList<>(IntStream.range(0, size).boxed().toList());
        linkedList = new LinkedList<>(arrayList);
    }

    @Benchmark
    public int arrayListGet() {
        return arrayList.get(size / 2);
    }

    @Benchmark
    public int linkedListGet() {
        return linkedList.get(size / 2);
    }
}

// Output — shows results for each param combination:
// CollectionBenchmark.arrayListGet   size=10     avgt  3.1 ± 0.1 ns/op
// CollectionBenchmark.arrayListGet   size=10000  avgt  3.2 ± 0.1 ns/op
// CollectionBenchmark.linkedListGet  size=10     avgt  8.1 ± 0.2 ns/op
// CollectionBenchmark.linkedListGet  size=10000  avgt  52341.2 ± 200 ns/op
```

---

## Fork and Warmup Configuration

```java
@Fork(
    value = 3,             // run in 3 separate forked JVM processes
    warmups = 1,           // 1 warmup fork (discarded)
    jvmArgs = {"-Xmx512m", "-XX:+UseG1GC"}
)
@Warmup(
    iterations = 5,        // 5 warmup iterations per fork
    time = 1,              // each iteration lasts 1 second
    timeUnit = TimeUnit.SECONDS
)
@Measurement(
    iterations = 10,       // 10 measurement iterations
    time = 2
)
public class SerializationBenchmark {

    @Benchmark
    public byte[] jacksonSerialize(JacksonState state) throws Exception {
        return state.mapper.writeValueAsBytes(state.order);
    }

    @Benchmark
    public byte[] protoSerialize(ProtoState state) {
        return state.orderProto.toByteArray();
    }
}
```

---

## Running Programmatically

```java
public static void main(String[] args) throws RunnerException {
    Options opts = new OptionsBuilder()
        .include(StringConcatBenchmark.class.getSimpleName())
        .forks(2)
        .warmupIterations(3)
        .measurementIterations(5)
        .mode(Mode.AverageTime)
        .timeUnit(TimeUnit.NANOSECONDS)
        .resultFormat(ResultFormatType.JSON)
        .result("benchmark-results.json")
        .build();

    new Runner(opts).run();
}
```

---

## Common Pitfalls

```java
// PITFALL 1: Constant folding — JIT computes result at compile time
@Benchmark
public double constantFold() {
    return Math.sqrt(4);    // JIT may fold to 2.0 at compile time
}
// FIX: use a @State field
@State(Scope.Thread)
public static class Data { double x = 4.0; }

@Benchmark
public double noFold(Data d) {
    return Math.sqrt(d.x);
}

// PITFALL 2: Loop unrolling / superword optimisation skews results
@Benchmark
public int sumArray(ArrayState s) {
    int sum = 0;
    for (int x : s.array) sum += x;   // JIT may vectorise this
    return sum;
}
// Accept it or use @Fork(jvmArgs = "-XX:-SuperWordMaxVectorSize=0")

// PITFALL 3: Sharing mutable state between threads
@State(Scope.Benchmark)   // shared → race condition in @Benchmark
public static class BadState { int counter = 0; }

@Benchmark
public int badIncrement(BadState s) { return s.counter++; }  // WRONG
// FIX: use Scope.Thread for mutable state
```

---

## JMH Summary

| Concept | Detail |
|---------|--------|
| `@Benchmark` | Marks a method as a benchmark; JMH generates harness code around it |
| `Mode.AverageTime` | Average nanoseconds per operation — most common mode |
| `Mode.Throughput` | Operations per second — use when you care about max throughput |
| `Mode.SampleTime` | Records latency distribution — reveals p99 outliers |
| `@State(Scope.Thread)` | One state instance per benchmark thread; safe for mutable state |
| `@State(Scope.Benchmark)` | Shared across all threads; use for read-only input data |
| `@Setup(Level.Trial)` | Runs once before all iterations — expensive setup |
| `Blackhole.consume(x)` | Prevents JIT from eliminating computed values |
| `@Fork(2)` | Run in 2 separate JVM processes — eliminates JVM-startup bias |
| `@Param({"10","100"})` | Parameterise benchmarks to test multiple input sizes |
| Constant folding | JIT computes constant expressions at compile time — use `@State` fields to prevent |

---

[← Back to README](../README.md)
