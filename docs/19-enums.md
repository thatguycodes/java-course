# Enums

[← Back to README](../README.md)

---

An **enum** (enumeration) is a special class that represents a fixed set of named constants. Enums in Java are full classes — they can have fields, constructors, methods, and implement interfaces.

---

## Basic Enum

```java
public enum Direction {
    NORTH, SOUTH, EAST, WEST
}

Direction d = Direction.NORTH;
System.out.println(d);           // NORTH
System.out.println(d.name());    // NORTH
System.out.println(d.ordinal()); // 0 (zero-based position)

// switch with enum
switch (d) {
    case NORTH -> System.out.println("Going north");
    case SOUTH -> System.out.println("Going south");
    default    -> System.out.println("Going sideways");
}
```

---

## Enum with Fields and Methods

Enums can have constructors, fields, and methods. The constructor is always `private`.

```java
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS  (4.869e+24, 6.0518e6),
    EARTH  (5.976e+24, 6.37814e6),
    MARS   (6.421e+23, 3.3972e6);

    private final double mass;    // kg
    private final double radius;  // metres

    Planet(double mass, double radius) {
        this.mass   = mass;
        this.radius = radius;
    }

    static final double G = 6.67300E-11;

    public double surfaceGravity() {
        return G * mass / (radius * radius);
    }

    public double surfaceWeight(double otherMass) {
        return otherMass * surfaceGravity();
    }
}

double earthWeight = 75.0;
double mass = earthWeight / Planet.EARTH.surfaceGravity();

for (Planet p : Planet.values()) {
    System.out.printf("Weight on %s is %6.2f%n", p, p.surfaceWeight(mass));
}
// Weight on MERCURY is  28.33
// Weight on VENUS   is  67.89
// Weight on EARTH   is  75.00
// Weight on MARS    is  28.46
```

---

## Enum with Abstract Methods

Each constant can override a method differently.

```java
public enum Operation {
    ADD {
        @Override
        public double apply(double x, double y) { return x + y; }
    },
    SUBTRACT {
        @Override
        public double apply(double x, double y) { return x - y; }
    },
    MULTIPLY {
        @Override
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE {
        @Override
        public double apply(double x, double y) { return x / y; }
    };

    public abstract double apply(double x, double y);
}

for (Operation op : Operation.values()) {
    System.out.printf("10 %s 3 = %.1f%n", op, op.apply(10, 3));
}
// 10 ADD      3 = 13.0
// 10 SUBTRACT 3 = 7.0
// 10 MULTIPLY 3 = 30.0
// 10 DIVIDE   3 = 3.3
```

---

## Enum Implementing an Interface

```java
public interface Printable {
    void print();
}

public enum Status implements Printable {
    PENDING, ACTIVE, INACTIVE, DELETED;

    @Override
    public void print() {
        System.out.println("Status: " + this.name());
    }
}

Status.ACTIVE.print();  // Status: ACTIVE
```

---

## Useful Enum Methods

```java
// values() — all constants as an array
for (Direction d : Direction.values()) {
    System.out.println(d);
}

// valueOf() — get constant by name (throws IllegalArgumentException if not found)
Direction d = Direction.valueOf("NORTH");

// name() — the declared name as a String
System.out.println(Direction.NORTH.name());    // NORTH

// ordinal() — zero-based position in declaration
System.out.println(Direction.EAST.ordinal());  // 2

// compareTo() — compares by ordinal
System.out.println(Direction.NORTH.compareTo(Direction.SOUTH)); // negative (NORTH < SOUTH)
```

---

## EnumSet

A highly efficient `Set` implementation backed by a bit vector — only works with enums.

```java
import java.util.EnumSet;

EnumSet<Direction> cardinals = EnumSet.of(Direction.NORTH, Direction.SOUTH,
                                           Direction.EAST,  Direction.WEST);

EnumSet<Direction> northSouth = EnumSet.of(Direction.NORTH, Direction.SOUTH);
EnumSet<Direction> eastWest   = EnumSet.complementOf(northSouth);  // EAST, WEST

EnumSet<Direction> all   = EnumSet.allOf(Direction.class);
EnumSet<Direction> empty = EnumSet.noneOf(Direction.class);

// range — all constants from one to another (by ordinal)
public enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

EnumSet<Day> weekend  = EnumSet.of(Day.SAT, Day.SUN);
EnumSet<Day> weekdays = EnumSet.range(Day.MON, Day.FRI);

System.out.println(weekend.contains(Day.SAT));  // true
```

---

## EnumMap

A `Map` implementation optimised for enum keys.

```java
import java.util.EnumMap;

EnumMap<Day, String> schedule = new EnumMap<>(Day.class);

schedule.put(Day.MON, "Team standup");
schedule.put(Day.WED, "Design review");
schedule.put(Day.FRI, "Sprint demo");

schedule.forEach((day, event) ->
    System.out.printf("%s: %s%n", day, event));

System.out.println(schedule.getOrDefault(Day.TUE, "No meetings"));
```

---

## Enum as Singleton

Enum constants are guaranteed to be instantiated exactly once by the JVM — making them the safest Singleton implementation.

```java
public enum AppConfig {
    INSTANCE;

    private final String host = System.getenv().getOrDefault("HOST", "localhost");
    private final int    port = Integer.parseInt(
                                    System.getenv().getOrDefault("PORT", "8080"));

    public String getHost() { return host; }
    public int    getPort() { return port; }
}

System.out.println(AppConfig.INSTANCE.getHost());  // localhost
System.out.println(AppConfig.INSTANCE.getPort());  // 8080
```

---

## Enum Strategy Pattern

Use enums to replace conditional logic with polymorphism.

```java
public enum DiscountStrategy {
    NONE {
        @Override public double apply(double price) { return price; }
    },
    STUDENT {
        @Override public double apply(double price) { return price * 0.80; }
    },
    SENIOR {
        @Override public double apply(double price) { return price * 0.75; }
    },
    EMPLOYEE {
        @Override public double apply(double price) { return price * 0.50; }
    };

    public abstract double apply(double price);
}

double price = 100.0;
for (DiscountStrategy strategy : DiscountStrategy.values()) {
    System.out.printf("%-10s → %.2f%n", strategy, strategy.apply(price));
}
// NONE       → 100.00
// STUDENT    → 80.00
// SENIOR     → 75.00
// EMPLOYEE   → 50.00
```

---

## Enums Summary

| Feature | Notes |
|---------|-------|
| Constants | Declared as `UPPER_CASE` by convention |
| `values()` | Returns all constants as an array |
| `valueOf(String)` | Lookup by name — throws if not found |
| `name()` / `ordinal()` | String name / int position |
| Fields and methods | Enums are full classes |
| Abstract methods | Each constant provides its own implementation |
| Interfaces | Enums can implement but not extend |
| `EnumSet` | Fast, memory-efficient set of enum constants |
| `EnumMap` | Fast map with enum keys |
| Singleton | The safest Singleton pattern in Java |

---

[← Back to README](../README.md)
