# Object-Oriented Programming (OOP)

[← Back to README](../README.md)

---

Java is built around four core OOP principles: **Encapsulation**, **Inheritance**, **Polymorphism**, and **Abstraction**.

## Classes and Objects

A **class** is a blueprint. An **object** is an instance of that blueprint created with `new`.

```java
public class Dog {
    // fields (state)
    String name;
    String breed;
    int    age;

    // constructor
    public Dog(String name, String breed, int age) {
        this.name  = name;
        this.breed = breed;
        this.age   = age;
    }

    // method (behaviour)
    public void bark() {
        System.out.println(name + " says: Woof!");
    }
}

Dog d1 = new Dog("Rex", "Labrador", 3);
Dog d2 = new Dog("Bella", "Poodle", 5);

d1.bark();  // Rex says: Woof!
d2.bark();  // Bella says: Woof!
```

## Constructors

A constructor initialises a new object. It has the same name as the class and no return type.

```java
public class Point {
    int x, y;

    // no-arg constructor
    public Point() {
        this.x = 0;
        this.y = 0;
    }

    // parameterised constructor
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

Point origin = new Point();     // (0, 0)
Point p      = new Point(3, 4); // (3, 4)
```

`this` refers to the current instance and disambiguates fields from parameters when they share a name.

---

## 1. Encapsulation

Hide internal state behind `private` fields and expose controlled access through `public` getters and setters.

```java
public class BankAccount {
    private double balance;

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) balance -= amount;
    }
}

BankAccount account = new BankAccount(1000);
account.deposit(500);
account.withdraw(200);
System.out.println(account.getBalance());  // 1300.0
// account.balance = -999;  // compile error — field is private
```

**Access modifiers at a glance:**

| Modifier    | Same class | Same package | Subclass | Everywhere |
|-------------|:----------:|:------------:|:--------:|:----------:|
| `private`   | ✓          |              |          |            |
| (default)   | ✓          | ✓            |          |            |
| `protected` | ✓          | ✓            | ✓        |            |
| `public`    | ✓          | ✓            | ✓        | ✓          |

---

## 2. Inheritance

A subclass **extends** a superclass to inherit its fields and methods, and can add or override behaviour. Java supports single inheritance for classes.

```java
public class Animal {
    String name;

    public Animal(String name) {
        this.name = name;
    }

    public void speak() {
        System.out.println(name + " makes a sound.");
    }
}

public class Cat extends Animal {

    public Cat(String name) {
        super(name);  // call the superclass constructor
    }

    @Override
    public void speak() {
        System.out.println(name + " says: Meow!");
    }

    public void purr() {
        System.out.println(name + " purrs...");
    }
}

Cat c = new Cat("Whiskers");
c.speak();  // Whiskers says: Meow!
c.purr();   // Whiskers purrs...
```

- `super(...)` calls the parent constructor.
- `@Override` tells the compiler you intend to override a parent method — it catches typos in the method signature.
- A subclass can call a parent method explicitly with `super.methodName()`.

---

## 3. Polymorphism

The same reference type can behave differently depending on the actual object it holds.

**Runtime polymorphism (method overriding):**

```java
Animal a1 = new Animal("Generic");
Animal a2 = new Cat("Luna");   // Cat stored in an Animal reference

a1.speak();  // Generic makes a sound.
a2.speak();  // Luna says: Meow!  — Cat's version is called
```

The JVM decides which `speak()` to call at runtime based on the actual object type.

**Compile-time polymorphism (method overloading)** — covered in [Methods](05-methods.md).

---

## 4. Abstraction

Abstraction hides implementation details and exposes only what is necessary. Java provides two mechanisms: **abstract classes** and **interfaces**.

### Abstract Classes

An abstract class cannot be instantiated. It can have abstract methods (no body) that subclasses must implement, as well as concrete methods.

```java
public abstract class Shape {
    String color;

    public Shape(String color) {
        this.color = color;
    }

    public abstract double area();  // subclasses must implement this

    public void describe() {        // concrete — inherited as-is
        System.out.println(color + " shape with area " + area());
    }
}

public class Circle extends Shape {
    double radius;

    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

Shape s = new Circle("Red", 5);
s.describe();  // Red shape with area 78.53981633974483
```

### Interfaces

An interface is a pure contract — it defines what a class must do, not how. A class **implements** an interface and must provide bodies for all its methods. A class can implement multiple interfaces.

```java
public interface Drawable {
    void draw();              // implicitly public and abstract
}

public interface Resizable {
    void resize(double factor);
}

public class Square implements Drawable, Resizable {
    double side;

    public Square(double side) {
        this.side = side;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a square with side " + side);
    }

    @Override
    public void resize(double factor) {
        side *= factor;
    }
}

Square sq = new Square(4);
sq.draw();       // Drawing a square with side 4.0
sq.resize(2.0);
sq.draw();       // Drawing a square with side 8.0
```

Interfaces can also have `default` methods (Java 8+) with a body, used to add behaviour without breaking existing implementations:

```java
public interface Printable {
    void print();

    default void printTwice() {
        print();
        print();
    }
}
```

### Abstract Class vs Interface

|                      | Abstract class                    | Interface                          |
|----------------------|-----------------------------------|------------------------------------|
| Instantiable         | No                                | No                                 |
| Can have fields      | Yes                               | Only `static final` constants      |
| Can have constructors | Yes                              | No                                 |
| Method bodies        | Yes (some abstract)               | Only `default`/`static` methods    |
| Extends / implements | `extends` (one only)              | `implements` (many)                |
| Use when             | Shared base with some common logic | Defining a capability/contract    |

---

## The `final` Keyword

`final` prevents modification at different levels:

```java
final int MAX = 100;          // constant — cannot be reassigned

final class Immutable { }     // cannot be subclassed

public final void lock() { }  // cannot be overridden in a subclass
```

---

## OOP Summary

| Concept       | Purpose                             | Mechanism                          |
|---------------|-------------------------------------|------------------------------------|
| Encapsulation | Protect internal state              | `private` fields + getters/setters |
| Inheritance   | Reuse and extend behaviour          | `extends`                          |
| Polymorphism  | One interface, many implementations | Method overriding / overloading    |
| Abstraction   | Hide complexity, expose essentials  | Abstract classes, interfaces       |

---

[← Back to README](../README.md)
