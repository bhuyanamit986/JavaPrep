# Core Java Fundamentals

## 1) Java compilation model (high level)

- **Source (`.java`) → bytecode (`.class`) → JVM execution**.
- The JVM can interpret bytecode and/or use **JIT compilation** to optimize hot code paths.
- Key interview point: Java is “compiled to bytecode” and **run on a VM**, not compiled directly to native machine code (though it *may* JIT to native during execution).

## 2) Primitives vs references

### Primitives

- Stored as raw values: `int`, `long`, `double`, `boolean`, etc.
- Not `null`.
- Have fixed sizes (per JLS; e.g., `int` is 32-bit).

### References (objects)

- A reference points to an object on the heap.
- A reference can be `null`.
- Two variables can reference the **same** object.

Common pitfall: confusing `==` and `.equals()`:

- `==` compares **primitive values** OR **reference identity**.
- `.equals()` compares **logical equality** (if overridden).

## 3) “Java is pass-by-value” (including objects)

Java always passes **copies of values** into methods.

- For primitives, the value is the primitive itself.
- For objects, the value is a **copy of the reference** (so you can mutate the object, but you can’t rebind the caller’s reference).

Example:

```java
static void mutate(StringBuilder sb) {
  sb.append("x");      // mutates the same object
  sb = new StringBuilder("new"); // rebinds local copy only
}
```

## 4) `String` and immutability

### Why `String` is immutable

- **Security**: safe for classloading, file paths, etc.
- **Caching**: `hashCode()` can be cached.
- **String pool** (interning): identical literals can share storage.
- **Thread-safety**: immutable objects are naturally thread-safe.

### String pool and interning

- String literals are interned.
- `new String("a")` creates a new object even if `"a"` exists in pool.
- `"a" == "a"` is true (same interned literal), but `new String("a") == "a"` is false.

## 5) `equals()` and `hashCode()` contract

If you override `equals`, you must override `hashCode`.

Contract highlights:

- If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` must be true.
- If `a.hashCode() == b.hashCode()`, `a.equals(b)` *may or may not* be true (hash collisions allowed).

Why it matters: hashed collections (`HashMap`, `HashSet`) rely on this.

Common interview pitfall: using mutable fields in `hashCode()` that change after insertion into a `HashSet` → lookup breaks.

## 6) `Comparable` vs `Comparator`

- `Comparable<T>`: natural ordering via `compareTo`.
- `Comparator<T>`: external ordering; can compose and pass to sorting methods.

Rules:

- Comparator should be **consistent with equals** when used in sorted sets/maps (e.g., `TreeSet`, `TreeMap`) to avoid “missing” elements.

## 7) `static`, initialization order, and `final`

### Initialization order (typical)

1. Static fields / static blocks (top to bottom)
2. Instance fields / instance init blocks (top to bottom)
3. Constructor

### `final`

- A `final` variable can be assigned once.
- A `final` reference can’t be rebound, but the object may still be mutable.
- In concurrency: properly constructed immutable objects are safer; `final` fields have special JMM guarantees (publication safety).

## 8) Nested, inner, local, and anonymous classes

- **Static nested class**: no implicit reference to outer instance.
- **Inner class**: has an implicit outer reference (`Outer.this`), can capture outer members.
- **Local/anonymous classes**: defined within methods; can capture effectively-final variables.

Interview angle: inner classes can accidentally keep outer objects alive (memory leak risk).

## 9) Date and time (`java.time`)

Prefer the modern API:

- `Instant` (timestamp), `Duration`
- `LocalDate`, `LocalTime`, `LocalDateTime`
- `ZonedDateTime` for timezone-aware operations

Avoid:

- `java.util.Date` / `Calendar` in new code (legacy, mutable, confusing).

## 10) `Optional` (usage guidelines)

Good:

- Return type to represent “may be absent”.
- Functional-style chaining: `map`, `flatMap`, `orElseGet`.

Bad:

- Using `Optional` for fields (often a code smell).
- Using `orElse(expensive())` when you mean `orElseGet(() -> expensive())`.

## 11) Annotations (basics)

- Compile-time vs runtime retention: `@Retention(RUNTIME)` is available via reflection.
- Common built-ins: `@Override`, `@Deprecated`, `@FunctionalInterface`, `@SuppressWarnings`.

## 12) Common interview questions

- Why is `String` immutable? What is the string pool?
- Explain `equals`/`hashCode` contract with a `HashMap` example.
- Difference between `Comparable` and `Comparator`. What happens in `TreeSet` if comparator treats different objects as equal?
- Is Java pass-by-value or pass-by-reference? Show an example.

