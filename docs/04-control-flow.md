# Control Flow

[← Back to README](../README.md)

---

Control flow statements determine the order in which code executes.

## if / else if / else

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

---

## switch Statement

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

---

## switch Expression (Java 14+)

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

---

## for Loop

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

---

## Enhanced for Loop (for-each)

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

---

## while Loop

Repeats while a condition is `true`. Best when the number of iterations is not known in advance.

```java
int n = 1;

while (n <= 5) {
    System.out.println(n);
    n++;
}
```

---

## do-while Loop

Like `while`, but the body executes **at least once** before the condition is checked.

```java
int n = 1;

do {
    System.out.println(n);
    n++;
} while (n <= 5);
```

---

## break and continue

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

---

## Control Flow Summary

| Statement   | Use when                                    |
|-------------|---------------------------------------------|
| `if / else` | Branching on a boolean condition            |
| `switch`    | Matching one value against many fixed cases |
| `for`       | Iteration count is known                    |
| `for-each`  | Iterating over all elements of a collection |
| `while`     | Iteration count is unknown, check first     |
| `do-while`  | Body must run at least once                 |
| `break`     | Exit a loop or switch early                 |
| `continue`  | Skip current iteration, keep looping        |

---

[← Back to README](../README.md)
