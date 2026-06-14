# Exception Handling

[← Back to README](../README.md)

---

An **exception** is an event that disrupts normal program flow. Java uses a structured mechanism — `try`, `catch`, `finally`, and `throw` — to detect and recover from errors gracefully.

## The Exception Hierarchy

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

## try / catch / finally

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

---

## Catching Multiple Exceptions

```java
try {
    String s = null;
    System.out.println(s.length());  // NullPointerException
    int result = 10 / 0;            // ArithmeticException
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

---

## throw and throws

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

---

## try-with-resources

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

---

## Custom Exceptions

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
    System.out.println(e.getMessage());           // Insufficient funds. Short by: 50.0
    System.out.println("Need: " + e.getAmount()); // Need: 50.0
}
```

---

## Best Practices

- **Catch specific exceptions** — avoid catching `Exception` or `Throwable` unless you genuinely need to.
- **Don't swallow exceptions** — an empty `catch` block hides bugs. At minimum, log the error.
- **Use finally or try-with-resources** — never leave resources open.
- **Throw early, catch late** — validate inputs at the boundary, handle exceptions where you have enough context to recover.
- **Prefer unchecked for programming errors** — use `IllegalArgumentException`, `IllegalStateException`, etc. for things the caller should fix in code.
- **Include context in messages** — `"Index out of bounds"` is less useful than `"Index 7 out of bounds for length 3"`.

---

## Summary

| Keyword              | Purpose                                                  |
|----------------------|----------------------------------------------------------|
| `try`                | Wraps code that may throw                                |
| `catch`              | Handles a specific exception type                        |
| `finally`            | Always executes — used for cleanup                       |
| `throw`              | Manually raises an exception                             |
| `throws`             | Declares that a method may propagate a checked exception |
| try-with-resources   | Auto-closes `AutoCloseable` resources                    |

---

[← Back to README](../README.md)
