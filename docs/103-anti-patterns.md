# Java Anti-Patterns

[← Back to README](../README.md)

---

Anti-patterns are common solutions that seem reasonable but cause problems in practice — tight coupling, poor testability, fragile code, or outright bugs. Recognising them is as important as knowing the good patterns.

---

## 1. Anemic Domain Model

The domain objects hold only data; all business logic lives in service classes. The result is a procedural design disguised as OOP.

```java
// ANTI-PATTERN — Order is a passive data bag
public class Order {
    private UUID id;
    private String status;
    private BigDecimal total;
    // getters and setters only
}

// Logic scattered across services
public class OrderService {
    public void confirm(Order order) {
        if (!order.getStatus().equals("PENDING")) throw new RuntimeException("...");
        order.setStatus("CONFIRMED");
    }
    public void cancel(Order order) {
        if (order.getStatus().equals("SHIPPED")) throw new RuntimeException("...");
        order.setStatus("CANCELLED");
    }
}
```

```java
// FIXED — behaviour lives in the model
public class Order {
    private UUID id;
    private OrderStatus status;
    private BigDecimal total;

    public void confirm() {
        if (status != OrderStatus.PENDING)
            throw new IllegalStateException("Can only confirm PENDING orders");
        this.status = OrderStatus.CONFIRMED;
    }

    public void cancel() {
        if (status == OrderStatus.SHIPPED)
            throw new IllegalStateException("Cannot cancel a shipped order");
        this.status = OrderStatus.CANCELLED;
    }
}
```

---

## 2. God Object / God Class

One class does everything — thousands of lines, dozens of responsibilities, impossible to test or change without breaking something else.

```java
// ANTI-PATTERN — OrderManager handles everything
public class OrderManager {
    public Order createOrder(...) { ... }
    public void sendConfirmationEmail(...) { ... }
    public void reserveInventory(...) { ... }
    public void chargePayment(...) { ... }
    public void generateInvoicePdf(...) { ... }
    public void updateLoyaltyPoints(...) { ... }
    public void notifyWarehouse(...) { ... }
    // 2 000 more lines...
}
```

```java
// FIXED — single-responsibility classes
public class OrderService      { Order placeOrder(...) { ... } }
public class EmailService      { void sendConfirmation(...) { ... } }
public class InventoryService  { void reserve(...) { ... } }
public class PaymentService    { PaymentResult charge(...) { ... } }
```

---

## 3. Service Locator

Components look up their dependencies from a central registry instead of receiving them via injection. Makes dependencies invisible and tests difficult.

```java
// ANTI-PATTERN
public class OrderService {
    public void placeOrder(PlaceOrderCommand cmd) {
        PaymentService payment = ServiceLocator.get(PaymentService.class);  // hidden dependency
        InventoryService inv   = ServiceLocator.get(InventoryService.class);
        payment.charge(...);
        inv.reserve(...);
    }
}
```

```java
// FIXED — explicit constructor injection
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;

    public OrderService(PaymentService paymentService,
                        InventoryService inventoryService) {
        this.paymentService   = paymentService;
        this.inventoryService = inventoryService;
    }
}
```

---

## 4. Primitive Obsession

Using primitives (`String`, `int`, `long`) for domain concepts that deserve their own type. Leads to validation scattered everywhere and easy parameter mix-ups.

```java
// ANTI-PATTERN
public Order createOrder(String customerId, String productId,
                          int quantity, double price) { ... }

// Easy to swap customerId and productId — same type!
createOrder(productId, customerId, quantity, price);  // compiles, silently wrong
```

```java
// FIXED — value objects / records
public record CustomerId(UUID value) {}
public record ProductId(UUID value) {}
public record Quantity(int value) {
    public Quantity { if (value <= 0) throw new IllegalArgumentException("Quantity must be positive"); }
}
public record Money(BigDecimal amount, String currency) {}

public Order createOrder(CustomerId customerId, ProductId productId,
                          Quantity quantity, Money price) { ... }
// CustomerId and ProductId are now distinct types — can't be swapped
```

---

## 5. Magic Numbers and Strings

Unexplained literals scattered through the code — impossible to understand or update safely.

```java
// ANTI-PATTERN
if (user.getRole().equals("ROLE_003")) { ... }
if (order.getStatus() == 4) { ... }
Thread.sleep(3000);
BigDecimal vat = price.multiply(new BigDecimal("0.15"));
```

```java
// FIXED — named constants and enums
public enum UserRole { ADMIN, CUSTOMER, SUPPORT }
public enum OrderStatus { PENDING, CONFIRMED, SHIPPED, DELIVERED }

private static final Duration RETRY_DELAY = Duration.ofSeconds(3);
private static final BigDecimal VAT_RATE   = new BigDecimal("0.15");

if (user.getRole() == UserRole.ADMIN) { ... }
if (order.getStatus() == OrderStatus.CONFIRMED) { ... }
```

---

## 6. Exception Swallowing

Catching exceptions and doing nothing — bugs disappear silently.

```java
// ANTI-PATTERN
try {
    processPayment(order);
} catch (Exception e) {
    // TODO: handle later
}
// Payment failed? No one knows.
```

```java
// FIXED — always log or rethrow
try {
    processPayment(order);
} catch (PaymentException e) {
    log.error("Payment failed for order {}: {}", order.getId(), e.getMessage(), e);
    throw new OrderProcessingException("Payment failed", e);
} catch (Exception e) {
    log.error("Unexpected error processing order {}", order.getId(), e);
    throw e;   // rethrow — don't hide unexpected errors
}
```

---

## 7. Returning `null` from Collections

Callers must null-check everywhere. One missed check → NullPointerException.

```java
// ANTI-PATTERN
public List<Order> findByCustomer(String customerId) {
    List<Order> orders = repo.query(customerId);
    return orders.isEmpty() ? null : orders;  // callers must null-check
}

// ANTI-PATTERN
public Order findById(UUID id) {
    return repo.findById(id);  // returns null if not found
}
```

```java
// FIXED — empty collection, Optional, or throw
public List<Order> findByCustomer(String customerId) {
    return repo.query(customerId);   // empty list, never null
}

public Optional<Order> findById(UUID id) {
    return repo.findById(id);   // caller explicitly handles absence
}

public Order getById(UUID id) {
    return repo.findById(id)
        .orElseThrow(() -> new OrderNotFoundException(id));
}
```

---

## 8. Mutable Static State / Singleton Abuse

Global mutable state makes code non-thread-safe and untestable.

```java
// ANTI-PATTERN
public class Config {
    public static String databaseUrl = "jdbc:postgresql://localhost/dev";
    public static int maxRetries = 3;
}

// Any thread can change this — chaos in production
Config.databaseUrl = "jdbc:postgresql://prod/orders";
```

```java
// FIXED — immutable config injected via Spring
@ConfigurationProperties(prefix = "app")
public record AppConfig(String databaseUrl, int maxRetries) {}

@Service
public class OrderService {
    private final AppConfig config;

    public OrderService(AppConfig config) { this.config = config; }
}
```

---

## 9. N+1 Queries

Loading a collection and then issuing one query per element — typically caused by lazy loading in JPA.

```java
// ANTI-PATTERN — N+1 with lazy loading
List<Order> orders = orderRepo.findAll();     // 1 query
for (Order order : orders) {
    String name = order.getCustomer().getName(); // 1 query per order = N+1
}
```

```java
// FIXED — JOIN FETCH or EntityGraph
@Query("SELECT o FROM Order o JOIN FETCH o.customer WHERE o.status = :status")
List<Order> findByStatusWithCustomer(@Param("status") OrderStatus status);

// Or with EntityGraph
@EntityGraph(attributePaths = {"customer", "lines"})
List<Order> findByStatus(OrderStatus status);
```

---

## 10. Overusing `@Transactional`

Annotating everything `@Transactional` without understanding scope causes long-held DB connections, lock contention, and performance degradation.

```java
// ANTI-PATTERN — overly broad transaction
@Transactional
public void processOrderBatch(List<UUID> orderIds) {
    for (UUID id : orderIds) {
        Order order = orderRepo.findById(id).orElseThrow();
        sendConfirmationEmail(order);   // network I/O inside transaction!
        callExternalPaymentApi(order);  // another network call — connection held for seconds
        orderRepo.save(order);
    }
    // DB connection held for entire method duration
}
```

```java
// FIXED — minimal transaction scope
public void processOrderBatch(List<UUID> orderIds) {
    for (UUID id : orderIds) {
        processOneOrder(id);   // each order gets its own short transaction
    }
}

@Transactional
private void processOneOrder(UUID id) {
    Order order = orderRepo.findById(id).orElseThrow();
    order.confirm();
    orderRepo.save(order);
    // transaction commits HERE — then email/payment happen outside
}

@Async
public void sendConfirmationEmail(Order order) { ... }  // outside transaction
```

---

## Anti-Patterns Summary

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Anemic Domain Model | Business logic scattered in services | Move behaviour into domain objects |
| God Object | One class does everything | Split by single responsibility |
| Service Locator | Hidden dependencies | Constructor injection |
| Primitive Obsession | Type confusion, scattered validation | Value objects / records |
| Magic Numbers/Strings | Unexplained constants | Named constants / enums |
| Exception Swallowing | Silent failures | Always log or rethrow |
| Returning null | NullPointerException landmines | Return `Optional`, empty collections, or throw |
| Mutable Static State | Thread-safety, untestable | Immutable config via DI |
| N+1 Queries | Exponential DB round-trips | JOIN FETCH / EntityGraph |
| Broad `@Transactional` | Long-held connections, lock contention | Minimal transaction scope; I/O outside transactions |

---

[← Back to README](../README.md)
