# Optional

[← Back to README](../README.md)

---

`Optional<T>` is a container that either holds a value or is empty. It makes the possibility of absence **explicit in the type system** — replacing `null` checks that are easy to forget and cause `NullPointerException`.

---

## Creating Optional

```java
import java.util.Optional;

// wrap a known non-null value
Optional<String> name = Optional.of("Alice");

// wrap a value that might be null
Optional<String> maybe = Optional.ofNullable(getUserName());  // empty if null

// explicitly empty
Optional<String> empty = Optional.empty();

// Optional.of(null) throws NullPointerException — use ofNullable instead
```

---

## Checking and Extracting

```java
Optional<String> opt = Optional.of("hello");

// check
opt.isPresent();   // true
opt.isEmpty();     // false (Java 11+)

// extract — throws NoSuchElementException if empty
opt.get();         // "hello" — avoid; only use after isPresent()

// safe extraction with fallback
opt.orElse("default");                          // "hello"
Optional.<String>empty().orElse("default");     // "default"

// fallback from a supplier (lazy — only called if empty)
opt.orElseGet(() -> computeDefault());

// throw a specific exception if empty
opt.orElseThrow(() -> new UserNotFoundException("Not found"));

// Java 10+ — throw NoSuchElementException with a message
opt.orElseThrow();
```

---

## Transforming

```java
Optional<String> name = Optional.of("  Alice  ");

// map — transform the value if present
Optional<String>  upper  = name.map(String::trim).map(String::toUpperCase);
Optional<Integer> length = name.map(String::trim).map(String::length);

// flatMap — when the transform itself returns Optional (avoids Optional<Optional<T>>)
Optional<User>    user   = findUser(1L);
Optional<Address> addr   = user.flatMap(User::getAddress);  // getAddress returns Optional<Address>

// or — replace an empty Optional with another Optional (Java 9+)
Optional<String> result = findInCache(key)
    .or(() -> findInDatabase(key));   // fallback to DB if cache miss
```

---

## Filtering

```java
Optional<String> name = Optional.of("Alice");

Optional<String> long_ = name.filter(s -> s.length() > 3);  // Optional["Alice"]
Optional<String> short_= name.filter(s -> s.length() > 10); // Optional.empty
```

---

## Consuming

```java
Optional<User> user = findUser(1L);

// ifPresent — run action if value exists
user.ifPresent(u -> System.out.println("Found: " + u.getName()));

// ifPresentOrElse — run one of two actions (Java 9+)
user.ifPresentOrElse(
    u  -> System.out.println("Found: " + u.getName()),
    () -> System.out.println("User not found")
);
```

---

## Converting to Stream

```java
// stream() returns a Stream of 0 or 1 elements (Java 9+)
List<User> users = ids.stream()
    .map(this::findUser)             // Stream<Optional<User>>
    .flatMap(Optional::stream)       // Stream<User> — skips empties
    .toList();

// equivalent to filter(Optional::isPresent).map(Optional::get) — but cleaner
```

---

## Optional in Method Signatures

```java
// Return Optional when the value may legitimately be absent
public Optional<User> findByEmail(String email) {
    return userRepository.findByEmail(email);  // returns Optional from Spring Data
}

// WRONG — Optional as a parameter (just use null or overloads instead)
public void process(Optional<String> name) { ... }   // avoid

// WRONG — Optional in a collection (use empty collection instead)
List<Optional<User>> users = ...;   // avoid
```

---

## Common Patterns

### Chaining fallbacks

```java
Optional<String> result = findInL1Cache(key)
    .or(() -> findInL2Cache(key))
    .or(() -> findInDatabase(key))
    .or(() -> Optional.of("DEFAULT"));
```

### Map + orElse pipeline

```java
String displayName = findUser(userId)
    .map(User::getDisplayName)
    .filter(s -> !s.isBlank())
    .orElse("Anonymous");
```

### Combining two Optionals

```java
Optional<String> first  = Optional.of("John");
Optional<String> last   = Optional.of("Doe");

// flatMap + map
Optional<String> full = first.flatMap(f -> last.map(l -> f + " " + l));
// Optional["John Doe"]
```

### Replacing null checks

```java
// before
String city = null;
if (user != null && user.getAddress() != null) {
    city = user.getAddress().getCity();
}
if (city == null) city = "Unknown";

// after
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("Unknown");
```

---

## Anti-Patterns

```java
// BAD — defeats the purpose; still throws NPE
if (opt.isPresent()) {
    String val = opt.get();
}
// GOOD
opt.ifPresent(val -> ...);
// or
String val = opt.orElse("default");

// BAD — Optional.get() without isPresent
String name = opt.get();   // throws if empty

// BAD — returning null from a method that should return Optional
public Optional<User> find(Long id) {
    return null;   // never do this — return Optional.empty() instead
}

// BAD — Optional<List<T>> — return an empty list instead
public Optional<List<Order>> getOrders() { ... }
// GOOD
public List<Order> getOrders() { return List.of(); }  // empty, not null

// BAD — Optional as a field (use @Nullable or redesign)
private Optional<String> middleName;  // don't store Optional in fields
```

---

## Optional Summary

| Task | Method |
|------|--------|
| Wrap nullable | `Optional.ofNullable(value)` |
| Wrap non-null | `Optional.of(value)` |
| Check empty | `opt.isEmpty()` / `opt.isPresent()` |
| Get or default | `opt.orElse(default)` |
| Get or compute | `opt.orElseGet(supplier)` |
| Get or throw | `opt.orElseThrow(exSupplier)` |
| Transform | `opt.map(fn)` |
| Transform to Optional | `opt.flatMap(fn)` |
| Filter | `opt.filter(predicate)` |
| Side effect if present | `opt.ifPresent(consumer)` |
| Both branches | `opt.ifPresentOrElse(consumer, runnable)` |
| Fallback Optional | `opt.or(supplier)` |
| Convert to Stream | `opt.stream()` |

---

[← Back to README](../README.md)
