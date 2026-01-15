# Core Java Fundamentals

## Definitions

- **Compilation model**: Java source compiles to bytecode, and the JVM interprets or JIT-compiles that bytecode at runtime.
- **Primitive vs reference**: primitives store raw values; references store a pointer to an object on the heap.
- **Pass-by-value**: Java always passes a copy of the value; for objects the copied value is the reference.
- **Immutability**: an immutable object cannot change state after creation (e.g., `String`), which improves safety and sharing.
- **Equality contract**: `equals()` defines logical equality, and `hashCode()` must be consistent with it for hash collections.
- **Ordering**: `Comparable` defines a natural order; `Comparator` defines an external order and can be composed.
- **`static`/`final`**: `static` belongs to the class, not instances; `final` prevents reassignment (but not necessarily mutation).
- **Records / enums / sealed classes**: records are concise data carriers, enums model fixed sets, and sealed classes restrict inheritance.

## Illustrations

- **Pass-by-value**: you give someone a copy of a house address. They can enter the same house (mutate the object), but changing the copy does not move your address.
- **String pool**: like a shared dictionary where identical literals reuse the same stored entry to save memory.
- **`equals` + `hashCode`**: think of `hashCode` as a neighborhood and `equals` as the exact house check; both are needed to find the right object.
- **`static` vs instance**: a `static` field is a shared billboard for the class, while instance fields are personal notebooks for each object.

## Code Examples

```java
static void update(StringBuilder sb) {
    sb.append("x"); // mutates shared object
    sb = new StringBuilder("new"); // rebinds local copy only
}

static boolean safeEquals(Object a, Object b) {
    return java.util.Objects.equals(a, b);
}
```

## Interview Questions

1. Why is Java considered pass-by-value? Show with an object example.
2. What problems happen if `equals()` and `hashCode()` are inconsistent?
3. Why is `String` immutable and what is the String pool?
4. `Comparable` vs `Comparator`: when do you use each?
5. What is the difference between `static`, `final`, and `static final`?
6. What are records and sealed classes, and when would you use them?

---

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

## 12) Wrapper Classes and Autoboxing

### Wrapper Classes

| Primitive | Wrapper |
|-----------|---------|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

### Autoboxing and Unboxing

```java
// Autoboxing: primitive → wrapper
Integer num = 10;  // Compiler: Integer.valueOf(10)

// Unboxing: wrapper → primitive
int value = num;   // Compiler: num.intValue()

// Pitfall: NullPointerException
Integer x = null;
int y = x;  // NPE! Unboxing null
```

### Integer Cache (-128 to 127)

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);  // true (cached)

Integer c = 200;
Integer d = 200;
System.out.println(c == d);  // false (new objects)
System.out.println(c.equals(d));  // true (always use equals!)
```

---

## 13) Varargs

```java
public void print(String... args) {
    for (String arg : args) {
        System.out.println(arg);
    }
}

// Usage
print("one");
print("one", "two", "three");
print(new String[]{"a", "b"});
```

**Rules:**
- Only one varargs parameter allowed
- Must be the last parameter
- Treated as an array internally

---

## 14) Enums

```java
public enum Status {
    PENDING("P", 1),
    ACTIVE("A", 2),
    CLOSED("C", 3);
    
    private final String code;
    private final int value;
    
    Status(String code, int value) {
        this.code = code;
        this.value = value;
    }
    
    public String getCode() { return code; }
    public int getValue() { return value; }
    
    // Find by code
    public static Status fromCode(String code) {
        for (Status s : values()) {
            if (s.code.equals(code)) return s;
        }
        throw new IllegalArgumentException("Unknown code: " + code);
    }
}

// Usage
Status status = Status.ACTIVE;
String name = status.name();      // "ACTIVE"
int ordinal = status.ordinal();   // 1
Status[] all = Status.values();
Status parsed = Status.valueOf("PENDING");
```

**Interview tip**: Enums are the best way to implement Singleton in Java.

---

## 15) Records (Java 14+)

```java
// Traditional class
public class Person {
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String name() { return name; }
    public int age() { return age; }
    // Plus equals(), hashCode(), toString()...
}

// Record (same functionality)
public record Person(String name, int age) {
    // Compact constructor for validation
    public Person {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
    }
    
    // Additional methods
    public String greeting() {
        return "Hello, " + name;
    }
}

// Usage
Person p = new Person("John", 30);
String name = p.name();  // Accessor method
```

---

## 16) Sealed Classes (Java 17+)

```java
// Only permitted classes can extend
public sealed class Shape permits Circle, Rectangle, Triangle {
    // ...
}

public final class Circle extends Shape {
    // Cannot be extended further
}

public sealed class Rectangle extends Shape permits Square {
    // Only Square can extend Rectangle
}

public non-sealed class Triangle extends Shape {
    // Any class can extend Triangle
}

public final class Square extends Rectangle {
    // ...
}
```

---

## 17) Pattern Matching

### instanceof Pattern Matching (Java 16+)

```java
// Old way
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// New way
if (obj instanceof String s) {
    System.out.println(s.length());  // s is already cast
}

// With negation
if (!(obj instanceof String s)) {
    return;
}
// s is in scope here
```

### Switch Pattern Matching (Java 21+)

```java
String format(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + i;
        case String s -> "String: " + s;
        case null -> "Null value";
        default -> "Unknown: " + obj;
    };
}
```

---

## 18) Common interview questions

- Why is `String` immutable? What is the string pool?
- Explain `equals`/`hashCode` contract with a `HashMap` example.
- Difference between `Comparable` and `Comparator`. What happens in `TreeSet` if comparator treats different objects as equal?
- Is Java pass-by-value or pass-by-reference? Show an example.
- What is autoboxing? What pitfalls can it cause?
- What is the Integer cache? Why does `==` sometimes work for Integer comparison?
- Explain the difference between final, finally, and finalize.
- What are Records in Java? When would you use them?
- What are Sealed Classes? What problem do they solve?
- How does pattern matching with instanceof work?

---

## 19) Code Examples

### Proper equals/hashCode Implementation

```java
public class Employee {
    private final String id;
    private final String name;
    private final int age;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee employee = (Employee) o;
        return age == employee.age &&
               Objects.equals(id, employee.id) &&
               Objects.equals(name, employee.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id, name, age);
    }
}
```

### Comparable vs Comparator

```java
// Comparable - natural ordering
public class Person implements Comparable<Person> {
    private String name;
    private int age;
    
    @Override
    public int compareTo(Person other) {
        return this.name.compareTo(other.name);
    }
}

// Comparator - external ordering
Comparator<Person> byAge = Comparator.comparingInt(Person::getAge);
Comparator<Person> byNameThenAge = Comparator
    .comparing(Person::getName)
    .thenComparingInt(Person::getAge);

// Reverse order
Comparator<Person> byAgeDesc = Comparator.comparingInt(Person::getAge).reversed();

// With nulls
Comparator<Person> nullsFirst = Comparator.nullsFirst(byAge);
```

### Optional Best Practices

```java
// Creating
Optional<String> empty = Optional.empty();
Optional<String> of = Optional.of("value");       // NPE if null
Optional<String> nullable = Optional.ofNullable(value);

// Using
String result = optional
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .orElse("default");

// orElse vs orElseGet
optional.orElse(computeExpensive());       // Always computed
optional.orElseGet(() -> computeExpensive()); // Computed only if needed

// Throwing
String value = optional.orElseThrow(() -> 
    new IllegalArgumentException("Value required"));

// ifPresent
optional.ifPresent(System.out::println);
optional.ifPresentOrElse(
    System.out::println,
    () -> System.out.println("Empty")
);
```

