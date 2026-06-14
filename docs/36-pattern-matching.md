# Pattern Matching

[← Back to README](../README.md)

---

Pattern matching lets you test a value's shape and **bind** it to a typed variable in a single expression — eliminating the cast-after-instanceof boilerplate and making `switch` expressions exhaustive and expressive.

---

## instanceof Pattern Matching (Java 16)

### Before

```java
Object obj = "Hello";

if (obj instanceof String) {
    String s = (String) obj;  // redundant cast
    System.out.println(s.toUpperCase());
}
```

### After

```java
Object obj = "Hello";

if (obj instanceof String s) {       // test + bind in one step
    System.out.println(s.toUpperCase());  // s is String here
}

// works with else branches too
if (obj instanceof Integer i) {
    System.out.println("Number: " + i);
} else if (obj instanceof String s && s.length() > 3) {
    System.out.println("Long string: " + s);
} else {
    System.out.println("Other: " + obj);
}
```

The pattern variable is only in scope where the match is guaranteed — the compiler enforces this.

---

## Switch Pattern Matching (Java 21)

`switch` can now match on types, not just constants. Combined with sealed types, it becomes exhaustive.

### Type patterns

```java
static String describe(Object obj) {
    return switch (obj) {
        case Integer i  -> "Integer: " + i;
        case Double  d  -> "Double:  " + d;
        case String  s  -> "String:  " + s;
        case int[]   a  -> "int[] of length " + a.length;
        case null       -> "null";
        default         -> "Other: " + obj.getClass().getSimpleName();
    };
}

System.out.println(describe(42));        // Integer: 42
System.out.println(describe("hello"));   // String:  hello
System.out.println(describe(null));      // null
```

### Guarded patterns (`when`)

Add a condition with `when` to refine a match:

```java
static String classify(Object obj) {
    return switch (obj) {
        case Integer i when i < 0   -> "negative int: " + i;
        case Integer i when i == 0  -> "zero";
        case Integer i              -> "positive int: " + i;
        case String  s when s.isBlank() -> "blank string";
        case String  s              -> "string: \"" + s + "\"";
        default                     -> "other";
    };
}

System.out.println(classify(-5));   // negative int: -5
System.out.println(classify(0));    // zero
System.out.println(classify("  ")); // blank string
```

---

## Sealed Types + Exhaustive Switch

Sealed classes tell the compiler exactly which subtypes exist — so the switch doesn't need `default`.

```java
public sealed interface Shape permits Circle, Rectangle, Triangle {}

public record Circle(double radius)              implements Shape {}
public record Rectangle(double width, double h)  implements Shape {}
public record Triangle(double base, double h)    implements Shape {}
```

```java
static double area(Shape shape) {
    return switch (shape) {
        case Circle    c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.h();
        case Triangle  t -> 0.5 * t.base() * t.h();
        // no default needed — compiler knows all cases are covered
    };
}

static String describe(Shape shape) {
    return switch (shape) {
        case Circle c    when c.radius() > 10 -> "large circle";
        case Circle c                          -> "small circle";
        case Rectangle r when r.width() == r.h() -> "square";
        case Rectangle r                       -> "rectangle";
        case Triangle  t                       -> "triangle";
    };
}
```

---

## Record Patterns (Java 21)

Record patterns let you **deconstruct** a record's components directly in a pattern.

```java
record Point(int x, int y) {}
record Line(Point start, Point end) {}
```

```java
Object obj = new Point(3, 4);

// deconstruct and bind components
if (obj instanceof Point(int x, int y)) {
    System.out.println("x=" + x + ", y=" + y);
}

// nested record patterns
Object line = new Line(new Point(0, 0), new Point(3, 4));

if (line instanceof Line(Point(int x1, int y1), Point(int x2, int y2))) {
    double length = Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
    System.out.println("Length: " + length);  // 5.0
}
```

### In switch

```java
static String describePoint(Object obj) {
    return switch (obj) {
        case Point(int x, int y) when x == 0 && y == 0 -> "origin";
        case Point(int x, int y) when x == 0           -> "on Y-axis at y=" + y;
        case Point(int x, int y) when y == 0           -> "on X-axis at x=" + x;
        case Point(int x, int y)                       -> "point(%d, %d)".formatted(x, y);
        default                                        -> "not a point";
    };
}

System.out.println(describePoint(new Point(0, 0)));  // origin
System.out.println(describePoint(new Point(0, 5)));  // on Y-axis at y=5
System.out.println(describePoint(new Point(3, 4)));  // point(3, 4)
```

---

## Practical Example — Expression Evaluator

```java
sealed interface Expr permits Num, Add, Mul, Neg {}
record Num(double value)      implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Mul(Expr left, Expr right) implements Expr {}
record Neg(Expr expr)         implements Expr {}

static double eval(Expr expr) {
    return switch (expr) {
        case Num(double v)        -> v;
        case Add(var l, var r)    -> eval(l) + eval(r);
        case Mul(var l, var r)    -> eval(l) * eval(r);
        case Neg(var e)           -> -eval(e);
    };
}

static String print(Expr expr) {
    return switch (expr) {
        case Num(double v)        -> String.valueOf(v);
        case Add(var l, var r)    -> "(%s + %s)".formatted(print(l), print(r));
        case Mul(var l, var r)    -> "(%s * %s)".formatted(print(l), print(r));
        case Neg(var e)           -> "(-" + print(e) + ")";
    };
}

// (2 + 3) * -(4)
Expr e = new Mul(new Add(new Num(2), new Num(3)), new Neg(new Num(4)));
System.out.println(print(e));  // ((2.0 + 3.0) * (-4.0))
System.out.println(eval(e));   // -20.0
```

---

## Pattern Matching Summary

| Feature | Java version | Example |
|---------|-------------|---------|
| `instanceof` with binding | Java 16 | `if (obj instanceof String s)` |
| Switch on types | Java 21 | `case Integer i ->` |
| Guarded patterns | Java 21 | `case Integer i when i > 0 ->` |
| `null` in switch | Java 21 | `case null ->` |
| Record patterns | Java 21 | `case Point(int x, int y) ->` |
| Nested record patterns | Java 21 | `case Line(Point(...), Point(...)) ->` |
| Exhaustive switch on sealed | Java 21 | No `default` needed |

> Pattern matching pairs naturally with **sealed types** (for exhaustiveness) and **records** (for deconstruction). Together they replace many uses of the visitor pattern.

---

[← Back to README](../README.md)
