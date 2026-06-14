# Records and Sealed Classes (Deep Dive)

[← Back to README](../README.md)

---

Java 16 introduced **records** as concise immutable data carriers; Java 17 introduced **sealed classes** for closed type hierarchies. Together with **pattern matching** (Java 21), they enable expressive, exhaustive data modelling without boilerplate.

---

## Records

### Basics

```java
// Compiler generates: constructor, getters (name()/age()), equals, hashCode, toString
public record Person(String name, int age) {}

Person alice = new Person("Alice", 30);
System.out.println(alice.name());  // Alice
System.out.println(alice);         // Person[name=Alice, age=30]
```

### Compact Constructor — Validation

```java
public record Money(BigDecimal amount, String currency) {

    // Compact constructor — no parameter list, fields already assigned
    public Money {
        Objects.requireNonNull(currency, "currency must not be null");
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("amount must be non-negative");
        }
        amount = amount.setScale(2, RoundingMode.HALF_UP);  // normalise
        currency = currency.toUpperCase();
    }
}
```

### Custom Accessors and Methods

```java
public record Temperature(double value, TemperatureUnit unit) {

    public enum TemperatureUnit { CELSIUS, FAHRENHEIT, KELVIN }

    // Custom accessor — computed, not stored
    public double toCelsius() {
        return switch (unit) {
            case CELSIUS    -> value;
            case FAHRENHEIT -> (value - 32) * 5 / 9;
            case KELVIN     -> value - 273.15;
        };
    }

    // Static factory
    public static Temperature ofCelsius(double value) {
        return new Temperature(value, TemperatureUnit.CELSIUS);
    }

    // Wither — return a new record with one field changed
    public Temperature withUnit(TemperatureUnit newUnit) {
        return new Temperature(toCelsius() * switch (newUnit) {
            case CELSIUS    -> 1;
            case FAHRENHEIT -> 9.0 / 5 + 32;
            case KELVIN     -> 1 + 273.15;
        }, newUnit);
    }
}
```

### Records Implementing Interfaces

```java
public interface Shape {
    double area();
    double perimeter();
}

public record Circle(double radius) implements Shape {
    @Override public double area()      { return Math.PI * radius * radius; }
    @Override public double perimeter() { return 2 * Math.PI * radius; }
}

public record Rectangle(double width, double height) implements Shape {
    @Override public double area()      { return width * height; }
    @Override public double perimeter() { return 2 * (width + height); }
}
```

### Records with JPA — What to Watch Out For

Records are immutable, but JPA requires mutable entities with a no-arg constructor. Use records for **DTOs and value objects**, not JPA entities:

```java
// Good — record as a DTO / projection
public record ProductSummary(Long id, String name, BigDecimal price) {}

// In a Spring Data repository
@Query("SELECT new com.example.dto.ProductSummary(p.id, p.name, p.price) FROM Product p")
List<ProductSummary> findAllSummaries();

// Or as a JPA interface projection (Spring Data)
public interface ProductSummary {
    Long getId();
    String getName();
    BigDecimal getPrice();
}
```

### Serialization with Jackson

Records work with Jackson out of the box in Spring Boot 2.5+:

```java
public record OrderResponse(
    UUID id,
    String status,
    BigDecimal total,
    Instant createdAt
) {}

// No annotations needed — Jackson maps fields by component name
```

---

## Sealed Classes

Sealed classes restrict which classes can extend them. The compiler knows the complete set of subtypes — enabling **exhaustive** switch expressions.

### Syntax

```java
// Only Circle, Rectangle, and Triangle can implement Shape
public sealed interface Shape
    permits Circle, Rectangle, Triangle {}

public record Circle(double radius)                    implements Shape {}
public record Rectangle(double width, double height)   implements Shape {}
public record Triangle(double a, double b, double c)   implements Shape {}
```

Permitted subtypes must be in the same package (or module). Subtypes must be `final`, `sealed`, or `non-sealed`.

```java
// final — cannot be extended further
public final class Circle implements Shape { ... }

// sealed — can be further restricted
public sealed class Vehicle permits Car, Truck { ... }

// non-sealed — opens the hierarchy again (breaks exhaustiveness)
public non-sealed class ElectricCar extends Car { ... }
```

---

## Pattern Matching with Sealed Classes

Because the compiler knows all permitted subtypes, `switch` expressions are **exhaustive** — no `default` needed:

```java
public double calculateArea(Shape shape) {
    return switch (shape) {
        case Circle c        -> Math.PI * c.radius() * c.radius();
        case Rectangle r     -> r.width() * r.height();
        case Triangle t      -> heronFormula(t.a(), t.b(), t.c());
    };  // no default needed — sealed hierarchy is complete
}
```

If you add a new permitted type (e.g., `Hexagon`) and forget to add a case, the **compiler reports an error** — not a runtime exception.

### Guarded Patterns

```java
public String classify(Shape shape) {
    return switch (shape) {
        case Circle c when c.radius() > 100 -> "large circle";
        case Circle c                        -> "small circle";
        case Rectangle r when r.width() == r.height() -> "square";
        case Rectangle r                     -> "rectangle";
        case Triangle t                      -> "triangle";
    };
}
```

### Nested Deconstruction

```java
public sealed interface Expr permits Num, Add, Mul {}
public record Num(int value)       implements Expr {}
public record Add(Expr left, Expr right) implements Expr {}
public record Mul(Expr left, Expr right) implements Expr {}

public int evaluate(Expr expr) {
    return switch (expr) {
        case Num(int v)              -> v;
        case Add(Expr l, Expr r)     -> evaluate(l) + evaluate(r);
        case Mul(Expr l, Expr r)     -> evaluate(l) * evaluate(r);
    };
}

// Nested deconstruction
public String describe(Expr expr) {
    return switch (expr) {
        case Mul(Num(int a), Num(int b)) -> a + " × " + b;  // specific case first
        case Mul m                        -> "multiplication";
        case Add a                        -> "addition";
        case Num(int v)                   -> String.valueOf(v);
    };
}
```

---

## Modelling Domain Errors

Sealed classes are ideal for typed error results instead of exceptions:

```java
public sealed interface OrderResult
    permits OrderResult.Success, OrderResult.PaymentFailed, OrderResult.OutOfStock {}

public record Success(OrderId orderId, BigDecimal total)
    implements OrderResult {}

public record PaymentFailed(String reason, String transactionId)
    implements OrderResult {}

public record OutOfStock(String productId, int requested, int available)
    implements OrderResult {}
```

```java
// Service
public OrderResult placeOrder(PlaceOrderCommand cmd) {
    if (!inventoryService.hasStock(cmd.productId(), cmd.quantity())) {
        StockLevel stock = inventoryService.getLevel(cmd.productId());
        return new OutOfStock(cmd.productId(), cmd.quantity(), stock.available());
    }
    PaymentResult payment = paymentService.charge(cmd.customerId(), cmd.total());
    if (!payment.success()) {
        return new PaymentFailed(payment.declineReason(), payment.transactionId());
    }
    Order order = orderRepo.save(new Order(cmd));
    return new Success(order.id(), order.total());
}

// Controller
OrderResult result = orderService.placeOrder(cmd);
return switch (result) {
    case Success s          -> ResponseEntity.status(201).body(new OrderResponse(s.orderId()));
    case PaymentFailed pf   -> ResponseEntity.unprocessableEntity()
        .body(new ErrorResponse("payment_failed", pf.reason()));
    case OutOfStock oos     -> ResponseEntity.unprocessableEntity()
        .body(new ErrorResponse("out_of_stock",
            "Only " + oos.available() + " units available"));
};
```

---

## Records and Sealed Classes — Summary

### Records

| Feature | Detail |
|---------|--------|
| Auto-generated | Constructor, component accessors, `equals`, `hashCode`, `toString` |
| Compact constructor | Validate/normalise fields without declaring parameters |
| Immutable | All fields are `final`; no setters |
| Implements interfaces | Can implement interfaces; cannot extend classes |
| JPA | Don't use as `@Entity` — use as DTOs / projections |
| Jackson | Works without annotations in Spring Boot 2.5+ |

### Sealed Classes

| Feature | Detail |
|---------|--------|
| `permits` clause | Lists all allowed direct subtypes |
| Subtypes must be | `final`, `sealed`, or `non-sealed` |
| Exhaustiveness | Compiler enforces `switch` covers all subtypes — no `default` needed |
| Adding a subtype | Compiler errors on all `switch` expressions that don't cover it |
| Guarded patterns | `case Circle c when c.radius() > 100 ->` |
| Record deconstruction | `case Add(Expr l, Expr r) ->` |

---

[← Back to README](../README.md)
