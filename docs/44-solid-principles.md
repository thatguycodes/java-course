# SOLID Principles

[← Back to README](../README.md)

---

**SOLID** is a set of five object-oriented design principles that make software easier to understand, change, and extend. They were popularised by Robert C. Martin and are the foundation of clean OO design.

| Letter | Principle | One-line rule |
|--------|-----------|--------------|
| **S** | Single Responsibility | A class has one reason to change |
| **O** | Open/Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Subtypes must be substitutable for their base type |
| **I** | Interface Segregation | Many small interfaces beat one large one |
| **D** | Dependency Inversion | Depend on abstractions, not concretions |

---

## S — Single Responsibility Principle

> A class should have only **one reason to change**.

### Violation

```java
// UserService does three unrelated things:
// authentication, email, and database access — three reasons to change
public class UserService {
    public User authenticate(String email, String password) { ... }
    public void sendWelcomeEmail(User user)                 { ... }
    public void saveToDatabase(User user)                   { ... }
}
```

### Fixed

```java
public class AuthService {
    private final UserRepository repo;
    private final PasswordEncoder encoder;

    public User authenticate(String email, String password) {
        User user = repo.findByEmail(email).orElseThrow();
        if (!encoder.matches(password, user.getPassword())) throw new BadCredentialsException();
        return user;
    }
}

public class EmailService {
    public void sendWelcomeEmail(User user) { /* SMTP logic */ }
}

public class UserRepository {
    public void save(User user) { /* DB logic */ }
}
```

Each class has one reason to change: `AuthService` changes when auth rules change, `EmailService` when templates change, `UserRepository` when the DB schema changes.

---

## O — Open/Closed Principle

> Software should be **open for extension** but **closed for modification**.

Add new behaviour by adding new code, not by editing existing code.

### Violation

```java
// adding a new shape requires editing this method
public double calculateArea(Object shape) {
    if (shape instanceof Circle c)    return Math.PI * c.radius() * c.radius();
    if (shape instanceof Rectangle r) return r.width() * r.height();
    // adding Triangle means editing this method — violates OCP
    throw new IllegalArgumentException("Unknown shape");
}
```

### Fixed

```java
// adding a new shape = adding a new class, zero edits to existing code
public interface Shape {
    double area();
}

public record Circle(double radius) implements Shape {
    public double area() { return Math.PI * radius * radius; }
}

public record Rectangle(double width, double height) implements Shape {
    public double area() { return width * height; }
}

public record Triangle(double base, double h) implements Shape {
    public double area() { return 0.5 * base * h; }
}

// never needs to change
public double calculateTotal(List<Shape> shapes) {
    return shapes.stream().mapToDouble(Shape::area).sum();
}
```

---

## L — Liskov Substitution Principle

> Objects of a subclass must be replaceable with objects of the superclass **without altering program correctness**.

### Violation

```java
public class Rectangle {
    protected int width, height;
    public void setWidth(int w)  { this.width  = w; }
    public void setHeight(int h) { this.height = h; }
    public int area() { return width * height; }
}

// Square "is-a" Rectangle — but breaks the contract
public class Square extends Rectangle {
    @Override public void setWidth(int w)  { width = height = w; }  // changes both!
    @Override public void setHeight(int h) { width = height = h; }
}

// this breaks — caller assumes width and height are independent
Rectangle r = new Square();
r.setWidth(5);
r.setHeight(4);
System.out.println(r.area());  // expected 20, got 16
```

### Fixed

```java
// keep them unrelated — both implement a common interface
public interface Shape { int area(); }

public record Rectangle(int width, int height) implements Shape {
    public int area() { return width * height; }
}

public record Square(int side) implements Shape {
    public int area() { return side * side; }
}
```

**LSP in practice:** if you find yourself overriding a method to do nothing, throw, or weaken a postcondition — you're violating LSP.

---

## I — Interface Segregation Principle

> No client should be forced to depend on methods it does not use.
> Prefer many small, focused interfaces over one large one.

### Violation

```java
// everything that "does work" must implement start() and stop() — even things that can't be paused
public interface Worker {
    void work();
    void eat();    // robots don't eat
    void sleep();  // robots don't sleep
}

public class Robot implements Worker {
    public void work()  { /* ok */ }
    public void eat()   { throw new UnsupportedOperationException(); }  // bad!
    public void sleep() { throw new UnsupportedOperationException(); }  // bad!
}
```

### Fixed

```java
public interface Workable  { void work(); }
public interface Feedable  { void eat(); }
public interface Restable  { void sleep(); }

public class Human implements Workable, Feedable, Restable {
    public void work()  { System.out.println("Working"); }
    public void eat()   { System.out.println("Eating"); }
    public void sleep() { System.out.println("Sleeping"); }
}

public class Robot implements Workable {
    public void work() { System.out.println("Working tirelessly"); }
    // no eat(), no sleep() — doesn't even know they exist
}
```

---

## D — Dependency Inversion Principle

> High-level modules should not depend on low-level modules.
> Both should depend on **abstractions**.

### Violation

```java
// OrderService is tightly coupled to EmailNotifier — can't swap without editing OrderService
public class OrderService {
    private final EmailNotifier notifier = new EmailNotifier();  // concrete!

    public void placeOrder(Order order) {
        processOrder(order);
        notifier.notify(order.getUserEmail(), "Order placed");
    }
}
```

### Fixed

```java
// abstraction
public interface Notifier {
    void notify(String recipient, String message);
}

// low-level modules implement the abstraction
public class EmailNotifier implements Notifier {
    public void notify(String recipient, String message) { /* send email */ }
}

public class SmsNotifier implements Notifier {
    public void notify(String recipient, String message) { /* send SMS */ }
}

// high-level module depends on abstraction — injected, not created
public class OrderService {
    private final Notifier notifier;   // interface, not concrete class

    public OrderService(Notifier notifier) {   // injected — Spring, manual, or test
        this.notifier = notifier;
    }

    public void placeOrder(Order order) {
        processOrder(order);
        notifier.notify(order.getUserEmail(), "Order placed");
    }
}
```

```java
// swap to SMS with zero changes to OrderService
OrderService service = new OrderService(new SmsNotifier());

// or in Spring
@Bean public Notifier notifier() { return new SmsNotifier(); }
```

---

## SOLID in Practice

The principles work together:

```java
// Report generator — all five principles applied

// S: each formatter has one job
public interface ReportFormatter {
    String format(Report report);
}

public class PdfFormatter  implements ReportFormatter { ... }
public class CsvFormatter  implements ReportFormatter { ... }
public class JsonFormatter implements ReportFormatter { ... }

// O: new format = new class, no edits
// I: only the format() method — no bloat
// L: any formatter can replace any other

// D: high-level generator depends on abstraction
public class ReportGenerator {
    private final ReportFormatter formatter;    // interface
    private final ReportRepository repository; // interface

    public ReportGenerator(ReportFormatter formatter, ReportRepository repository) {
        this.formatter  = formatter;
        this.repository = repository;
    }

    public String generate(long reportId) {
        Report report = repository.findById(reportId);  // S: data access is separate
        return formatter.format(report);                // O: open for new formatters
    }
}
```

---

## SOLID Summary

| Principle | Smells that violate it | Fix |
|-----------|----------------------|-----|
| **SRP** | Class has many unrelated methods; changes for multiple reasons | Split into focused classes |
| **OCP** | Adding a feature requires editing existing code | Use interfaces/abstract types; add classes |
| **LSP** | Overriding a method to throw `UnsupportedOperationException` | Redesign hierarchy; use composition |
| **ISP** | Interface forces implementors to stub methods they don't need | Split into smaller interfaces |
| **DIP** | Class calls `new ConcreteService()` inside a method | Inject dependencies through constructors |

---

[← Back to README](../README.md)
