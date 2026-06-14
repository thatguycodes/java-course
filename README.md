# Java Data Types

Java is a statically typed language — every variable must have a declared type. Types fall into two categories: **primitive** and **reference**.

---

## Primitive Types

Primitives are the building blocks of Java data. They hold values directly (not references), live on the stack, and have fixed sizes regardless of platform.

Java has exactly **8 primitive types**:

### Integer Types

| Type    | Size    | Range                                                   | Default |
|---------|---------|----------------------------------------------------------|---------|
| `byte`  | 8-bit   | -128 to 127                                              | `0`     |
| `short` | 16-bit  | -32,768 to 32,767                                        | `0`     |
| `int`   | 32-bit  | -2,147,483,648 to 2,147,483,647                          | `0`     |
| `long`  | 64-bit  | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | `0L`    |

```java
byte  age       = 25;
short year      = 2026;
int   population = 1_000_000;   // underscores improve readability
long  distance  = 9_460_730_472_580_800L;  // note the L suffix
```

### Floating-Point Types

| Type     | Size    | Precision          | Default |
|----------|---------|--------------------|---------|
| `float`  | 32-bit  | ~6–7 decimal digits  | `0.0f`  |
| `double` | 64-bit  | ~15–16 decimal digits | `0.0d`  |

```java
float  price    = 9.99f;      // f suffix required
double pi       = 3.141592653589793;
```

> Prefer `double` over `float` for most calculations — it is more precise and is the default for decimal literals.

### Character Type

| Type   | Size    | Range             | Default    |
|--------|---------|-------------------|------------|
| `char` | 16-bit  | `\u0000` to `\uffff` (0–65,535) | `'\u0000'` |

`char` stores a single Unicode character.

```java
char letter  = 'A';
char unicode = 'A';  // also 'A'
```

### Boolean Type

| Type      | Size        | Values          | Default |
|-----------|-------------|-----------------|---------|
| `boolean` | JVM-defined | `true` / `false` | `false` |

```java
boolean isActive = true;
boolean isAdmin  = false;
```

---

## Key Rules for Primitives

- **Default values** only apply to class fields, not local variables. Using an uninitialized local variable is a compile error.
- **Type widening** happens automatically when assigning a smaller type to a larger one (`int` → `long` → `double`).
- **Type narrowing** requires an explicit cast and may lose data:

```java
double d = 9.99;
int    i = (int) d;  // i = 9, decimal is truncated
```

- Integer literals are `int` by default; append `L` for `long`.
- Decimal literals are `double` by default; append `f` for `float`.

---

## Reference Types

Reference types store a **memory address** (a reference) pointing to an object on the heap, not the value itself. The default value for any reference type is `null`.

Every class, interface, array, and enum in Java is a reference type.

### Strings

`String` is the most commonly used reference type. Strings are **immutable** — once created, their content cannot change.

```java
String name    = "Alice";
String greeting = new String("Hello");  // rarely needed; prefer literals
```

String literals are stored in the **string pool** — Java reuses existing identical strings to save memory.

```java
String a = "hello";
String b = "hello";
System.out.println(a == b);       // true  — same pool reference
System.out.println(a.equals(b));  // true  — same content
```

> Always use `.equals()` to compare string content, never `==`. The `==` operator compares references, not values.

Common `String` methods:

```java
String s = "Hello, World!";

s.length();           // 13
s.toUpperCase();      // "HELLO, WORLD!"
s.substring(7, 12);  // "World"
s.contains("World"); // true
s.replace("World", "Java"); // "Hello, Java!"
s.trim();            // strips leading/trailing whitespace
s.split(", ");       // ["Hello", "World!"]
```

---

### Arrays

An array holds a fixed-size, ordered collection of elements of the same type. Arrays can hold primitives or reference types.

```java
// declaration and initialization
int[]    numbers = new int[5];          // [0, 0, 0, 0, 0]
String[] names   = new String[3];       // [null, null, null]

// inline initialization
int[]    scores  = {90, 85, 78, 92};
String[] fruits  = {"apple", "banana", "cherry"};

// access by index (zero-based)
System.out.println(scores[0]);  // 90
scores[1] = 100;

// length property
System.out.println(scores.length);  // 4
```

**Multi-dimensional arrays:**

```java
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6}
};

System.out.println(matrix[1][2]);  // 6
```

> Arrays have a fixed size after creation. Use `ArrayList` when you need a resizable list.

---

### Classes and Objects

Any class you define (or use from the Java standard library) is a reference type.

```java
// using a built-in class
java.util.ArrayList<String> list = new java.util.ArrayList<>();
list.add("one");
list.add("two");

// using a user-defined class
public class Person {
    String name;
    int    age;
}

Person p = new Person();
p.name = "Alice";
p.age  = 30;
```

When you assign a reference variable, both variables point to the **same object**:

```java
Person a = new Person();
Person b = a;       // b points to the same object as a
b.name = "Bob";
System.out.println(a.name);  // "Bob" — a is affected too
```

---

### Wrapper Classes

Every primitive has a corresponding **wrapper class** in `java.lang` that boxes it into an object. This is needed when working with generics (e.g., `ArrayList<Integer>`).

| Primitive | Wrapper     |
|-----------|-------------|
| `byte`    | `Byte`      |
| `short`   | `Short`     |
| `int`     | `Integer`   |
| `long`    | `Long`      |
| `float`   | `Float`     |
| `double`  | `Double`    |
| `char`    | `Character` |
| `boolean` | `Boolean`   |

**Autoboxing** converts a primitive to its wrapper automatically; **unboxing** does the reverse:

```java
int     primitive = 42;
Integer boxed     = primitive;   // autoboxing
int     back      = boxed;       // unboxing

// required for generics
java.util.ArrayList<Integer> nums = new java.util.ArrayList<>();
nums.add(10);   // autoboxes int → Integer
int val = nums.get(0);  // unboxes Integer → int
```

Wrapper classes also provide useful utility methods:

```java
int parsed = Integer.parseInt("123");   // String → int
String s   = Integer.toString(456);    // int → String
int max    = Integer.MAX_VALUE;        // 2147483647
```

---

## Primitives vs Reference Types — Quick Comparison

| Feature          | Primitive              | Reference                    |
|------------------|------------------------|------------------------------|
| Stores           | Value directly         | Address of object on heap    |
| Default value    | `0`, `false`, `'\0'`  | `null`                       |
| Memory location  | Stack                  | Heap (reference on stack)    |
| Comparison (`==`) | Compares values       | Compares references          |
| Nullable         | No                     | Yes                          |
| Used in generics | No (use wrapper)       | Yes                          |

---

## Type Casting and Promotion Rules

### Widening (Implicit) Conversion

Java automatically converts a smaller type to a larger compatible type — no cast needed, no data lost.

The widening order for numeric types:

```
byte → short → int → long → float → double
                char ↗
```

```java
int    i = 100;
long   l = i;      // int → long, automatic
double d = l;      // long → double, automatic

char   c = 'A';
int    n = c;      // char → int, gives Unicode value (65)
```

### Narrowing (Explicit) Conversion

Going the other direction requires an explicit **cast** using `(type)`. Data can be lost or distorted.

```java
double d = 9.99;
int    i = (int) d;     // 9 — decimal portion is truncated, not rounded

long   l = 1_000_000_000_000L;
int    x = (int) l;     // overflow: result is unpredictable

int    n = 65;
char   c = (char) n;    // 'A'
```

> Casting between unrelated types (e.g., `String` to `int`) is not a cast — use wrapper methods like `Integer.parseInt()`.

### Numeric Promotion in Expressions

Before arithmetic is performed, Java promotes operands according to these rules:

1. If either operand is `double`, the other is promoted to `double`.
2. If either operand is `float`, the other is promoted to `float`.
3. If either operand is `long`, the other is promoted to `long`.
4. Otherwise, both operands are promoted to `int` (even if they are `byte`, `short`, or `char`).

```java
byte a = 10, b = 20;
// byte result = a + b;  // compile error — a + b is promoted to int
int  result = a + b;     // correct

short s = 5;
// s = s + 1;   // compile error — s + 1 is int
s = (short)(s + 1);      // explicit cast required

double total = 7 / 2;    // 3.0 — integer division happens first
double exact = 7.0 / 2;  // 3.5 — 2 is promoted to double
```

### Casting with Reference Types

Reference type casting works within an inheritance hierarchy.

**Upcasting** (subclass → superclass) is implicit and always safe:

```java
// Dog extends Animal
Animal a = new Dog();   // upcast — automatic
```

**Downcasting** (superclass → subclass) requires an explicit cast and can throw `ClassCastException` at runtime if the object isn't actually that type:

```java
Animal a = new Dog();
Dog    d = (Dog) a;      // downcast — explicit cast required, safe here

Animal a2 = new Animal();
Dog    d2 = (Dog) a2;    // compiles, but throws ClassCastException at runtime
```

Use `instanceof` to guard before downcasting:

```java
if (a instanceof Dog) {
    Dog d = (Dog) a;    // safe
}

// Java 16+ pattern matching — cast and bind in one step
if (a instanceof Dog dog) {
    dog.fetch();
}
```

---

## `var` and Local Variable Type Inference (Java 10+)

Java 10 introduced `var`, which lets the compiler infer the type of a local variable from its initializer. The type is still static and fixed at compile time — `var` is not dynamic typing.

### Basic Usage

```java
var name    = "Alice";          // inferred as String
var age     = 30;               // inferred as int
var price   = 9.99;             // inferred as double
var active  = true;             // inferred as boolean
var scores  = new int[]{1,2,3}; // inferred as int[]
```

This is equivalent to writing the type explicitly — the bytecode is identical.

### Where `var` Can Be Used

`var` is only allowed for **local variables with an initializer**:

```java
// local variables
var list = new java.util.ArrayList<String>();

// for-loop variables
for (var item : list) {
    System.out.println(item);
}

// traditional for loop
for (var i = 0; i < 10; i++) {
    System.out.println(i);
}

// try-with-resources
try (var reader = new java.io.FileReader("file.txt")) {
    // ...
}
```

### Where `var` Cannot Be Used

```java
// class fields — not allowed
var count = 0;  // compile error if used as a field

// method parameters — not allowed
void greet(var name) { }  // compile error

// return types — not allowed
var getAge() { return 30; }  // compile error

// without an initializer — not allowed
var x;  // compile error — compiler has nothing to infer from

// null initializer — not allowed
var obj = null;  // compile error — type is ambiguous
```

### `var` with Generics

`var` shines most when the type is verbose but obvious from the right-hand side:

```java
// without var
java.util.Map<String, java.util.List<Integer>> map = new java.util.HashMap<>();

// with var
var map = new java.util.HashMap<String, java.util.List<Integer>>();
```

### When to Use `var`

Use `var` when the type is **clear from context** and the variable name communicates intent:

```java
var users = new java.util.ArrayList<String>();  // clear
var result = compute();                          // unclear — what type is result?
```

Avoid `var` when the inferred type would surprise a reader or obscure what the variable holds.

---

## Operators and Expressions

An **expression** is any combination of variables, literals, and operators that evaluates to a single value.

### Arithmetic Operators

| Operator | Name           | Example      | Result |
|----------|----------------|--------------|--------|
| `+`      | Addition       | `5 + 3`      | `8`    |
| `-`      | Subtraction    | `5 - 3`      | `2`    |
| `*`      | Multiplication | `5 * 3`      | `15`   |
| `/`      | Division       | `7 / 2`      | `3` (integer division) |
| `%`      | Modulus        | `7 % 2`      | `1`    |

```java
int a = 10, b = 3;

System.out.println(a + b);   // 13
System.out.println(a - b);   // 7
System.out.println(a * b);   // 30
System.out.println(a / b);   // 3  — truncates, does not round
System.out.println(a % b);   // 1  — remainder
```

> `/` between two integers performs integer division. Use a `double` operand to get a decimal result: `10.0 / 3` → `3.3333...`

### Assignment Operators

| Operator | Equivalent to  |
|----------|----------------|
| `=`      | assign         |
| `+=`     | `x = x + n`   |
| `-=`     | `x = x - n`   |
| `*=`     | `x = x * n`   |
| `/=`     | `x = x / n`   |
| `%=`     | `x = x % n`   |

```java
int x = 10;
x += 5;   // x = 15
x -= 3;   // x = 12
x *= 2;   // x = 24
x /= 4;   // x = 6
x %= 4;   // x = 2
```

### Increment and Decrement

| Operator | Name            | Effect       |
|----------|-----------------|--------------|
| `++x`    | Pre-increment   | Increments, then returns new value |
| `x++`    | Post-increment  | Returns current value, then increments |
| `--x`    | Pre-decrement   | Decrements, then returns new value |
| `x--`    | Post-decrement  | Returns current value, then decrements |

```java
int n = 5;
System.out.println(++n);  // 6 — n is 6
System.out.println(n++);  // 6 — n becomes 7 after this line
System.out.println(n);    // 7
```

### Comparison Operators

Always return `boolean`.

| Operator | Meaning                  |
|----------|--------------------------|
| `==`     | Equal to                 |
| `!=`     | Not equal to             |
| `>`      | Greater than             |
| `<`      | Less than                |
| `>=`     | Greater than or equal to |
| `<=`     | Less than or equal to    |

```java
int a = 5, b = 10;

System.out.println(a == b);  // false
System.out.println(a != b);  // true
System.out.println(a < b);   // true
System.out.println(a >= 5);  // true
```

> Use `.equals()` to compare object content (e.g., Strings). `==` on reference types compares addresses, not values.

### Logical Operators

| Operator | Name | Description                              |
|----------|------|------------------------------------------|
| `&&`     | AND  | `true` if both operands are `true`       |
| `\|\|`   | OR   | `true` if at least one operand is `true` |
| `!`      | NOT  | Inverts the boolean value                |

```java
boolean sunny = true, warm = false;

System.out.println(sunny && warm);   // false
System.out.println(sunny || warm);   // true
System.out.println(!sunny);          // false
```

`&&` and `||` are **short-circuit** operators — the right operand is only evaluated if needed:

```java
int x = 0;
if (x != 0 && 10 / x > 1) {  // 10/x is never reached, no ArithmeticException
    System.out.println("ok");
}
```

### Bitwise Operators

Operate on individual bits of integer types.

| Operator | Name         | Example (`5 & 3`) | Result |
|----------|--------------|-------------------|--------|
| `&`      | AND          | `0101 & 0011`     | `0001` (1) |
| `\|`     | OR           | `0101 \| 0011`    | `0111` (7) |
| `^`      | XOR          | `0101 ^ 0011`     | `0110` (6) |
| `~`      | NOT (unary)  | `~0101`           | `1010` (-6 in two's complement) |
| `<<`     | Left shift   | `1 << 3`          | `8`    |
| `>>`     | Right shift  | `8 >> 2`          | `2`    |
| `>>>`    | Unsigned right shift | `−1 >>> 1` | `Integer.MAX_VALUE` |

```java
int a = 5, b = 3;   // 5 = 0101, 3 = 0011

System.out.println(a & b);   // 1
System.out.println(a | b);   // 7
System.out.println(a ^ b);   // 6
System.out.println(a << 1);  // 10 (multiply by 2)
System.out.println(a >> 1);  // 2  (divide by 2)
```

### Ternary Operator

A compact `if-else` in expression form: `condition ? valueIfTrue : valueIfFalse`

```java
int age = 20;
String status = age >= 18 ? "adult" : "minor";
System.out.println(status);  // "adult"

int a = 5, b = 10;
int max = a > b ? a : b;     // 10
```

### Operator Precedence

Higher precedence operators are evaluated first. When in doubt, use parentheses.

| Precedence | Operators |
|------------|-----------|
| Highest    | `++` `--` (postfix), `!` `~` (unary) |
|            | `*` `/` `%` |
|            | `+` `-` |
|            | `<<` `>>` `>>>` |
|            | `<` `>` `<=` `>=` `instanceof` |
|            | `==` `!=` |
|            | `&` |
|            | `^` |
|            | `\|` |
|            | `&&` |
|            | `\|\|` |
|            | `?:` (ternary) |
| Lowest     | `=` `+=` `-=` etc. (assignment) |

```java
int result = 2 + 3 * 4;       // 14, not 20 — * before +
int clear  = (2 + 3) * 4;     // 20 — parentheses override precedence
boolean b  = 5 > 3 && 2 < 4;  // true — comparisons before &&
```

---

## Control Flow

Control flow statements determine the order in which code executes.

### if / else if / else

```java
int score = 75;

if (score >= 90) {
    System.out.println("A");
} else if (score >= 80) {
    System.out.println("B");
} else if (score >= 70) {
    System.out.println("C");
} else {
    System.out.println("F");
}
```

Braces are optional for single-statement bodies, but always using them avoids subtle bugs.

### switch Statement

Matches a value against a set of cases. Works with `int`, `char`, `String`, and enums.

```java
int day = 3;

switch (day) {
    case 1:
        System.out.println("Monday");
        break;
    case 2:
        System.out.println("Tuesday");
        break;
    case 3:
        System.out.println("Wednesday");
        break;
    default:
        System.out.println("Other");
}
```

> Without `break`, execution **falls through** to the next case. This is rarely intentional — always include `break` unless you explicitly want fall-through.

### switch Expression (Java 14+)

A cleaner form that returns a value and requires no `break`. Uses `->` syntax.

```java
int day = 3;

String name = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4 -> "Thursday";
    case 5 -> "Friday";
    default -> "Weekend";
};

System.out.println(name);  // "Wednesday"
```

Multiple labels can share a branch:

```java
String type = switch (day) {
    case 1, 2, 3, 4, 5 -> "Weekday";
    case 6, 7           -> "Weekend";
    default             -> "Unknown";
};
```

### for Loop

Best when the number of iterations is known in advance.

```java
for (int i = 0; i < 5; i++) {
    System.out.println(i);  // 0, 1, 2, 3, 4
}

// counting down
for (int i = 10; i > 0; i--) {
    System.out.println(i);
}
```

### Enhanced for Loop (for-each)

Iterates over arrays and collections without managing an index.

```java
int[] scores = {90, 85, 78, 92};

for (int score : scores) {
    System.out.println(score);
}

var fruits = java.util.List.of("apple", "banana", "cherry");

for (var fruit : fruits) {
    System.out.println(fruit);
}
```

### while Loop

Repeats while a condition is `true`. Best when the number of iterations is not known in advance.

```java
int n = 1;

while (n <= 5) {
    System.out.println(n);
    n++;
}
```

### do-while Loop

Like `while`, but the body executes **at least once** before the condition is checked.

```java
int n = 1;

do {
    System.out.println(n);
    n++;
} while (n <= 5);
```

### break and continue

`break` exits the loop immediately. `continue` skips the rest of the current iteration and moves to the next.

```java
// break — stop at first even number
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) {
        System.out.println("First even: " + i);
        break;
    }
}

// continue — skip odd numbers
for (int i = 0; i < 10; i++) {
    if (i % 2 != 0) continue;
    System.out.println(i);  // 0, 2, 4, 6, 8
}
```

**Labeled break** exits a specific outer loop:

```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == 1 && j == 1) break outer;
        System.out.println(i + "," + j);
    }
}
// prints: 0,0  0,1  0,2  1,0
```

### Control Flow Summary

| Statement       | Use when                                      |
|-----------------|-----------------------------------------------|
| `if / else`     | Branching on a boolean condition              |
| `switch`        | Matching one value against many fixed cases   |
| `for`           | Iteration count is known                      |
| `for-each`      | Iterating over all elements of a collection   |
| `while`         | Iteration count is unknown, check first       |
| `do-while`      | Body must run at least once                   |
| `break`         | Exit a loop or switch early                   |
| `continue`      | Skip current iteration, keep looping         |

---

## Methods

In Java, functions are called **methods** — they always belong to a class. A method groups reusable logic under a name, accepts optional inputs (parameters), and optionally returns a value.

### Anatomy of a Method

```java
accessModifier returnType methodName(parameterList) {
    // body
    return value;
}
```

```java
public static int add(int a, int b) {
    return a + b;
}
```

| Part            | Example       | Meaning                                      |
|-----------------|---------------|----------------------------------------------|
| Access modifier | `public`      | Who can call this method                     |
| `static`        | `static`      | Belongs to the class, not an instance        |
| Return type     | `int`         | Type of value returned (`void` if none)      |
| Method name     | `add`         | Identifier, camelCase by convention          |
| Parameters      | `int a, int b` | Input variables, typed and comma-separated  |
| `return`        | `return a + b` | Exits the method and sends a value back     |

### Defining and Calling Methods

```java
public class Calculator {

    public static int add(int a, int b) {
        return a + b;
    }

    public static void printGreeting(String name) {  // void — no return value
        System.out.println("Hello, " + name + "!");
    }

    public static void main(String[] args) {
        int sum = add(3, 7);          // call with arguments
        System.out.println(sum);      // 10

        printGreeting("Alice");       // Hello, Alice!
    }
}
```

### Parameters vs Arguments

- **Parameters** are the variables declared in the method signature: `int a, int b`
- **Arguments** are the actual values passed when calling the method: `add(3, 7)`

### void Methods

A method with return type `void` performs an action but returns nothing. Using `return` in a void method exits early but carries no value.

```java
public static void printRange(int start, int end) {
    if (start >= end) return;  // early exit
    for (int i = start; i < end; i++) {
        System.out.println(i);
    }
}
```

### Method Overloading

Multiple methods can share the same name if their **parameter lists differ** (different types or different count). The compiler picks the right one based on the arguments passed.

```java
public static int multiply(int a, int b) {
    return a * b;
}

public static double multiply(double a, double b) {
    return a * b;
}

public static int multiply(int a, int b, int c) {
    return a * b * c;
}

// calls
multiply(2, 3);          // int version → 6
multiply(2.0, 3.5);      // double version → 7.0
multiply(2, 3, 4);       // three-arg version → 24
```

> Return type alone is not enough to distinguish overloads — two methods with the same name and parameters but different return types won't compile.

### Varargs

A varargs parameter accepts zero or more arguments of the same type. Declared with `...`, it must be the last parameter.

```java
public static int sum(int... numbers) {
    int total = 0;
    for (int n : numbers) {
        total += n;
    }
    return total;
}

sum();              // 0
sum(1, 2);          // 3
sum(1, 2, 3, 4);   // 10
```

### Pass by Value

Java always passes arguments **by value** — the method gets a copy. For primitives this means changes inside the method don't affect the caller. For reference types, the reference itself is copied, so the method can mutate the object's fields but cannot reassign the caller's variable.

```java
public static void doubleIt(int n) {
    n = n * 2;  // only modifies the local copy
}

int x = 5;
doubleIt(x);
System.out.println(x);  // still 5

// reference type — mutation is visible to caller
public static void clearList(java.util.List<String> list) {
    list.clear();  // modifies the object the reference points to
}

var names = new java.util.ArrayList<>(java.util.List.of("Alice", "Bob"));
clearList(names);
System.out.println(names);  // []
```

### Recursion

A method can call itself. Every recursive method needs a **base case** to stop the recursion.

```java
public static int factorial(int n) {
    if (n <= 1) return 1;          // base case
    return n * factorial(n - 1);   // recursive case
}

System.out.println(factorial(5));  // 120
```

### static vs Instance Methods

| | `static` method | Instance method |
|---|---|---|
| Belongs to | The class | An object (instance) |
| Called via | `ClassName.method()` | `object.method()` |
| Can access `static` fields | Yes | Yes |
| Can access instance fields | No | Yes |

```java
public class Counter {
    private int count = 0;  // instance field

    public void increment() {   // instance method
        count++;
    }

    public int getCount() {     // instance method
        return count;
    }

    public static int max(int a, int b) {  // static method
        return a > b ? a : b;
    }
}

Counter c = new Counter();
c.increment();
c.increment();
System.out.println(c.getCount());     // 2
System.out.println(Counter.max(3, 7)); // 7
```

---

## Object-Oriented Programming (OOP)

Java is built around four core OOP principles: **Encapsulation**, **Inheritance**, **Polymorphism**, and **Abstraction**.

---

### Classes and Objects

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

### Constructors

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

Point origin = new Point();       // (0, 0)
Point p      = new Point(3, 4);   // (3, 4)
```

`this` refers to the current instance and disambiguates fields from parameters when they share a name.

---

### 1. Encapsulation

Hide internal state behind `private` fields and expose controlled access through `public` getters and setters.

```java
public class BankAccount {
    private double balance;  // hidden from outside

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }

    public double getBalance() {       // getter
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

### 2. Inheritance

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

### 3. Polymorphism

The same reference type can behave differently depending on the actual object it holds. There are two forms:

**Runtime polymorphism (method overriding):**

```java
Animal a1 = new Animal("Generic");
Animal a2 = new Cat("Luna");   // Cat stored in an Animal reference

a1.speak();  // Generic makes a sound.
a2.speak();  // Luna says: Meow!  — Cat's version is called
```

The JVM decides which `speak()` to call at runtime based on the actual object type.

**Compile-time polymorphism (method overloading)** — covered in the Methods section.

---

### 4. Abstraction

Abstraction hides implementation details and exposes only what is necessary. Java provides two mechanisms: **abstract classes** and **interfaces**.

#### Abstract Classes

An abstract class cannot be instantiated. It can have abstract methods (no body) that subclasses must implement, as well as concrete methods.

```java
public abstract class Shape {
    String color;

    public Shape(String color) {
        this.color = color;
    }

    public abstract double area();   // subclasses must implement this

    public void describe() {         // concrete — inherited as-is
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

#### Interfaces

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
sq.draw();         // Drawing a square with side 4.0
sq.resize(2.0);
sq.draw();         // Drawing a square with side 8.0
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

#### Abstract Class vs Interface

| | Abstract class | Interface |
|---|---|---|
| Instantiable | No | No |
| Can have fields | Yes | Only `static final` constants |
| Can have constructors | Yes | No |
| Method bodies | Yes (some abstract) | Only `default`/`static` methods |
| Extends / implements | `extends` (one only) | `implements` (many) |
| Use when | Shared base with some common logic | Defining a capability/contract |

---

### The `final` Keyword

`final` prevents modification at different levels:

```java
final int MAX = 100;          // constant — cannot be reassigned

final class Immutable { }     // cannot be subclassed

public final void lock() { }  // cannot be overridden in a subclass
```

---

### OOP Summary

| Concept       | Purpose                                              | Mechanism                       |
|---------------|------------------------------------------------------|---------------------------------|
| Encapsulation | Protect internal state                               | `private` fields + getters/setters |
| Inheritance   | Reuse and extend behaviour                           | `extends`                       |
| Polymorphism  | One interface, many implementations                  | Method overriding / overloading |
| Abstraction   | Hide complexity, expose essentials                   | Abstract classes, interfaces    |

---

## Exception Handling

An **exception** is an event that disrupts normal program flow. Java uses a structured mechanism — `try`, `catch`, `finally`, and `throw` — to detect and recover from errors gracefully.

### The Exception Hierarchy

```
Throwable
├── Error              (JVM-level problems — do not catch these)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception
    ├── RuntimeException       (unchecked — compiler does not require handling)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── IllegalArgumentException
    │   ├── ArithmeticException
    │   └── ClassCastException
    └── IOException            (checked — must be handled or declared)
        ├── FileNotFoundException
        └── ...
```

- **Checked exceptions** must be either caught or declared with `throws` in the method signature.
- **Unchecked exceptions** (`RuntimeException` and subclasses) do not need to be declared — they signal programming errors.

---

### try / catch / finally

```java
try {
    // code that might throw
} catch (ExceptionType name) {
    // handle the exception
} finally {
    // always runs, whether or not an exception occurred
}
```

```java
int[] numbers = {1, 2, 3};

try {
    System.out.println(numbers[5]);   // throws ArrayIndexOutOfBoundsException
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Index out of range: " + e.getMessage());
} finally {
    System.out.println("This always runs.");
}
```

`finally` is typically used to release resources (close files, database connections, etc.). It runs even if a `return` statement is hit inside `try` or `catch`.

### Catching Multiple Exceptions

```java
try {
    String s = null;
    System.out.println(s.length());     // NullPointerException
    int result = 10 / 0;               // ArithmeticException
} catch (NullPointerException e) {
    System.out.println("Null value: " + e.getMessage());
} catch (ArithmeticException e) {
    System.out.println("Math error: " + e.getMessage());
}
```

**Multi-catch** — handle multiple types in one block when the response is the same:

```java
try {
    // ...
} catch (NullPointerException | IllegalArgumentException e) {
    System.out.println("Input error: " + e.getMessage());
}
```

> Catch from most specific to most general. A broader catch block listed first would swallow exceptions meant for a narrower one.

### throw and throws

`throw` manually raises an exception. `throws` declares that a method may propagate a checked exception to its caller.

```java
public static double divide(int a, int b) {
    if (b == 0) throw new IllegalArgumentException("Divisor cannot be zero");
    return (double) a / b;
}

// checked exception — caller must handle or re-declare
public static String readFile(String path) throws java.io.IOException {
    return java.nio.file.Files.readString(java.nio.file.Path.of(path));
}

// caller handles the checked exception
try {
    String content = readFile("data.txt");
} catch (java.io.IOException e) {
    System.out.println("Could not read file: " + e.getMessage());
}
```

### try-with-resources

Automatically closes resources that implement `AutoCloseable` — no need for a `finally` block to call `.close()`.

```java
try (var reader = new java.io.BufferedReader(new java.io.FileReader("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        System.out.println(line);
    }
} catch (java.io.IOException e) {
    System.out.println("Error reading file: " + e.getMessage());
}
// reader.close() is called automatically here
```

Multiple resources can be opened in one statement, separated by `;`. They are closed in reverse order.

### Custom Exceptions

Extend `Exception` for a checked custom exception, or `RuntimeException` for an unchecked one.

```java
public class InsufficientFundsException extends Exception {
    private double amount;

    public InsufficientFundsException(double amount) {
        super("Insufficient funds. Short by: " + amount);
        this.amount = amount;
    }

    public double getAmount() {
        return amount;
    }
}

public class BankAccount {
    private double balance;

    public BankAccount(double balance) {
        this.balance = balance;
    }

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(amount - balance);
        }
        balance -= amount;
    }
}

BankAccount account = new BankAccount(100);

try {
    account.withdraw(150);
} catch (InsufficientFundsException e) {
    System.out.println(e.getMessage());       // Insufficient funds. Short by: 50.0
    System.out.println("Need: " + e.getAmount());  // Need: 50.0
}
```

### Exception Handling Best Practices

- **Catch specific exceptions** — avoid catching `Exception` or `Throwable` unless you genuinely need to.
- **Don't swallow exceptions** — an empty `catch` block hides bugs. At minimum, log the error.
- **Use finally or try-with-resources** — never leave resources open.
- **Throw early, catch late** — validate inputs at the boundary, handle exceptions where you have enough context to recover.
- **Prefer unchecked for programming errors** — use `IllegalArgumentException`, `IllegalStateException`, etc. for things the caller should fix in code.
- **Include context in messages** — `"Index out of bounds"` is less useful than `"Index 7 out of bounds for length 3"`.

### Exception Handling Summary

| Keyword          | Purpose                                                  |
|------------------|----------------------------------------------------------|
| `try`            | Wraps code that may throw                                |
| `catch`          | Handles a specific exception type                        |
| `finally`        | Always executes — used for cleanup                       |
| `throw`          | Manually raises an exception                             |
| `throws`         | Declares that a method may propagate a checked exception |
| try-with-resources | Auto-closes `AutoCloseable` resources               |

