# Database Sharding & Partitioning

[← Back to README](../README.md)

---

**Sharding** splits a large dataset horizontally across multiple database instances (shards), each owning a subset of the data. **Partitioning** splits a large table within a single database instance. Both techniques allow datasets and write throughput to scale beyond what a single machine can handle.

```mermaid
flowchart LR
    app["Application"]
    router["Shard Router\n(AbstractRoutingDataSource)"]
    s1["Shard 0\nPostgreSQL\ncustomers A–M"]
    s2["Shard 1\nPostgreSQL\ncustomers N–Z"]

    app --> router
    router -->|hash(customerId) % 2 == 0| s1
    router -->|hash(customerId) % 2 == 1| s2
```

---

## Sharding Strategies

### Range-Based Sharding

```
Shard 0: customer_id  0 000 000 – 3 333 333
Shard 1: customer_id  3 333 334 – 6 666 666
Shard 2: customer_id  6 666 667 – 9 999 999
```

- **Pro:** Range queries stay on one shard
- **Con:** Hot spots if new data accumulates at the high end

### Hash-Based Sharding

```java
int shardIndex = Math.abs(customerId.hashCode()) % NUM_SHARDS;
```

- **Pro:** Even distribution
- **Con:** Range queries require all shards; resharding is expensive

### Directory-Based Sharding

```java
// Lookup table maps entity ID → shard
String shard = shardDirectory.lookup(customerId);
```

- **Pro:** Flexible; can rebalance without hashing
- **Con:** Directory is a bottleneck/SPOF

---

## AbstractRoutingDataSource — Multi-Shard in Spring

```java
@Configuration
public class ShardDataSourceConfig {

    @Bean
    public DataSource shardedDataSource() {
        Map<Object, Object> shards = new HashMap<>();

        shards.put("shard-0", buildDataSource("jdbc:postgresql://shard0:5432/orders"));
        shards.put("shard-1", buildDataSource("jdbc:postgresql://shard1:5432/orders"));
        shards.put("shard-2", buildDataSource("jdbc:postgresql://shard2:5432/orders"));

        ShardRoutingDataSource routing = new ShardRoutingDataSource();
        routing.setTargetDataSources(shards);
        routing.setDefaultTargetDataSource(shards.get("shard-0"));
        return routing;
    }

    private DataSource buildDataSource(String url) {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(url);
        config.setUsername("app");
        config.setPassword(System.getenv("DB_PASSWORD"));
        config.setMaximumPoolSize(10);
        return new HikariDataSource(config);
    }
}
```

```java
// Thread-local shard key
public class ShardContext {
    private static final ThreadLocal<String> CURRENT_SHARD = new ThreadLocal<>();

    public static void set(String shard) { CURRENT_SHARD.set(shard); }
    public static String get()           { return CURRENT_SHARD.get(); }
    public static void clear()           { CURRENT_SHARD.remove(); }
}

// Routing data source reads the thread-local
public class ShardRoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return ShardContext.get();
    }
}
```

```java
// Service resolves shard before DB call
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ShardResolver shardResolver;
    private final OrderRepository orderRepo;

    public Order placeOrder(PlaceOrderCommand cmd) {
        String shard = shardResolver.resolve(cmd.customerId());
        ShardContext.set(shard);
        try {
            return orderRepo.save(Order.create(cmd));
        } finally {
            ShardContext.clear();
        }
    }
}

@Component
public class ShardResolver {

    private static final int NUM_SHARDS = 3;

    public String resolve(UUID customerId) {
        int index = Math.abs(customerId.hashCode()) % NUM_SHARDS;
        return "shard-" + index;
    }
}
```

---

## Consistent Hashing

Consistent hashing minimises remapping when shards are added or removed:

```java
public class ConsistentHashRing {

    private final NavigableMap<Long, String> ring = new TreeMap<>();
    private final int virtualNodes;

    public ConsistentHashRing(List<String> shards, int virtualNodes) {
        this.virtualNodes = virtualNodes;
        shards.forEach(this::addShard);
    }

    public void addShard(String shard) {
        for (int i = 0; i < virtualNodes; i++) {
            long hash = hash(shard + "#" + i);
            ring.put(hash, shard);
        }
    }

    public void removeShard(String shard) {
        for (int i = 0; i < virtualNodes; i++) {
            ring.remove(hash(shard + "#" + i));
        }
    }

    public String getShard(UUID key) {
        if (ring.isEmpty()) throw new IllegalStateException("No shards available");
        long hash = hash(key.toString());
        Map.Entry<Long, String> entry = ring.ceilingEntry(hash);
        return (entry != null ? entry : ring.firstEntry()).getValue();
    }

    private long hash(String key) {
        return Hashing.murmur3_128().hashString(key, StandardCharsets.UTF_8).asLong();
    }
}
```

---

## PostgreSQL Table Partitioning

Partitioning splits a large table within a single PostgreSQL instance. The query planner prunes irrelevant partitions.

### Range Partitioning (by date)

```sql
-- Parent table
CREATE TABLE orders (
    id          UUID        NOT NULL,
    customer_id UUID        NOT NULL,
    status      TEXT        NOT NULL,
    total       NUMERIC     NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL
) PARTITION BY RANGE (created_at);

-- Monthly partitions
CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Default partition catches anything unmatched
CREATE TABLE orders_default PARTITION OF orders DEFAULT;

-- Index on each partition
CREATE INDEX ON orders_2024_01 (customer_id);
CREATE INDEX ON orders_2024_02 (customer_id);
```

### Hash Partitioning

```sql
CREATE TABLE orders (
    id          UUID PRIMARY KEY,
    customer_id UUID NOT NULL,
    total       NUMERIC NOT NULL
) PARTITION BY HASH (customer_id);

CREATE TABLE orders_p0 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE orders_p1 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE orders_p2 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE orders_p3 PARTITION OF orders
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

### Automatic Partition Creation

```java
@Scheduled(cron = "0 0 1 1 * *")   // first day of every month
@Transactional
public void createNextMonthPartition() {
    YearMonth next = YearMonth.now().plusMonths(1);
    String partitionName = "orders_" + next.format(DateTimeFormatter.ofPattern("yyyy_MM"));
    String from = next.atDay(1).toString();
    String to   = next.plusMonths(1).atDay(1).toString();

    jdbc.execute("""
        CREATE TABLE IF NOT EXISTS %s PARTITION OF orders
        FOR VALUES FROM ('%s') TO ('%s')
        """.formatted(partitionName, from, to));

    jdbc.execute("CREATE INDEX IF NOT EXISTS ON %s (customer_id)".formatted(partitionName));
}
```

---

## Cross-Shard Queries

Queries spanning multiple shards must be executed in parallel and merged:

```java
@Service
@RequiredArgsConstructor
public class CrossShardOrderService {

    private final List<OrderRepository> shardRepos;   // one per shard
    private final ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

    public List<Order> findAllPendingOrders() {
        List<CompletableFuture<List<Order>>> futures = shardRepos.stream()
            .map(repo -> CompletableFuture.supplyAsync(
                repo::findByStatusPending, executor))
            .toList();

        return CompletableFuture.allOf(futures.toArray(CompletableFuture[]::new))
            .thenApply(v -> futures.stream()
                .flatMap(f -> f.join().stream())
                .sorted(Comparator.comparing(Order::getCreatedAt).reversed())
                .toList())
            .join();
    }

    public long countAllOrders() {
        return shardRepos.stream()
            .mapToLong(OrderRepository::count)
            .sum();
    }
}
```

---

## Sharding Pitfalls

| Pitfall | Detail |
|---------|--------|
| Cross-shard JOINs | Not possible at DB level — must be done in application code |
| Cross-shard transactions | No distributed ACID — use Saga or 2PC (expensive) |
| Resharding | Rebalancing data across new shards requires careful migration |
| Global IDs | Auto-increment PKs conflict; use UUID, Snowflake IDs, or sequence offsets |
| Schema changes | Must apply to all shards simultaneously |

---

## Database Sharding Summary

| Concept | Detail |
|---------|--------|
| Range sharding | Contiguous key ranges per shard — good for range queries |
| Hash sharding | `hash(key) % shards` — even distribution, bad for ranges |
| Consistent hashing | Minimises resharding cost when nodes join/leave |
| `AbstractRoutingDataSource` | Spring mechanism for selecting DataSource per request via thread-local |
| `ShardContext` | `ThreadLocal<String>` storing current shard key — cleared in `finally` |
| PostgreSQL partitioning | Table-level; planner prunes partitions — no app changes needed |
| Range partition | `PARTITION BY RANGE` — date-based, archive old partitions |
| Hash partition | `PARTITION BY HASH` — even distribution within one PostgreSQL instance |
| Cross-shard queries | Execute per-shard in parallel, merge results in application |
| Shard key selection | Choose a key that keeps related data co-located (e.g., `customer_id`) |

---

[← Back to README](../README.md)
