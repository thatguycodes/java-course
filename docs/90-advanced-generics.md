# Advanced Generics

[← Back to README](../README.md)

---

Java Generics provide compile-time type safety without runtime overhead. Beyond basic `List<String>`, generics have a rich system of wildcards, bounds, and type inference that is essential for writing flexible, reusable library code.

---

## Type Parameters

```java
// Single type parameter
public class Box<T> {
    private T value;
    public Box(T value)  { this.value = value; }
    public T get()       { return value; }
    public <R> Box<R> map(Function<T, R> f) { return new Box<>(f.apply(value)); }
}

// Multiple type parameters
public class Pair<A, B> {
    private final A first;
    private final B second;
    public Pair(A first, B second) { this.first = first; this.second = second; }
    public A first()  { return first; }
    public B second() { return second; }
    public static <X, Y> Pair<X, Y> of(X x, Y y) { return new Pair<>(x, y); }
}
```

---

## Bounded Type Parameters

```java
// Upper bound — T must be a Number or a subclass of Number
public <T extends Number> double sum(List<T> list) {
    return list.stream().mapToDouble(Number::doubleValue).sum();
}

// Multiple bounds — T must implement both Comparable and Serializable
public <T extends Comparable<T> & Serializable> T max(List<T> list) {
    return list.stream().max(Comparator.naturalOrder()).orElseThrow();
}

// Bounded on a generic class
public class SortedList<T extends Comparable<T>> {
    private final List<T> items = new ArrayList<>();

    public void add(T item) {
        items.add(item);
        Collections.sort(items);
    }
}
```

---

## Wildcards

### Unbounded Wildcard `<?>`

```java
// Accept any List regardless of element type — read-only
public void printList(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}

printList(List.of(1, 2, 3));       // OK
printList(List.of("a", "b", "c")); // OK
```

### Upper Bounded Wildcard `<? extends T>` — Producer

```java
// Read numbers from any list of Number or subtype (Integer, Double, etc.)
public double sumNumbers(List<? extends Number> list) {
    return list.stream().mapToDouble(Number::doubleValue).sum();
}

sumNumbers(List.of(1, 2, 3));          // Integer extends Number ✓
sumNumbers(List.of(1.5, 2.5, 3.5));   // Double extends Number ✓
```

You can **read** from `List<? extends T>` but **cannot add** to it (the exact type is unknown):

```java
List<? extends Number> nums = new ArrayList<Integer>();
Number n = nums.get(0);   // OK — safe to read as Number
nums.add(42);             // COMPILE ERROR — could be List<Double>, not List<Integer>
```

### Lower Bounded Wildcard `<? super T>` — Consumer

```java
// Write integers into any list that can hold Integer or a supertype
public void addIntegers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
    list.add(3);
}

List<Number>  numbers = new ArrayList<>();
List<Object>  objects = new ArrayList<>();
addIntegers(numbers);   // OK — Number is supertype of Integer
addIntegers(objects);   // OK — Object is supertype of Integer
```

---

## PECS — Producer Extends, Consumer Super

The canonical rule for choosing wildcards:

> If a collection **produces** (you read from it) → `<? extends T>`  
> If a collection **consumes** (you write to it) → `<? super T>`

```java
// Classic PECS example — copy elements from source to dest
public static <T> void copy(
        List<? extends T> source,   // producer — we read from it
        List<? super T>   dest) {   // consumer — we write to it
    for (T item : source) {
        dest.add(item);
    }
}

List<Integer> ints  = List.of(1, 2, 3);
List<Number>  nums  = new ArrayList<>();
copy(ints, nums);   // OK — Integer extends Number; Number super Integer
```

---

## Generic Methods

```java
// Type parameter on the method — separate from the class's type parameter
public class Collections {

    public static <T extends Comparable<? super T>> void sort(List<T> list) { ... }

    public static <T> Optional<T> findFirst(List<T> list, Predicate<T> predicate) {
        return list.stream().filter(predicate).findFirst();
    }

    // Return type inferred from arguments
    public static <K, V> Map<K, V> mapOf(K key, V value) {
        return Map.of(key, value);
    }
}
```

---

## Type Erasure

At runtime, all generic type information is **erased** — `List<String>` and `List<Integer>` are both just `List` at the bytecode level.

```java
// At runtime, these are the same type
List<String>  strings = new ArrayList<>();
List<Integer> ints    = new ArrayList<>();
System.out.println(strings.getClass() == ints.getClass()); // true

// Cannot use generic types with instanceof
if (list instanceof List<String>) { }  // COMPILE ERROR

// Cannot create generic arrays
T[] array = new T[10];               // COMPILE ERROR

// Cannot overload by generic parameter alone
void process(List<String> list)  {}
void process(List<Integer> list) {}  // COMPILE ERROR — same erasure
```

### Retaining Type Information with Super-Type Tokens

```java
// Use TypeReference to pass generic type info at runtime (Jackson pattern)
public <T> T deserialize(String json, Class<T> type) {
    return objectMapper.readValue(json, type);
}

// For generic types, use TypeReference
List<Order> orders = objectMapper.readValue(json,
    new TypeReference<List<Order>>() {});

// Spring's ParameterizedTypeReference
ResponseEntity<List<Order>> response = restClient.get()
    .uri("/api/orders")
    .retrieve()
    .toEntity(new ParameterizedTypeReference<List<Order>>() {});
```

---

## Recursive Generics (F-Bounded Polymorphism)

```java
// Builder pattern where the builder returns its own type
public abstract class Builder<T, B extends Builder<T, B>> {

    @SuppressWarnings("unchecked")
    protected B self() { return (B) this; }

    public abstract T build();
}

public class PersonBuilder extends Builder<Person, PersonBuilder> {
    private String name;
    private int    age;

    public PersonBuilder name(String name) { this.name = name; return self(); }
    public PersonBuilder age(int age)      { this.age  = age;  return self(); }

    @Override
    public Person build() { return new Person(name, age); }
}

// Fluent chaining works with correct return type
Person p = new PersonBuilder().name("Alice").age(30).build();
```

---

## Generic Interfaces and Variance

```java
// Covariant return type — subclass can return a more specific type
public interface Repository<T, ID> {
    Optional<T> findById(ID id);
    List<T> findAll();
    T save(T entity);
}

public class OrderRepository implements Repository<Order, UUID> {
    @Override public Optional<Order> findById(UUID id) { ... }
    @Override public List<Order>     findAll()          { ... }
    @Override public Order           save(Order order)  { ... }
}
```

---

## Practical Patterns

### Generic Result Type

```java
public sealed interface Result<T>
    permits Result.Success, Result.Failure {

    record Success<T>(T value)            implements Result<T> {}
    record Failure<T>(String error, Exception cause) implements Result<T> {}

    static <T> Result<T> of(Supplier<T> supplier) {
        try {
            return new Success<>(supplier.get());
        } catch (Exception e) {
            return new Failure<>(e.getMessage(), e);
        }
    }

    default <R> Result<R> map(Function<T, R> mapper) {
        return switch (this) {
            case Success<T> s  -> Result.of(() -> mapper.apply(s.value()));
            case Failure<T> f  -> new Failure<>(f.error(), f.cause());
        };
    }
}

// Usage
Result<Order> result = Result.of(() -> orderService.place(cmd));
switch (result) {
    case Result.Success<Order> s -> ResponseEntity.ok(s.value());
    case Result.Failure<Order> f -> ResponseEntity.badRequest().body(f.error());
}
```

---

## Advanced Generics Summary

| Concept | Syntax | Use Case |
|---------|--------|---------|
| Upper bound | `<T extends Number>` | Restrict to a type or its subtypes |
| Multiple bounds | `<T extends A & B>` | Restrict to types implementing both |
| Upper wildcard | `<? extends T>` | Read from a collection (producer) |
| Lower wildcard | `<? super T>` | Write to a collection (consumer) |
| PECS | Producer Extends, Consumer Super | Choose the right wildcard |
| Type erasure | `List<String>` → `List` at runtime | No generic instanceof, no generic arrays |
| Super-type token | `new TypeReference<List<T>>(){}` | Preserve generic type info at runtime |
| Recursive bound | `<B extends Builder<T, B>>` | Fluent builder with correct return types |

---

[← Back to README](../README.md)
