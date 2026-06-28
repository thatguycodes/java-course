# Apache Commons & Google Guava

[← Back to README](../README.md)

---

**Apache Commons** and **Google Guava** are the two most widely used Java utility libraries. Commons provides battle-tested helpers for strings, collections, I/O, and math. Guava adds immutable collections, `Multimap`, `Table`, `RangeMap`, `Optional` extensions, `Preconditions`, functional types, and a caching framework. Both reduce boilerplate and eliminate entire categories of bugs.

---

## Apache Commons Lang

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

### StringUtils

```java
// Null-safe string operations
StringUtils.isBlank(null);              // true
StringUtils.isBlank("  ");             // true
StringUtils.isNotBlank("hello");       // true

StringUtils.capitalize("hello world"); // "Hello world"
StringUtils.abbreviate("Long string",  // "Long..."
    8);

StringUtils.leftPad("42", 5, '0');     // "00042"
StringUtils.rightPad("hello", 10);     // "hello     "

StringUtils.join(List.of("a","b","c"), ", ");  // "a, b, c"

StringUtils.countMatches("abcabc", "a");       // 2
StringUtils.substringBetween("(hello)", "(", ")");  // "hello"

StringUtils.removeAll("hello123world", "[0-9]"); // "helloworld"

// Difference
StringUtils.difference("abcde", "abxyz"); // "xyz"
```

### ObjectUtils

```java
ObjectUtils.defaultIfNull(null, "fallback");   // "fallback"
ObjectUtils.firstNonNull(null, null, "third"); // "third"

// Null-safe compare
ObjectUtils.compare("a", "b");  // negative (like Comparator)
ObjectUtils.compare(null, "b"); // null sorts first
```

### ArrayUtils

```java
int[] arr = {1, 2, 3, 4, 5};

ArrayUtils.contains(arr, 3);          // true
ArrayUtils.indexOf(arr, 3);           // 2
ArrayUtils.reverse(arr);              // {5,4,3,2,1} in-place
int[] removed = ArrayUtils.remove(arr, 2);  // removes index 2
int[] added   = ArrayUtils.add(arr, 6);     // appends 6

String[] names = {"Alice", "Bob"};
String[] joined = ArrayUtils.addAll(names, "Carol", "Dave");
```

### StopWatch

```java
StopWatch sw = StopWatch.createStarted();
performOperation();
sw.stop();
log.info("Took {}ms", sw.getTime(TimeUnit.MILLISECONDS));

// Named splits
StopWatch multi = new StopWatch("orderProcessing");
multi.start("validation");  validateOrder();  multi.stop();
multi.start("persistence"); saveOrder();      multi.stop();
multi.start("notification"); sendEmail();     multi.stop();
log.info(multi.prettyPrint());
// StopWatch 'orderProcessing': running time = 237ms
//   validation   42ms  17%
//   persistence  180ms 75%
//   notification 15ms  6%
```

---

## Apache Commons Collections

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>
```

```java
// CollectionUtils — null-safe operations
CollectionUtils.isEmpty(null);          // true
CollectionUtils.isNotEmpty(list);       // true
CollectionUtils.union(listA, listB);    // A ∪ B
CollectionUtils.intersection(a, b);     // A ∩ B
CollectionUtils.subtract(a, b);         // A − B

// Partition a list into sublists of size n
CollectionUtils.partition(bigList, 100).forEach(batch -> processBatch(batch));

// MultiValuedMap — one key, multiple values
MultiValuedMap<String, String> map = new ArrayListValuedHashMap<>();
map.put("fruits", "apple");
map.put("fruits", "banana");
map.get("fruits");   // ["apple", "banana"]
```

---

## Google Guava

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.2.1-jre</version>
</dependency>
```

### Immutable Collections

```java
// Immutable — no UnsupportedOperationException surprises
ImmutableList<String> list  = ImmutableList.of("a", "b", "c");
ImmutableSet<Integer> set   = ImmutableSet.of(1, 2, 3);
ImmutableMap<String, Integer> map = ImmutableMap.of("one", 1, "two", 2);

// Builder pattern for larger collections
ImmutableMap<String, String> config = ImmutableMap.<String, String>builder()
    .put("host", "localhost")
    .put("port", "5432")
    .put("db",   "orders")
    .buildOrThrow();   // throws on duplicate keys
```

### Multimap — One Key, Multiple Values

```java
// ListMultimap — preserves insertion order per key
ListMultimap<String, String> multimap = ArrayListMultimap.create();
multimap.put("fruits", "apple");
multimap.put("fruits", "banana");
multimap.put("veggies", "carrot");

multimap.get("fruits");          // ["apple", "banana"]
multimap.keySet();               // {"fruits", "veggies"}
multimap.values();               // all values flat
multimap.asMap();                // Map<String, Collection<String>>
```

### Table — Two-Key Map

```java
// Like a spreadsheet: row + column → value
Table<String, String, Integer> salesTable = HashBasedTable.create();
salesTable.put("Q1", "North", 1000);
salesTable.put("Q1", "South", 850);
salesTable.put("Q2", "North", 1200);

salesTable.get("Q1", "North");    // 1000
salesTable.row("Q1");             // Map<column, value> for Q1
salesTable.column("North");       // Map<row, value> for North
salesTable.rowKeySet();           // {"Q1", "Q2"}
```

### Preconditions — Fast-Fail Validation

```java
public void processOrder(Order order, int quantity) {
    Preconditions.checkNotNull(order, "order must not be null");
    Preconditions.checkArgument(quantity > 0, "quantity must be positive, was %s", quantity);
    Preconditions.checkState(order.isActive(), "order %s is not active", order.getId());

    // More concise than:
    if (order == null) throw new NullPointerException("order must not be null");
    if (quantity <= 0) throw new IllegalArgumentException("quantity must be positive");
}
```

### Strings, Joiner & Splitter

```java
// Strings
Strings.isNullOrEmpty(null);              // true
Strings.nullToEmpty(null);               // ""
Strings.emptyToNull("");                 // null
Strings.repeat("ab", 3);                // "ababab"
Strings.padStart("42", 5, '0');         // "00042"
Strings.commonPrefix("interview", "intercom");  // "inter"

// Joiner
Joiner.on(", ").skipNulls().join("Alice", null, "Bob");  // "Alice, Bob"
Joiner.on(" | ").useForNull("N/A").join("a", null, "b"); // "a | N/A | b"

// Map joiner
Joiner.on("; ").withKeyValueSeparator("=")
    .join(Map.of("k1", "v1", "k2", "v2")); // "k1=v1; k2=v2"

// Splitter — opposite of Joiner
Splitter.on(',').trimResults().omitEmptyStrings()
    .splitToList("  a , b,,  c  "); // ["a", "b", "c"]

Splitter.on('=').limit(2)
    .splitToList("key=value=extra"); // ["key", "value=extra"]
```

### RangeMap — Interval-Based Lookup

```java
// Map IP CIDR ranges to data centres
RangeMap<Integer, String> ipToDataCentre = TreeRangeMap.create();
ipToDataCentre.put(Range.closed(1, 100),    "dc-east");
ipToDataCentre.put(Range.closed(101, 200),  "dc-west");
ipToDataCentre.put(Range.greaterThan(200),  "dc-cloud");

ipToDataCentre.get(50);   // "dc-east"
ipToDataCentre.get(150);  // "dc-west"
ipToDataCentre.get(300);  // "dc-cloud"
```

### Guava Cache

```java
LoadingCache<String, Product> cache = CacheBuilder.newBuilder()
    .maximumSize(10_000)
    .expireAfterWrite(30, TimeUnit.MINUTES)
    .recordStats()
    .build(key -> productRepository.findById(key)
        .orElseThrow(() -> new ProductNotFoundException(key)));

Product product = cache.get("SKU-001");   // loads if absent

CacheStats stats = cache.stats();
log.info("Hit rate: {:.2f}%, load avg: {}ms",
    stats.hitRate() * 100,
    stats.averageLoadPenalty() / 1_000_000);
```

---

## Apache Commons & Guava Summary

| API | Detail |
|-----|--------|
| `StringUtils.isBlank` | Null + whitespace-safe blank check |
| `StringUtils.join` | Null-safe join with separator |
| `ObjectUtils.firstNonNull` | Returns first non-null argument — null coalescing |
| `StopWatch.prettyPrint()` | Formatted multi-split timing report |
| `CollectionUtils.partition` | Split list into fixed-size sublists |
| `ImmutableList/Set/Map` | Truly immutable — safe to share across threads |
| `ListMultimap` | `Map<K, List<V>>` with ergonomic `put/get` API |
| `Table<R, C, V>` | Two-key map; `row(r)` / `column(c)` views |
| `Preconditions.checkArgument` | Fast-fail with formatted message; throws `IllegalArgumentException` |
| `Splitter` + `Joiner` | Null-safe, trimming, limit-aware string splitting and joining |
| `RangeMap` | Interval-to-value map; `get(key)` finds the range containing `key` |
| `CacheBuilder` | Guava's in-process cache with LRU, TTL, stats, and loading function |

---

[← Back to README](../README.md)
