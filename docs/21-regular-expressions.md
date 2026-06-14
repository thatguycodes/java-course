# Regular Expressions

[← Back to README](../README.md)

---

A **regular expression** (regex) is a pattern used to match, search, and manipulate text. Java's regex engine lives in `java.util.regex` — the two core classes are `Pattern` and `Matcher`.

---

## Quick Reference — Regex Syntax

### Character Classes

| Pattern | Matches |
|---------|---------|
| `.` | Any character except newline |
| `\d` | A digit (`[0-9]`) |
| `\D` | A non-digit |
| `\w` | A word character (`[a-zA-Z0-9_]`) |
| `\W` | A non-word character |
| `\s` | Whitespace (space, tab, newline) |
| `\S` | Non-whitespace |
| `[abc]` | `a`, `b`, or `c` |
| `[^abc]` | Any character except `a`, `b`, `c` |
| `[a-z]` | Any lowercase letter |
| `[A-Za-z0-9]` | Any alphanumeric character |

### Quantifiers

| Pattern | Matches |
|---------|---------|
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one |
| `{n}` | Exactly n times |
| `{n,}` | n or more times |
| `{n,m}` | Between n and m times |
| `*?` `+?` `??` | Lazy (non-greedy) versions |

### Anchors and Boundaries

| Pattern | Matches |
|---------|---------|
| `^` | Start of string (or line with `MULTILINE`) |
| `$` | End of string (or line with `MULTILINE`) |
| `\b` | Word boundary |
| `\B` | Non-word boundary |

### Groups and Alternation

| Pattern | Matches |
|---------|---------|
| `(abc)` | Capturing group |
| `(?:abc)` | Non-capturing group |
| `(?<name>abc)` | Named capturing group |
| `a\|b` | `a` or `b` |
| `(?=abc)` | Positive lookahead |
| `(?!abc)` | Negative lookahead |
| `(?<=abc)` | Positive lookbehind |
| `(?<!abc)` | Negative lookbehind |

---

## Pattern and Matcher

```java
import java.util.regex.*;

// compile the pattern once — reuse for multiple matches
Pattern pattern = Pattern.compile("\\d{3}-\\d{4}");
Matcher matcher = pattern.matcher("Call us on 555-1234 or 555-9876");

// find all matches
while (matcher.find()) {
    System.out.println("Found: " + matcher.group());  // 555-1234, then 555-9876
    System.out.println("  at index: " + matcher.start() + "–" + matcher.end());
}
```

### matches() vs find()

```java
Pattern p = Pattern.compile("\\d+");

// matches() — entire string must match the pattern
System.out.println(p.matcher("12345").matches()); // true
System.out.println(p.matcher("123ab").matches()); // false

// find() — looks for the pattern anywhere in the string
System.out.println(p.matcher("123ab").find());    // true
```

---

## Capturing Groups

```java
Pattern datePattern = Pattern.compile("(\\d{4})-(\\d{2})-(\\d{2})");
Matcher m = datePattern.matcher("Today is 2026-06-14.");

if (m.find()) {
    System.out.println("Full match: " + m.group(0));  // 2026-06-14
    System.out.println("Year:  "      + m.group(1));  // 2026
    System.out.println("Month: "      + m.group(2));  // 06
    System.out.println("Day:   "      + m.group(3));  // 14
}

// named groups — more readable
Pattern named = Pattern.compile("(?<year>\\d{4})-(?<month>\\d{2})-(?<day>\\d{2})");
Matcher nm = named.matcher("2026-06-14");

if (nm.matches()) {
    System.out.println(nm.group("year"));   // 2026
    System.out.println(nm.group("month"));  // 06
    System.out.println(nm.group("day"));    // 14
}
```

---

## String Convenience Methods

`String` has regex-powered methods — useful for simple cases without creating `Pattern`/`Matcher`.

```java
String text = "Hello, World! 123";

// matches — whole string must match
System.out.println("12345".matches("\\d+"));        // true
System.out.println("12345".matches("[0-9]{1,5}"));  // true

// replaceAll / replaceFirst
System.out.println(text.replaceAll("\\d", "#"));     // Hello, World! ###
System.out.println(text.replaceFirst("\\w+", "Hi")); // Hi, World! 123

// split
String csv = "one, two,  three,four";
String[] parts = csv.split(",\\s*");  // split on comma + optional whitespace
System.out.println(java.util.Arrays.toString(parts)); // [one, two, three, four]
```

---

## Common Patterns

```java
// email (simplified)
String EMAIL = "^[\\w.+-]+@[\\w-]+\\.[\\w.]+$";
System.out.println("alice@example.com".matches(EMAIL));   // true
System.out.println("not-an-email".matches(EMAIL));        // false

// South African phone number
String SA_PHONE = "^(\\+27|0)[6-8][0-9]{8}$";
System.out.println("0821234567".matches(SA_PHONE));       // true
System.out.println("+27821234567".matches(SA_PHONE));     // true

// URL (simplified)
String URL = "https?://[\\w.-]+(/[\\w./?=%&-]*)?";
System.out.println("https://example.com/path?q=1".matches(URL)); // true

// IPv4 address
String IPV4 = "^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)$";
System.out.println("192.168.1.1".matches(IPV4));  // true
System.out.println("999.0.0.1".matches(IPV4));    // false

// digits only
System.out.println("12345".matches("\\d+"));   // true

// alphanumeric
System.out.println("abc123".matches("[\\w]+"));  // true

// whitespace-only
System.out.println("   ".matches("\\s+"));  // true

// password: 8+ chars, at least one upper, lower, digit, special char
String PASSWORD = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$";
System.out.println("Secret@1".matches(PASSWORD));  // true
System.out.println("weakpass".matches(PASSWORD));  // false
```

---

## Pattern Flags

```java
// case-insensitive matching
Pattern p = Pattern.compile("hello", Pattern.CASE_INSENSITIVE);
System.out.println(p.matcher("Hello World").find());  // true
System.out.println(p.matcher("HELLO").find());        // true

// multiline — ^ and $ match start/end of each line
Pattern ml = Pattern.compile("^\\d+", Pattern.MULTILINE);
Matcher m  = ml.matcher("123 abc\n456 def\n789 ghi");
while (m.find()) System.out.println(m.group());  // 123, 456, 789

// dotall — . matches newlines too
Pattern ds = Pattern.compile("start.*end", Pattern.DOTALL);
System.out.println(ds.matcher("start\nmiddle\nend").matches());  // true

// combine flags
Pattern combined = Pattern.compile("hello", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE);

// inline flags inside the pattern
Pattern inline = Pattern.compile("(?i)hello");  // case-insensitive
```

---

## Find All Matches as a Stream (Java 9+)

```java
Pattern p = Pattern.compile("\\b\\w{5}\\b");  // 5-letter words
Matcher m = p.matcher("Java regex makes things simple and clean");

m.results()
 .map(java.util.regex.MatchResult::group)
 .forEach(System.out::println);
// regex, makes, things, simple, clean
```

---

## Replace with a Function (Java 9+)

```java
Pattern p = Pattern.compile("\\d+");
String result = p.matcher("Price: 100, Discount: 20, Total: 80")
                 .replaceAll(mr -> "$" + mr.group());

System.out.println(result);  // Price: $100, Discount: $20, Total: $80
```

---

## Splitting

```java
// basic split
String[] words = "one two  three".split("\\s+");  // handles multiple spaces

// split with limit — max N parts
String[] parts = "a:b:c:d".split(":", 3);  // ["a", "b", "c:d"]

// using Pattern.split (compiled, reusable)
Pattern comma = Pattern.compile(",\\s*");
String[] items = comma.split("apple, banana,  cherry");
System.out.println(java.util.Arrays.toString(items));  // [apple, banana, cherry]
```

---

## Performance Tips

- **Compile patterns once** — `Pattern.compile()` is expensive; store the result in a `static final` field.
- **Use non-capturing groups `(?:...)` when you don't need the captured value** — slightly faster.
- **Prefer possessive quantifiers `*+` `++`** for patterns that should never backtrack.
- **Avoid catastrophic backtracking** — nested quantifiers like `(a+)+` can cause exponential time on bad input.
- **Use `String` methods for simple cases** — `contains()`, `startsWith()`, `endsWith()` are faster than regex when you don't need pattern power.

```java
// compile once
private static final Pattern DIGITS = Pattern.compile("\\d+");

// reuse
boolean hasDigits = DIGITS.matcher(input).find();
```

---

## Regex Summary

| Task | Method |
|------|--------|
| Check if whole string matches | `pattern.matcher(s).matches()` or `s.matches(regex)` |
| Find first occurrence | `matcher.find()` then `matcher.group()` |
| Find all occurrences | `matcher.results()` (Java 9+) or `while (matcher.find())` loop |
| Extract groups | `matcher.group(n)` or `matcher.group("name")` |
| Replace all matches | `s.replaceAll(regex, replacement)` or `matcher.replaceAll()` |
| Replace with function | `matcher.replaceAll(mr -> ...)` (Java 9+) |
| Split a string | `s.split(regex)` or `pattern.split(s)` |

---

[← Back to README](../README.md)
