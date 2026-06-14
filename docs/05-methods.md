# Methods

[← Back to README](../README.md)

---

In Java, functions are called **methods** — they always belong to a class. A method groups reusable logic under a name, accepts optional inputs (parameters), and optionally returns a value.

## Anatomy of a Method

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

| Part            | Example        | Meaning                                     |
|-----------------|----------------|---------------------------------------------|
| Access modifier | `public`       | Who can call this method                    |
| `static`        | `static`       | Belongs to the class, not an instance       |
| Return type     | `int`          | Type of value returned (`void` if none)     |
| Method name     | `add`          | Identifier, camelCase by convention         |
| Parameters      | `int a, int b` | Input variables, typed and comma-separated  |
| `return`        | `return a + b` | Exits the method and sends a value back     |

---

## Defining and Calling Methods

```java
public class Calculator {

    public static int add(int a, int b) {
        return a + b;
    }

    public static void printGreeting(String name) {  // void — no return value
        System.out.println("Hello, " + name + "!");
    }

    public static void main(String[] args) {
        int sum = add(3, 7);     // call with arguments
        System.out.println(sum); // 10

        printGreeting("Alice");  // Hello, Alice!
    }
}
```

---

## Parameters vs Arguments

- **Parameters** are the variables declared in the method signature: `int a, int b`
- **Arguments** are the actual values passed when calling the method: `add(3, 7)`

---

## void Methods

A method with return type `void` performs an action but returns nothing. Using `return` in a void method exits early but carries no value.

```java
public static void printRange(int start, int end) {
    if (start >= end) return;  // early exit
    for (int i = start; i < end; i++) {
        System.out.println(i);
    }
}
```

---

## Method Overloading

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

multiply(2, 3);       // int version → 6
multiply(2.0, 3.5);   // double version → 7.0
multiply(2, 3, 4);    // three-arg version → 24
```

> Return type alone is not enough to distinguish overloads — two methods with the same name and parameters but different return types won't compile.

---

## Varargs

A varargs parameter accepts zero or more arguments of the same type. Declared with `...`, it must be the last parameter.

```java
public static int sum(int... numbers) {
    int total = 0;
    for (int n : numbers) {
        total += n;
    }
    return total;
}

sum();            // 0
sum(1, 2);        // 3
sum(1, 2, 3, 4);  // 10
```

---

## Pass by Value

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

---

## Recursion

A method can call itself. Every recursive method needs a **base case** to stop the recursion.

```java
public static int factorial(int n) {
    if (n <= 1) return 1;          // base case
    return n * factorial(n - 1);   // recursive case
}

System.out.println(factorial(5));  // 120
```

---

## static vs Instance Methods

|                         | `static` method        | Instance method      |
|-------------------------|------------------------|----------------------|
| Belongs to              | The class              | An object (instance) |
| Called via              | `ClassName.method()`   | `object.method()`    |
| Can access static fields | Yes                   | Yes                  |
| Can access instance fields | No                 | Yes                  |

```java
public class Counter {
    private int count = 0;  // instance field

    public void increment() {             // instance method
        count++;
    }

    public int getCount() {               // instance method
        return count;
    }

    public static int max(int a, int b) { // static method
        return a > b ? a : b;
    }
}

Counter c = new Counter();
c.increment();
c.increment();
System.out.println(c.getCount());      // 2
System.out.println(Counter.max(3, 7)); // 7
```

---

[← Back to README](../README.md)
