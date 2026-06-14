# Redis Deep Dive

[← Back to README](../README.md)

---

**Redis** is an in-memory data structure store used for caching, pub/sub messaging, session storage, leaderboards, rate limiting, and distributed locking. This doc covers Redis's core data structures and Spring integration beyond basic caching.

---

## Running Redis Locally

```yaml
# compose.yml
services:
  redis:
    image: redis:7.2-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes   # persistence
    volumes:
      - redis-data:/data

volumes:
  redis-data:
```

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: ${REDIS_PASSWORD:}
      lettuce:
        pool:
          max-active: 10
          max-idle: 5
```

---

## RedisTemplate — Low-Level Access

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory cf) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(cf);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
}
```

---

## Data Structures

### Strings (most common)

```java
ValueOperations<String, Object> ops = redisTemplate.opsForValue();

// Set with TTL
ops.set("session:user123", sessionData, Duration.ofMinutes(30));

// Atomic increment (counters, rate limiting)
ops.increment("counter:page-views");
Long views = (Long) ops.get("counter:page-views");

// Set if absent (distributed lock / idempotency)
Boolean created = ops.setIfAbsent("lock:order-123", "locked", Duration.ofSeconds(30));

// Get and set atomically
Object previous = ops.getAndSet("key", "newValue");
```

### Hashes (objects/maps)

```java
HashOperations<String, String, Object> hash = redisTemplate.opsForHash();

// Store a user session as fields
hash.put("user:123", "name",      "Alice");
hash.put("user:123", "email",     "alice@example.com");
hash.put("user:123", "loginTime", Instant.now().toString());

// Get all fields
Map<String, Object> user = hash.entries("user:123");

// Get one field
String name = (String) hash.get("user:123", "name");

// Delete a field
hash.delete("user:123", "loginTime");

// Increment a numeric field
hash.increment("user:123", "loginCount", 1);
```

### Lists (queues / activity feeds)

```java
ListOperations<String, Object> list = redisTemplate.opsForList();

// Push to left (queue head)
list.leftPush("notifications:user123", notification);

// Pop from right (dequeue — FIFO)
Object next = list.rightPop("notifications:user123");

// Blocking pop — waits up to 5 seconds for an item
Object item = list.rightPop("work-queue", Duration.ofSeconds(5));

// Trim to last 100 items (activity feed)
list.trim("feed:user123", 0, 99);

// Get range
List<Object> recent = list.range("feed:user123", 0, 9);  // latest 10
```

### Sets (unique members, tags)

```java
SetOperations<String, Object> set = redisTemplate.opsForSet();

set.add("online-users", "alice", "bob", "carol");
Boolean isOnline = set.isMember("online-users", "alice");
Long count = set.size("online-users");
set.remove("online-users", "bob");

// Set operations
Set<Object> friends = set.intersect("friends:alice", "friends:bob");
Set<Object> all     = set.union("friends:alice", "friends:bob");
```

### Sorted Sets (leaderboards, scheduling)

```java
ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();

// Add with score
zset.add("leaderboard", "alice", 4250.0);
zset.add("leaderboard", "bob",   3100.0);
zset.add("leaderboard", "carol", 5800.0);

// Increment score
zset.incrementScore("leaderboard", "alice", 150.0);

// Top 10 (highest score first)
Set<Object> top10 = zset.reverseRange("leaderboard", 0, 9);

// Top 10 with scores
Set<ZSetOperations.TypedTuple<Object>> top10Scored =
    zset.reverseRangeWithScores("leaderboard", 0, 9);

// Rank of a member (0-indexed, lowest rank = highest score)
Long rank = zset.reverseRank("leaderboard", "alice");
```

---

## Pub/Sub — Message Broadcasting

```java
// Publisher
@Service
public class EventPublisher {

    private final RedisTemplate<String, Object> redisTemplate;

    public void publishOrderEvent(OrderEvent event) {
        redisTemplate.convertAndSend("orders:events", event);
    }
}

// Subscriber
@Component
public class OrderEventSubscriber implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] pattern) {
        String body = new String(message.getBody());
        OrderEvent event = objectMapper.readValue(body, OrderEvent.class);
        log.info("Received order event: {}", event);
    }
}

// Wire up the subscriber
@Configuration
public class RedisPubSubConfig {

    @Bean
    public RedisMessageListenerContainer listenerContainer(
            RedisConnectionFactory cf,
            OrderEventSubscriber subscriber) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(cf);
        container.addMessageListener(subscriber,
            new ChannelTopic("orders:events"));
        return container;
    }
}
```

---

## Redis Streams — Persistent Pub/Sub

Unlike pub/sub, streams persist messages and allow consumer groups (like Kafka):

```java
// Produce a message
StreamOperations<String, String, String> streams = redisTemplate.opsForStream();

MapRecord<String, String, String> record = StreamRecords.newRecord()
    .in("orders:stream")
    .ofMap(Map.of(
        "orderId",    "order-123",
        "customerId", "cust-456",
        "total",      "99.99"));

RecordId id = streams.add(record);

// Consume from a consumer group
streams.createGroup("orders:stream", ReadOffset.from("0"), "inventory-group");

List<MapRecord<String, String, String>> messages = streams.read(
    Consumer.from("inventory-group", "consumer-1"),
    StreamReadOptions.empty().count(10),
    StreamOffset.create("orders:stream", ReadOffset.lastConsumed()));

for (MapRecord<String, String, String> msg : messages) {
    processOrder(msg.getValue());
    streams.acknowledge("orders:stream", "inventory-group", msg.getId());
}
```

---

## Distributed Lock

```java
@Service
public class DistributedLockService {

    private final RedisTemplate<String, String> redisTemplate;

    public boolean acquireLock(String lockKey, String lockValue, Duration ttl) {
        Boolean acquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, lockValue, ttl);
        return Boolean.TRUE.equals(acquired);
    }

    public void releaseLock(String lockKey, String lockValue) {
        // Only release if we own the lock (Lua script for atomicity)
        String script = """
            if redis.call('get', KEYS[1]) == ARGV[1] then
                return redis.call('del', KEYS[1])
            else
                return 0
            end
            """;
        redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            List.of(lockKey),
            lockValue);
    }
}

// Usage
String lockKey   = "lock:order-processing:" + orderId;
String lockValue = UUID.randomUUID().toString();

if (lockService.acquireLock(lockKey, lockValue, Duration.ofSeconds(30))) {
    try {
        processOrder(orderId);
    } finally {
        lockService.releaseLock(lockKey, lockValue);
    }
}
```

---

## Lua Scripts — Atomic Multi-Command Operations

```java
// Atomic check-and-increment (rate limiting)
String script = """
    local current = redis.call('incr', KEYS[1])
    if current == 1 then
        redis.call('expire', KEYS[1], ARGV[1])
    end
    return current
    """;

Long count = redisTemplate.execute(
    new DefaultRedisScript<>(script, Long.class),
    List.of("rate:user123"),
    "60");   // 60-second window

if (count > 100) throw new RateLimitExceededException();
```

---

## Transactions (MULTI/EXEC)

```java
List<Object> results = redisTemplate.execute(new SessionCallback<>() {
    @Override
    public List<Object> execute(RedisOperations ops) {
        ops.watch("inventory:PROD-1");           // watch for changes

        ops.multi();                              // start transaction
        ops.opsForValue().decrement("inventory:PROD-1");
        ops.opsForValue().increment("orders:count");

        return ops.exec();                        // exec — null if watched key changed
    }
});
```

---

## Key Expiry and TTL

```java
// Set TTL
redisTemplate.expire("session:user123", Duration.ofMinutes(30));

// Get remaining TTL
Duration ttl = redisTemplate.getExpire("session:user123");

// Remove expiry (make key permanent)
redisTemplate.persist("session:user123");

// Keys expiring in a pattern (use scan, not keys — non-blocking)
ScanOptions opts = ScanOptions.scanOptions().match("session:*").count(100).build();
try (Cursor<byte[]> cursor = redisTemplate.getConnectionFactory()
        .getConnection().scan(opts)) {
    cursor.forEachRemaining(key -> System.out.println(new String(key)));
}
```

---

## Redis Deep Dive Summary

| Data Structure | Best For |
|----------------|---------|
| String | Counters, sessions, simple values, distributed locks |
| Hash | Objects with multiple fields (user profiles) |
| List | Queues, activity feeds, recent items |
| Set | Unique membership, tagging, set operations |
| Sorted Set | Leaderboards, scheduled tasks, range queries by score |
| Stream | Persistent pub/sub with consumer groups (like Kafka) |
| Pub/Sub | Fire-and-forget broadcast (no persistence) |

| Feature | API |
|---------|-----|
| Atomic increment | `opsForValue().increment(key)` |
| Set if absent | `opsForValue().setIfAbsent(key, value, ttl)` |
| Distributed lock | `setIfAbsent` + Lua script release |
| Atomic multi-command | Lua script via `RedisScript` |
| Transactions | `SessionCallback` + `multi()` / `exec()` |
| Non-blocking key scan | `connection.scan(ScanOptions)` |

---

[← Back to README](../README.md)
