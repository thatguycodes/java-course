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

