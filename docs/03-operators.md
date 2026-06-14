# Operators and Expressions

[← Back to README](../README.md)

---

An **expression** is any combination of variables, literals, and operators that evaluates to a single value.

## Arithmetic Operators

| Operator | Name           | Example | Result                 |
|----------|----------------|---------|------------------------|
| `+`      | Addition       | `5 + 3` | `8`                    |
| `-`      | Subtraction    | `5 - 3` | `2`                    |
| `*`      | Multiplication | `5 * 3` | `15`                   |
| `/`      | Division       | `7 / 2` | `3` (integer division) |
| `%`      | Modulus        | `7 % 2` | `1`                    |

```java
int a = 10, b = 3;

System.out.println(a + b);   // 13
System.out.println(a - b);   // 7
System.out.println(a * b);   // 30
System.out.println(a / b);   // 3  — truncates, does not round
System.out.println(a % b);   // 1  — remainder
```

> `/` between two integers performs integer division. Use a `double` operand to get a decimal result: `10.0 / 3` → `3.3333...`

---

## Assignment Operators

| Operator | Equivalent to |
|----------|---------------|
| `=`      | assign        |
| `+=`     | `x = x + n`  |
| `-=`     | `x = x - n`  |
| `*=`     | `x = x * n`  |
| `/=`     | `x = x / n`  |
| `%=`     | `x = x % n`  |

```java
int x = 10;
x += 5;   // x = 15
x -= 3;   // x = 12
x *= 2;   // x = 24
x /= 4;   // x = 6
x %= 4;   // x = 2
```

---

## Increment and Decrement

| Operator | Name           | Effect                                     |
|----------|----------------|--------------------------------------------|
| `++x`    | Pre-increment  | Increments, then returns new value         |
| `x++`    | Post-increment | Returns current value, then increments     |
| `--x`    | Pre-decrement  | Decrements, then returns new value         |
| `x--`    | Post-decrement | Returns current value, then decrements     |

```java
int n = 5;
System.out.println(++n);  // 6 — n is 6
System.out.println(n++);  // 6 — n becomes 7 after this line
System.out.println(n);    // 7
```

---

## Comparison Operators

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

---

## Logical Operators

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

---

## Bitwise Operators

Operate on individual bits of integer types.

| Operator | Name                 | Example (`5 op 3`) | Result                          |
|----------|----------------------|--------------------|---------------------------------|
| `&`      | AND                  | `0101 & 0011`      | `0001` (1)                      |
| `\|`     | OR                   | `0101 \| 0011`     | `0111` (7)                      |
| `^`      | XOR                  | `0101 ^ 0011`      | `0110` (6)                      |
| `~`      | NOT (unary)          | `~0101`            | `1010` (-6 in two's complement) |
| `<<`     | Left shift           | `1 << 3`           | `8`                             |
| `>>`     | Right shift          | `8 >> 2`           | `2`                             |
| `>>>`    | Unsigned right shift | `-1 >>> 1`         | `Integer.MAX_VALUE`             |

```java
int a = 5, b = 3;   // 5 = 0101, 3 = 0011

System.out.println(a & b);   // 1
System.out.println(a | b);   // 7
System.out.println(a ^ b);   // 6
System.out.println(a << 1);  // 10 (multiply by 2)
System.out.println(a >> 1);  // 2  (divide by 2)
```

---

## Ternary Operator

A compact `if-else` in expression form: `condition ? valueIfTrue : valueIfFalse`

```java
int age = 20;
String status = age >= 18 ? "adult" : "minor";
System.out.println(status);  // "adult"

int a = 5, b = 10;
int max = a > b ? a : b;     // 10
```

---

## Operator Precedence

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

[← Back to README](../README.md)
