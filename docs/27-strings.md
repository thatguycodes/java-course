# Strings

[← Back to README](../README.md)

---

`String` is one of the most-used types in Java. Strings are **immutable** — every operation that appears to modify a string actually creates a new one. For repeated modifications, use `StringBuilder`.

```mermaid
flowchart LR
    literal["String literal\n\"hello\""] --> pool["String Pool\n(heap)"]
    new_["new String(\"hello\")"] --> heap["Heap (outside pool)"]
    pool -->|intern()| pool
```

---

## Creating Strings

```java
// string literal — stored in the string pool
String a = "hello";
String b = "hello";
System.out.println(a == b);       // true  — same pool reference
System.out.println(a.equals(b));  // true

// new String — always creates a new object
String c = new String("hello");
System.out.println(a == c);       // false — different reference
System.out.println(a.equals(c));  // true  — same content

// from char array
char[] chars = {'h', 'e', 'l', 'l', 'o'};
String d = new String(chars);

// from bytes
byte[] bytes = "hello".getBytes(java.nio.charset.StandardCharsets.UTF_8);
String e = new String(bytes, java.nio.charset.StandardCharsets.UTF_8);
```

> Always use `.equals()` to compare string content, never `==`.

---

## Common String Methods

```java
String s = "  Hello, World!  ";

// length and emptiness
s.length();          // 17
s.isEmpty();         // false
s.isBlank();         // false (Java 11+) — true if only whitespace
"".isEmpty();        // true
"   ".isBlank();     // true

// case
s.toUpperCase();     // "  HELLO, WORLD!  "
s.toLowerCase();     // "  hello, world!  "

// trimming
s.trim();            // "Hello, World!"       — strips ASCII whitespace
s.strip();           // "Hello, World!"       — strips Unicode whitespace (Java 11+)
s.stripLeading();    // "Hello, World!  "
s.stripTrailing();   // "  Hello, World!"

// searching
s.contains("World");       // true
s.startsWith("  Hello");   // true
s.endsWith("!  ");         // true
s.indexOf("o");            // 5   — first occurrence
s.lastIndexOf("o");        // 9   — last occurrence
s.indexOf("o", 6);         // 9   — search from index 6

// extracting
s.charAt(2);               // 'H'
s.substring(2);            // "Hello, World!  "
s.substring(2, 7);         // "Hello"

// replacing
s.replace("World", "Java");          // "  Hello, Java!  "
s.replaceAll("\\s+", "_");           // "__Hello,_World!__"  (regex)
s.replaceFirst("[A-Z]", "*");        // "  *ello, World!  " (regex)

// splitting
"a,b,,c".split(",");                 // ["a", "b", "", "c"]
"a,b,,c".split(",", -1);            // ["a", "b", "", "c"]  (keeps trailing)
"one two  three".split("\\s+");     // ["one", "two", "three"]

// joining
String.join(", ", "a", "b", "c");   // "a, b, c"
String.join("-", List.of("x","y")); // "x-y"

// checking content
"abc123".chars().allMatch(Character::isLetterOrDigit);  // true

// repeat (Java 11+)
"ab".repeat(3);  // "ababab"

// lines (Java 11+) — splits on \n, \r, \r\n
"line1\nline2\nline3".lines().toList();  // ["line1", "line2", "line3"]
```

---

## String Formatting

### String.format

```java
String name = "Alice";
int    age  = 30;
double pi   = 3.14159;

String s = String.format("Name: %-10s Age: %3d  Pi: %.2f", name, age, pi);
// "Name: Alice      Age:  30  Pi: 3.14"
```

### formatted (Java 15+)

```java
String s = "Name: %-10s Age: %3d".formatted(name, age);
```

### Common format specifiers

| Specifier | Type | Example |
|-----------|------|---------|
| `%s` | String | `"hello"` |
| `%d` | Integer | `42` |
| `%f` | Float/double | `3.141593` |
| `%.2f` | 2 decimal places | `3.14` |
| `%10s` | Right-pad to width 10 | `"     hello"` |
| `%-10s` | Left-pad to width 10 | `"hello     "` |
| `%05d` | Zero-pad integer | `"00042"` |
| `%n` | Platform newline | |
| `%x` | Hex integer | `"2a"` |
| `%b` | Boolean | `"true"` |

### printf

```java
System.out.printf("%-20s %5d%n", "Alice", 42);
System.out.printf("%-20s %5d%n", "Bob",   7);
// Alice                   42
// Bob                      7
```

---

## Text Blocks (Java 15+)

```java
String json = """
        {
            "name": "Alice",
            "age": 30
        }
        """;

String html = """
        <html>
            <body>
                <p>Hello, %s!</p>
            </body>
        </html>
        """.formatted("Alice");
```

The common leading whitespace is automatically stripped. The closing `"""` position controls indentation.

---

## StringBuilder

Use `StringBuilder` for building strings in a loop or with many concatenations — it mutates a single internal buffer instead of creating new objects.

```java
StringBuilder sb = new StringBuilder();

sb.append("Hello");
sb.append(", ");
sb.append("World");
sb.append("!");

System.out.println(sb.toString());  // "Hello, World!"
System.out.println(sb.length());    // 13

// other methods
sb.insert(5, " there");        // "Hello there, World!"
sb.delete(5, 11);              // "Hello, World!"
sb.replace(7, 12, "Java");     // "Hello, Java!"
sb.reverse();                  // "!avaJ ,olleH"
sb.deleteCharAt(0);            // "avaJ ,olleH"

// chaining
String result = new StringBuilder()
    .append("foo")
    .append("bar")
    .append(42)
    .toString();  // "foobar42"
```

### When to use what

| Situation | Use |
|-----------|-----|
| Simple concatenation (a few strings) | `+` operator or `String.join()` |
| Building in a loop | `StringBuilder` |
| Multi-threaded string building | `StringBuffer` (thread-safe, slower) |
| Fixed template with values | `String.format()` / `formatted()` |
| Multi-line literal | Text block `"""..."""` |

> The `+` operator on strings in a loop compiles to `StringBuilder` in simple cases (Java 9+), but be explicit when building inside complex loops.

---

## String Comparison

```java
String a = "Hello";
String b = "hello";

// content equality
a.equals(b);                    // false
a.equalsIgnoreCase(b);          // true

// lexicographic order
a.compareTo(b);                 // negative (H < h in Unicode)
a.compareToIgnoreCase(b);       // 0

// null-safe comparison
java.util.Objects.equals(a, null);  // false (no NullPointerException)
```

---

## char and Character

```java
String s = "Hello, 123!";

// iterate characters
for (char c : s.toCharArray()) {
    System.out.print(c + " ");
}

// stream of int code points (Java 8+)
s.chars()
 .filter(Character::isDigit)
 .forEach(c -> System.out.print((char) c));  // 123

// Character utility methods
Character.isDigit('5');        // true
Character.isLetter('A');       // true
Character.isWhitespace(' ');   // true
Character.isUpperCase('A');    // true
Character.toLowerCase('Z');    // 'z'
Character.toUpperCase('a');    // 'A'
```

---

## String Interning and Memory

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);          // false
System.out.println(a.intern() == b.intern());  // true — both from pool

// large strings — avoid keeping references longer than needed
// strings are GC'd like any other object when unreferenced
```

---

## String Summary

| Task | Method |
|------|--------|
| Compare content | `.equals()`, `.equalsIgnoreCase()` |
| Search | `.contains()`, `.indexOf()`, `.startsWith()`, `.endsWith()` |
| Extract | `.substring()`, `.charAt()` |
| Modify | `.replace()`, `.replaceAll()`, `.trim()`, `.strip()` |
| Split | `.split(regex)` |
| Join | `String.join(delimiter, ...)` |
| Format | `String.format()`, `.formatted()` |
| Build efficiently | `StringBuilder.append()` |
| Multi-line literal | Text block `"""..."""` |
| Iterate characters | `.toCharArray()`, `.chars()` |

---

[← Back to README](../README.md)
