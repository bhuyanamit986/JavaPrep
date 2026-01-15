# Core Java Fundamentals

This chapter covers the foundational concepts of Java that every developer must know. These topics form the basis for more advanced concepts and are frequently tested in interviews.

---

## Definitions

- **Compilation model**: Java source code (`.java`) is compiled to platform-independent bytecode (`.class`), which the JVM interprets or JIT-compiles at runtime.

- **Primitive types**: The eight basic data types (`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`) that store raw values directly in memory.

- **Reference types**: Variables that store a reference (memory address) to an object on the heap. Includes classes, interfaces, arrays, and enums.

- **Pass-by-value**: Java always passes a copy of the value to methods. For primitives, it's the value itself; for objects, it's a copy of the reference (not the object).

- **Immutability**: An object whose state cannot change after creation. Immutable objects are thread-safe and can be safely shared.

- **String pool**: A special memory region where the JVM stores unique String literals to save memory through interning.

- **equals() contract**: Defines logical equality between objects. Must be reflexive, symmetric, transitive, consistent, and handle null.

- **hashCode() contract**: Returns an integer representation of an object. Equal objects must have equal hash codes.

- **Comparable**: Defines natural ordering for a class via the `compareTo()` method.

- **Comparator**: Defines external ordering that can be passed to sorting methods without modifying the class.

- **static**: Belongs to the class rather than instances. Shared across all instances.

- **final**: Prevents reassignment (variables), overriding (methods), or inheritance (classes).

- **Records**: Immutable data carriers with auto-generated constructors, accessors, `equals()`, `hashCode()`, and `toString()` (Java 14+).

- **Sealed classes**: Classes that restrict which other classes can extend them (Java 17+).

---

## Illustrations

### Java Compilation and Execution Model

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Java Compilation & Execution                         │
│                                                                          │
│   Source Code          Bytecode              Native Code                 │
│   (.java)              (.class)              (Machine)                   │
│                                                                          │
│   ┌──────────┐        ┌──────────┐         ┌──────────┐                 │
│   │ Hello.   │  javac │  Hello.  │   JVM   │ 01001010 │                 │
│   │   java   │───────▶│  class   │────────▶│ 11010010 │                 │
│   │          │        │          │         │ 00101101 │                 │
│   └──────────┘        └──────────┘         └──────────┘                 │
│                            │                     │                       │
│                            │                     │                       │
│                     Platform-Independent    Platform-Specific           │
│                     (Write Once)            (Run Anywhere)              │
│                                                                          │
│   JVM Execution:                                                        │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │  Interpreter ──▶ Bytecode interpreted line by line (slow)   │       │
│   │       │                                                      │       │
│   │       ▼                                                      │       │
│   │  JIT Compiler ──▶ Hot code compiled to native (fast)        │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
```

### Primitive vs Reference Types

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Memory Layout: Stack vs Heap                          │
│                                                                          │
│   STACK (per thread)                    HEAP (shared)                   │
│   ┌─────────────────┐                  ┌─────────────────────────┐      │
│   │ int x = 42;     │                  │                         │      │
│   │ ┌───┐           │                  │  ┌─────────────────┐    │      │
│   │ │42 │ (value)   │                  │  │ Person Object   │    │      │
│   │ └───┘           │                  │  │ ┌─────────────┐ │    │      │
│   │                 │                  │  │ │ name: "John"│ │    │      │
│   │ Person p;       │      ref         │  │ │ age: 30     │ │    │      │
│   │ ┌───┐           │─────────────────▶│  │ └─────────────┘ │    │      │
│   │ │ ● │ (address) │                  │  └─────────────────┘    │      │
│   │ └───┘           │                  │                         │      │
│   │                 │                  │  ┌─────────────────┐    │      │
│   │ String s;       │      ref         │  │ String "Hello" │    │      │
│   │ ┌───┐           │─────────────────▶│  │ (in String Pool)│    │      │
│   │ │ ● │           │                  │  └─────────────────┘    │      │
│   │ └───┘           │                  │                         │      │
│   └─────────────────┘                  └─────────────────────────┘      │
│                                                                          │
│   Primitive: Value stored directly on stack                             │
│   Reference: Address stored on stack, object on heap                    │
└─────────────────────────────────────────────────────────────────────────┘
```

### Pass-by-Value Explained

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Java is ALWAYS Pass-by-Value                         │
│                                                                          │
│   For Primitives:                                                        │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │  main() {              │  modify(int y) {                    │       │
│   │    int x = 10;         │    y = 20;        // y is a COPY   │       │
│   │    modify(x);          │  }                                  │       │
│   │    // x is still 10!   │                                     │       │
│   │  }                     │                                     │       │
│   │                        │                                     │       │
│   │  Stack:                │  Stack:                             │       │
│   │  ┌────┐                │  ┌────┐                             │       │
│   │  │x=10│ ──copy──────▶  │  │y=10│ → y=20                      │       │
│   │  └────┘                │  └────┘                             │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   For Objects:                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │  main() {                    │  modify(Person p2) {          │       │
│   │    Person p1 = new Person(); │    p2.name = "Bob";  // ✓     │       │
│   │    p1.name = "Alice";        │    p2 = new Person();// local │       │
│   │    modify(p1);               │    p2.name = "Carol";// local │       │
│   │    // p1.name is "Bob"!      │  }                            │       │
│   │  }                           │                               │       │
│   │                              │                               │       │
│   │  p1 ─┐     COPY of reference │  p2 ─┐                        │       │
│   │      │    ─────────────────▶ │      │                        │       │
│   │      ▼                       │      ▼                        │       │
│   │  ┌────────┐                  │  ┌────────┐                   │       │
│   │  │ Person │ ◀───────same────────│ Person │                   │       │
│   │  │ Alice  │    object!       │  │  Bob   │                   │       │
│   │  └────────┘                  │  └────────┘                   │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   Key insight: You can MUTATE the object, but you cannot REASSIGN       │
│   the caller's reference to point to a different object.                │
└─────────────────────────────────────────────────────────────────────────┘
```

### String Pool and Interning

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         String Pool (in Heap)                            │
│                                                                          │
│   String s1 = "Hello";       // Goes to pool                            │
│   String s2 = "Hello";       // Same reference from pool                │
│   String s3 = new String("Hello");  // New object on heap               │
│   String s4 = s3.intern();   // Gets pooled reference                   │
│                                                                          │
│   Stack                              Heap                                │
│   ┌──────┐                          ┌────────────────────────────┐      │
│   │ s1 ●─┼───────────────────────┐  │                            │      │
│   ├──────┤                       │  │   String Pool              │      │
│   │ s2 ●─┼───────────────────────┼─▶│   ┌─────────────┐          │      │
│   ├──────┤                       │  │   │  "Hello"    │          │      │
│   │ s3 ●─┼───────────────────┐   │  │   └─────────────┘          │      │
│   ├──────┤                   │   │  │          ▲                 │      │
│   │ s4 ●─┼───────────────────┼───┘  │          │                 │      │
│   └──────┘                   │      │   ┌──────┴──────┐          │      │
│                              └─────▶│   │   "Hello"   │ (copy)   │      │
│                                     │   └─────────────┘          │      │
│                                     └────────────────────────────┘      │
│                                                                          │
│   s1 == s2  → true  (same pooled object)                                │
│   s1 == s3  → false (different objects)                                 │
│   s1 == s4  → true  (intern() returns pooled reference)                 │
│   s1.equals(s3) → true (same content)                                   │
└─────────────────────────────────────────────────────────────────────────┘
```

### equals() and hashCode() Contract

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    equals() and hashCode() Relationship                  │
│                                                                          │
│   Rule 1: If a.equals(b) is TRUE, then hashCode must be EQUAL           │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │  a.equals(b) == true  ───▶  a.hashCode() == b.hashCode()  │         │
│   └───────────────────────────────────────────────────────────┘         │
│                                                                          │
│   Rule 2: If hashCode is DIFFERENT, equals must be FALSE                │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │  a.hashCode() != b.hashCode()  ───▶  a.equals(b) == false │         │
│   └───────────────────────────────────────────────────────────┘         │
│                                                                          │
│   Note: Same hashCode does NOT guarantee equals (collisions allowed)    │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │  a.hashCode() == b.hashCode()  ─/─▶  a.equals(b) == ???   │         │
│   │                                       (may be true or false)│         │
│   └───────────────────────────────────────────────────────────┘         │
│                                                                          │
│   HashMap Lookup Process:                                               │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │  1. Compute hashCode() → bucket index                        │       │
│   │  2. Search bucket using equals() for exact match             │       │
│   │                                                              │       │
│   │  Buckets:  [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]           │       │
│   │             │         │                   │                  │       │
│   │             ▼         ▼                   ▼                  │       │
│   │           ┌───┐     ┌───┐               ┌───┐                │       │
│   │           │ A │     │ C │               │ E │                │       │
│   │           └─┬─┘     └─┬─┘               └───┘                │       │
│   │             │         │                                      │       │
│   │           ┌─▼─┐     ┌─▼─┐    (collision: same hash,          │       │
│   │           │ B │     │ D │     different objects)             │       │
│   │           └───┘     └───┘                                    │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
```

### Comparable vs Comparator

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Comparable vs Comparator                              │
│                                                                          │
│   Comparable<T>                       Comparator<T>                      │
│   ┌─────────────────────────┐        ┌─────────────────────────┐        │
│   │ • Natural ordering       │        │ • External ordering      │        │
│   │ • Implemented IN class   │        │ • Separate from class    │        │
│   │ • One per class          │        │ • Multiple comparators   │        │
│   │ • compareTo(T other)     │        │ • compare(T o1, T o2)    │        │
│   └─────────────────────────┘        └─────────────────────────┘        │
│                                                                          │
│   class Person implements Comparable<Person> {                          │
│       public int compareTo(Person other) {                              │
│           return this.name.compareTo(other.name);  // Natural: by name  │
│       }                                                                 │
│   }                                                                     │
│                                                                          │
│   // Multiple Comparators for different orderings:                      │
│   Comparator<Person> byAge = Comparator.comparingInt(Person::getAge);   │
│   Comparator<Person> bySalary = Comparator.comparing(Person::getSalary);│
│   Comparator<Person> byNameDesc = Comparator.comparing(Person::getName) │
│                                            .reversed();                 │
│                                                                          │
│   Return Values:                                                        │
│   ┌──────────────────────────────────────────────────────────┐         │
│   │  Negative  →  this < other  (or o1 < o2)                 │         │
│   │  Zero      →  this == other (or o1 == o2)                │         │
│   │  Positive  →  this > other  (or o1 > o2)                 │         │
│   └──────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────────┘
```

### static vs Instance Members

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    static vs Instance Members                            │
│                                                                          │
│   class Counter {                                                        │
│       static int totalCount = 0;    // Shared by ALL instances          │
│       int instanceCount = 0;        // Unique per instance              │
│                                                                          │
│       Counter() {                                                        │
│           totalCount++;                                                  │
│           instanceCount++;                                               │
│       }                                                                  │
│   }                                                                      │
│                                                                          │
│   Memory Layout:                                                        │
│   ┌────────────────────────────────────────────────────────────┐        │
│   │  Method Area (Class Data)          Heap (Instances)        │        │
│   │  ┌──────────────────┐              ┌────────────────┐      │        │
│   │  │ Counter.class    │              │ Counter obj1   │      │        │
│   │  │ ┌──────────────┐ │              │ instanceCount=1│      │        │
│   │  │ │totalCount = 3│◀┼──────┐       └────────────────┘      │        │
│   │  │ └──────────────┘ │      │       ┌────────────────┐      │        │
│   │  │ static methods   │      │       │ Counter obj2   │      │        │
│   │  └──────────────────┘      ├───────│ instanceCount=1│      │        │
│   │                            │       └────────────────┘      │        │
│   │                            │       ┌────────────────┐      │        │
│   │                            │       │ Counter obj3   │      │        │
│   │                            └───────│ instanceCount=1│      │        │
│   │                                    └────────────────┘      │        │
│   └────────────────────────────────────────────────────────────┘        │
│                                                                          │
│   After creating 3 Counter instances:                                   │
│   • Counter.totalCount = 3  (shared, accessed via class)               │
│   • obj1.instanceCount = 1  (each has its own)                         │
│   • obj2.instanceCount = 1                                              │
│   • obj3.instanceCount = 1                                              │
└─────────────────────────────────────────────────────────────────────────┘
```

### Initialization Order

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Class and Instance Initialization Order               │
│                                                                          │
│   1. Static members (when class is first loaded)                        │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │  ① Static field declarations & initializers (top to bottom) │       │
│   │  ② Static initializer blocks (top to bottom)                │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                        ↓ (Only once per class)                          │
│                                                                          │
│   2. Instance members (each time new object created)                    │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │  ③ Instance field declarations & initializers (top to bottom)│       │
│   │  ④ Instance initializer blocks (top to bottom)              │       │
│   │  ⑤ Constructor body                                         │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   class Example {                                                        │
│       static int a = 1;              // ①                               │
│       static { System.out.println("Static block"); }  // ②              │
│       int b = 2;                     // ③                               │
│       { System.out.println("Instance block"); }  // ④                   │
│       Example() { System.out.println("Constructor"); }  // ⑤            │
│   }                                                                      │
│                                                                          │
│   Output for: new Example();                                            │
│   Static block    (first time class loaded)                             │
│   Instance block                                                        │
│   Constructor                                                           │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Code Examples

### 1) Primitives and Wrapper Classes

```java
// Primitive types and their sizes
byte b = 127;                    // 8-bit, -128 to 127
short s = 32767;                 // 16-bit
int i = 2_147_483_647;           // 32-bit (underscores for readability)
long l = 9_223_372_036_854_775_807L;  // 64-bit (L suffix)
float f = 3.14f;                 // 32-bit (f suffix required)
double d = 3.141592653589793;    // 64-bit (default for decimals)
char c = 'A';                    // 16-bit Unicode
boolean bool = true;             // true or false

// Default values (for fields, not local variables)
class Defaults {
    int intField;           // 0
    double doubleField;     // 0.0
    boolean boolField;      // false
    char charField;         // '\u0000'
    String stringField;     // null (reference type)
}

// Wrapper classes and autoboxing
Integer wrapped = 100;           // Autoboxing: int → Integer
int unwrapped = wrapped;         // Unboxing: Integer → int

// Integer cache (-128 to 127)
Integer a = 100;
Integer b = 100;
System.out.println(a == b);      // true (cached)

Integer c = 200;
Integer d = 200;
System.out.println(c == d);      // false (not cached)
System.out.println(c.equals(d)); // true (always use equals!)

// NullPointerException pitfall
Integer nullInt = null;
int primitive = nullInt;         // NPE! Unboxing null

// Safe unboxing
int safe = (nullInt != null) ? nullInt : 0;
int safe2 = Objects.requireNonNullElse(nullInt, 0);

// Parsing strings
int parsed = Integer.parseInt("123");
int parsedRadix = Integer.parseInt("FF", 16);  // 255 (hex)
Integer parsedOrNull = Integer.valueOf("123");

// Number comparisons
int cmp = Integer.compare(5, 10);  // -1 (5 < 10)
int max = Math.max(5, 10);         // 10
int min = Math.min(5, 10);         // 5
```

### 2) String Operations and Immutability

```java
// String creation
String literal = "Hello";              // String pool
String constructed = new String("Hello");  // New heap object
String fromChars = new String(new char[]{'H', 'e', 'l', 'l', 'o'});

// Why String is immutable:
// 1. Security - safe for classloading, network connections
// 2. Thread safety - can be shared without synchronization
// 3. Caching - hashCode can be cached
// 4. String pool - identical strings can share storage

// String methods (all return NEW strings)
String s = "  Hello World  ";
s.length();                     // 15
s.charAt(2);                    // 'H'
s.indexOf("World");             // 8
s.lastIndexOf('o');             // 10
s.substring(2, 7);              // "Hello"
s.toLowerCase();                // "  hello world  "
s.toUpperCase();                // "  HELLO WORLD  "
s.trim();                       // "Hello World" (removes leading/trailing whitespace)
s.strip();                      // "Hello World" (Unicode-aware, Java 11+)
s.stripLeading();               // "Hello World  "
s.stripTrailing();              // "  Hello World"
s.replace('o', '0');            // "  Hell0 W0rld  "
s.replace("World", "Java");     // "  Hello Java  "
s.replaceAll("\\s+", "-");      // "-Hello-World-" (regex)
s.split("\\s+");                // ["", "Hello", "World", ""]
s.contains("World");            // true
s.startsWith("  He");           // true
s.endsWith("  ");               // true
s.isEmpty();                    // false
s.isBlank();                    // false (whitespace-only = true)
s.repeat(2);                    // "  Hello World    Hello World  "

// String comparison
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");

s1 == s2;                       // true (same pool reference)
s1 == s3;                       // false (different objects)
s1.equals(s3);                  // true (same content)
s1.equalsIgnoreCase("HELLO");   // true
s1.compareTo("Hello");          // 0 (equal)
s1.compareTo("Apple");          // positive (H > A)
s1.compareToIgnoreCase("HELLO"); // 0

// String concatenation
String concat1 = "Hello" + " " + "World";  // Compiler optimizes literals
String concat2 = s1 + s2;       // Creates new String object

// StringBuilder (mutable, single-threaded)
StringBuilder sb = new StringBuilder();
sb.append("Hello");
sb.append(" ");
sb.append("World");
sb.insert(5, ",");              // "Hello, World"
sb.delete(5, 6);                // "Hello World"
sb.reverse();                   // "dlroW olleH"
sb.setCharAt(0, 'D');           // "DlroW olleH"
String result = sb.toString();

// StringBuffer (mutable, thread-safe, slower)
StringBuffer sbuf = new StringBuffer();
sbuf.append("Thread-safe");

// String formatting
String formatted = String.format("Name: %s, Age: %d", "John", 30);
String formatted2 = "Name: %s, Age: %d".formatted("John", 30);  // Java 15+

// Text blocks (Java 15+)
String json = """
    {
        "name": "John",
        "age": 30
    }
    """;

// String interning
String interned = new String("Hello").intern();  // Returns pooled reference
"Hello" == interned;            // true
```

### 3) equals() and hashCode() Implementation

```java
public class Person {
    private final String name;
    private final int age;
    private final String email;
    
    public Person(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    @Override
    public boolean equals(Object o) {
        // 1. Same reference check (optimization)
        if (this == o) return true;
        
        // 2. Null and type check
        if (o == null || getClass() != o.getClass()) return false;
        
        // 3. Cast and compare fields
        Person person = (Person) o;
        return age == person.age &&
               Objects.equals(name, person.name) &&
               Objects.equals(email, person.email);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age, email);
    }
    
    // Alternative: manual hashCode implementation
    // @Override
    // public int hashCode() {
    //     int result = name != null ? name.hashCode() : 0;
    //     result = 31 * result + age;
    //     result = 31 * result + (email != null ? email.hashCode() : 0);
    //     return result;
    // }
}

// Using instanceof (allows subclass equality - sometimes desired)
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Person)) return false;  // Allows subclasses
    Person person = (Person) o;
    return age == person.age && Objects.equals(name, person.name);
}

// Pattern matching (Java 16+)
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Person person)) return false;  // Pattern matching
    return age == person.age && Objects.equals(name, person.name);
}

// Common mistakes:
// ❌ Wrong signature (doesn't override Object.equals)
public boolean equals(Person other) {  // Should be Object!
    return this.name.equals(other.name);
}

// ❌ Using mutable fields in hashCode
public class BadKey {
    private String name;  // Mutable!
    
    public void setName(String name) {
        this.name = name;
    }
    
    @Override
    public int hashCode() {
        return name.hashCode();  // Changes if name changes!
    }
}

// Problem: Key becomes "lost" in HashMap after mutation
Map<BadKey, String> map = new HashMap<>();
BadKey key = new BadKey();
key.setName("original");
map.put(key, "value");

key.setName("changed");  // hashCode changes!
map.get(key);            // null! Can't find the key anymore
```

### 4) Comparable and Comparator

```java
// Comparable - natural ordering
public class Employee implements Comparable<Employee> {
    private String name;
    private int salary;
    private LocalDate hireDate;
    
    // Natural order: by name
    @Override
    public int compareTo(Employee other) {
        return this.name.compareTo(other.name);
    }
}

// Usage
List<Employee> employees = new ArrayList<>();
Collections.sort(employees);          // Uses natural order (by name)
employees.sort(null);                 // Same as above

// Comparator - external ordering
Comparator<Employee> bySalary = new Comparator<>() {
    @Override
    public int compare(Employee e1, Employee e2) {
        return Integer.compare(e1.getSalary(), e2.getSalary());
    }
};

// Modern Comparator creation (Java 8+)
Comparator<Employee> bySalaryLambda = (e1, e2) -> 
    Integer.compare(e1.getSalary(), e2.getSalary());

Comparator<Employee> bySalaryMethod = Comparator.comparingInt(Employee::getSalary);

// Chained comparators
Comparator<Employee> complex = Comparator
    .comparing(Employee::getDepartment)           // First by department
    .thenComparing(Employee::getName)             // Then by name
    .thenComparingInt(Employee::getSalary)        // Then by salary
    .reversed();                                   // Descending order

// Null handling
Comparator<Employee> nullSafe = Comparator
    .nullsFirst(Comparator.comparing(Employee::getName));

Comparator<Employee> nullsLast = Comparator
    .nullsLast(Comparator.comparing(Employee::getName));

// Comparing with null-safe field access
Comparator<Employee> safeCompare = Comparator.comparing(
    Employee::getManager,                          // May return null
    Comparator.nullsLast(Comparator.naturalOrder())
);

// Usage with collections
employees.sort(bySalary);
employees.sort(bySalary.reversed());
Collections.sort(employees, complex);

// In TreeSet/TreeMap (must be consistent with equals)
TreeSet<Employee> sortedSet = new TreeSet<>(bySalary);

// Binary search requires same comparator
Collections.binarySearch(employees, targetEmployee, bySalary);
```

### 5) static and final Keywords

```java
// static field - shared across all instances
public class Counter {
    private static int totalCount = 0;
    private int instanceCount = 0;
    
    public Counter() {
        totalCount++;
        instanceCount++;
    }
    
    // static method - belongs to class
    public static int getTotalCount() {
        return totalCount;
        // Cannot access instanceCount here - no 'this' context
    }
    
    // static block - runs when class is loaded
    static {
        System.out.println("Class loaded!");
        // Initialize complex static fields
    }
}

// Accessing static members
Counter.getTotalCount();          // Preferred: via class name
Counter c = new Counter();
c.getTotalCount();                // Works but discouraged

// static import
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;

double result = sqrt(PI);         // No Math. prefix needed

// final variable - cannot be reassigned
final int MAX_SIZE = 100;
// MAX_SIZE = 200;                // Compile error!

// final reference - reference cannot change, object can be modified
final List<String> list = new ArrayList<>();
list.add("item");                 // OK - modifying object
// list = new ArrayList<>();      // Compile error - changing reference

// final parameter
public void process(final String input) {
    // input = "other";           // Compile error!
    System.out.println(input);
}

// final method - cannot be overridden
public class Parent {
    public final void doSomething() {
        System.out.println("Cannot override this");
    }
}

// final class - cannot be extended
public final class ImmutableClass {
    private final String value;
    
    public ImmutableClass(String value) {
        this.value = value;
    }
}
// class Subclass extends ImmutableClass {}  // Compile error!

// static final - constants
public class Constants {
    public static final int MAX_USERS = 100;
    public static final String APP_NAME = "MyApp";
    
    // Constant naming convention: UPPER_SNAKE_CASE
}
```

### 6) Nested Classes

```java
// 1. Static nested class - no reference to outer instance
public class Outer {
    private static int staticField = 1;
    private int instanceField = 2;
    
    public static class StaticNested {
        public void method() {
            System.out.println(staticField);      // Can access static
            // System.out.println(instanceField); // Cannot access instance
        }
    }
}

// Usage
Outer.StaticNested nested = new Outer.StaticNested();

// 2. Inner class (non-static nested) - has reference to outer instance
public class Outer {
    private int value = 10;
    
    public class Inner {
        public void method() {
            System.out.println(value);            // Can access outer's members
            System.out.println(Outer.this.value); // Explicit outer reference
        }
    }
}

// Usage - requires outer instance
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();

// 3. Local class - defined inside a method
public class Outer {
    public void method() {
        final int localVar = 5;  // Must be effectively final
        
        class LocalClass {
            public void print() {
                System.out.println(localVar);
            }
        }
        
        LocalClass local = new LocalClass();
        local.print();
    }
}

// 4. Anonymous class - class without a name
public class Outer {
    public void method() {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("Anonymous class");
            }
        };
        
        // Anonymous class extending abstract class
        AbstractClass obj = new AbstractClass() {
            @Override
            void abstractMethod() {
                System.out.println("Implementation");
            }
        };
    }
}

// Memory leak warning: Inner classes hold reference to outer
public class LeakExample {
    private byte[] largeData = new byte[1000000];
    
    public Runnable createTask() {
        return new Runnable() {  // Holds reference to LeakExample.this
            public void run() {
                // Even if largeData not used, LeakExample cannot be GC'd
            }
        };
    }
    
    // Solution: Use static nested class or lambda
    public Runnable createTaskSafe() {
        return () -> System.out.println("No outer reference");
    }
}
```

### 7) Enums

```java
// Basic enum
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// Enum usage
Day today = Day.MONDAY;
Day[] allDays = Day.values();
Day parsed = Day.valueOf("MONDAY");
String name = today.name();        // "MONDAY"
int ordinal = today.ordinal();     // 0

// Switch with enum
switch (today) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> System.out.println("Weekday");
    case SATURDAY, SUNDAY -> System.out.println("Weekend");
}

// Enum with fields and methods
public enum Planet {
    MERCURY(3.303e+23, 2.4397e6),
    VENUS(4.869e+24, 6.0518e6),
    EARTH(5.976e+24, 6.37814e6),
    MARS(6.421e+23, 3.3972e6);
    
    private final double mass;    // in kilograms
    private final double radius;  // in meters
    
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
    }
    
    public double getMass() { return mass; }
    public double getRadius() { return radius; }
    
    // Gravitational constant
    public static final double G = 6.67300E-11;
    
    public double surfaceGravity() {
        return G * mass / (radius * radius);
    }
    
    public double surfaceWeight(double otherMass) {
        return otherMass * surfaceGravity();
    }
}

// Enum with abstract method
public enum Operation {
    ADD {
        @Override
        public double apply(double x, double y) { return x + y; }
    },
    SUBTRACT {
        @Override
        public double apply(double x, double y) { return x - y; }
    },
    MULTIPLY {
        @Override
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE {
        @Override
        public double apply(double x, double y) { return x / y; }
    };
    
    public abstract double apply(double x, double y);
}

// Usage
double result = Operation.ADD.apply(5, 3);  // 8.0

// Enum implementing interface
public enum PaymentMethod implements Payable {
    CREDIT_CARD {
        @Override
        public void pay(double amount) {
            System.out.println("Paying " + amount + " with credit card");
        }
    },
    PAYPAL {
        @Override
        public void pay(double amount) {
            System.out.println("Paying " + amount + " with PayPal");
        }
    }
}

// EnumSet and EnumMap (optimized for enums)
EnumSet<Day> weekend = EnumSet.of(Day.SATURDAY, Day.SUNDAY);
EnumSet<Day> weekdays = EnumSet.range(Day.MONDAY, Day.FRIDAY);
EnumSet<Day> allDays = EnumSet.allOf(Day.class);

EnumMap<Day, String> schedule = new EnumMap<>(Day.class);
schedule.put(Day.MONDAY, "Work");
schedule.put(Day.SATURDAY, "Rest");

// Enum singleton pattern (thread-safe, serialization-safe)
public enum Singleton {
    INSTANCE;
    
    private final Connection connection;
    
    Singleton() {
        connection = createConnection();
    }
    
    public Connection getConnection() {
        return connection;
    }
}
```

### 8) Records (Java 14+)

```java
// Basic record - immutable data carrier
public record Point(int x, int y) { }

// What the compiler generates:
// - private final fields
// - canonical constructor
// - accessor methods (x(), y() - not getX())
// - equals(), hashCode(), toString()

// Usage
Point p = new Point(10, 20);
int x = p.x();                    // Accessor (not getX!)
System.out.println(p);            // Point[x=10, y=20]
Point p2 = new Point(10, 20);
p.equals(p2);                     // true

// Record with validation (compact constructor)
public record Person(String name, int age) {
    public Person {  // Compact constructor - no parameter list
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
        // Implicit: this.name = name; this.age = age;
    }
}

// Record with additional constructor
public record Range(int start, int end) {
    public Range {
        if (start > end) {
            throw new IllegalArgumentException("start must be <= end");
        }
    }
    
    // Additional constructor must delegate to canonical
    public Range(int singleValue) {
        this(singleValue, singleValue);
    }
}

// Record with additional methods
public record Rectangle(double width, double height) {
    public double area() {
        return width * height;
    }
    
    public double perimeter() {
        return 2 * (width + height);
    }
    
    // Static factory method
    public static Rectangle square(double side) {
        return new Rectangle(side, side);
    }
}

// Record implementing interface
public record Employee(String name, int salary) implements Comparable<Employee> {
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }
}

// Records cannot:
// - extend other classes (implicitly extend Record)
// - be extended (implicitly final)
// - have instance fields other than components
// - have mutable components (can have mutable objects, but should avoid)

// Record with mutable component (anti-pattern)
public record BadRecord(List<String> items) {
    // items list can be modified! This breaks immutability.
}

// Better: defensive copy
public record GoodRecord(List<String> items) {
    public GoodRecord {
        items = List.copyOf(items);  // Immutable copy
    }
}
```

### 9) Sealed Classes (Java 17+)

```java
// Sealed class - restricts which classes can extend it
public sealed class Shape permits Circle, Rectangle, Triangle {
    // Common shape properties
}

// Permitted subclasses must be final, sealed, or non-sealed
public final class Circle extends Shape {
    private final double radius;
    public Circle(double radius) { this.radius = radius; }
}

public sealed class Rectangle extends Shape permits Square {
    protected final double width, height;
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
}

public final class Square extends Rectangle {
    public Square(double side) {
        super(side, side);
    }
}

public non-sealed class Triangle extends Shape {
    // Any class can extend Triangle
    private final double a, b, c;
    public Triangle(double a, double b, double c) {
        this.a = a; this.b = b; this.c = c;
    }
}

// Sealed interface
public sealed interface Vehicle permits Car, Truck, Motorcycle {
    void start();
}

public final class Car implements Vehicle {
    @Override
    public void start() { System.out.println("Car starting"); }
}

// Pattern matching with sealed classes (Java 21+)
public double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.getRadius() * c.getRadius();
        case Rectangle r -> r.getWidth() * r.getHeight();
        case Triangle t -> calculateTriangleArea(t);
        // No default needed - compiler knows all permitted types
    };
}

// Benefits of sealed classes:
// 1. Exhaustive pattern matching - compiler knows all subtypes
// 2. Domain modeling - explicitly define type hierarchy
// 3. Security - prevent unauthorized extensions
// 4. API design - control extension points
```

### 10) Pattern Matching (Java 16+)

```java
// Traditional instanceof
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// Pattern matching instanceof (Java 16+)
if (obj instanceof String s) {
    System.out.println(s.length());  // s already cast
}

// With negation
if (!(obj instanceof String s)) {
    return;  // Early exit
}
// s is in scope here!
System.out.println(s.length());

// In conditional expressions
if (obj instanceof String s && s.length() > 5) {
    // s is in scope and length > 5
}

// Switch pattern matching (Java 21+)
String describe(Object obj) {
    return switch (obj) {
        case null -> "null";
        case Integer i -> "Integer: " + i;
        case Long l -> "Long: " + l;
        case Double d -> "Double: " + d;
        case String s -> "String: " + s;
        case int[] arr -> "int array of length " + arr.length;
        case Person p -> "Person: " + p.name();
        default -> "Unknown: " + obj.getClass();
    };
}

// Guarded patterns
String categorize(Object obj) {
    return switch (obj) {
        case Integer i when i < 0 -> "negative integer";
        case Integer i when i == 0 -> "zero";
        case Integer i -> "positive integer";
        case String s when s.isEmpty() -> "empty string";
        case String s -> "non-empty string: " + s;
        default -> "other";
    };
}

// Record patterns (Java 21+)
record Point(int x, int y) {}

void printCoordinates(Object obj) {
    if (obj instanceof Point(int x, int y)) {
        System.out.println("x=" + x + ", y=" + y);
    }
}

// Nested record patterns
record Rectangle(Point topLeft, Point bottomRight) {}

String describeRectangle(Object obj) {
    return switch (obj) {
        case Rectangle(Point(int x1, int y1), Point(int x2, int y2)) 
            -> "Rectangle from (" + x1 + "," + y1 + ") to (" + x2 + "," + y2 + ")";
        default -> "Not a rectangle";
    };
}
```

### 11) Optional

```java
import java.util.Optional;

// Creating Optional
Optional<String> empty = Optional.empty();
Optional<String> present = Optional.of("Hello");        // NPE if null
Optional<String> nullable = Optional.ofNullable(null);  // OK with null

// Checking presence
if (present.isPresent()) {
    System.out.println(present.get());
}
if (empty.isEmpty()) {  // Java 11+
    System.out.println("Empty");
}

// Getting values
String value1 = present.get();                          // Throws if empty!
String value2 = present.orElse("default");              // Default if empty
String value3 = present.orElseGet(() -> computeDefault()); // Lazy default
String value4 = present.orElseThrow();                  // NoSuchElementException
String value5 = present.orElseThrow(() -> new RuntimeException("Not found"));

// ❌ orElse vs orElseGet pitfall
optional.orElse(expensiveOperation());       // ALWAYS computed!
optional.orElseGet(() -> expensiveOperation()); // Only if empty

// Transforming
Optional<Integer> length = present.map(String::length);
Optional<String> filtered = present.filter(s -> s.length() > 3);

// Chaining
Optional<String> result = fetchUser(id)
    .flatMap(User::getAddress)                // Optional<Address>
    .flatMap(Address::getCity)                // Optional<City>
    .map(City::getName);                      // Optional<String>

// ifPresent and ifPresentOrElse
present.ifPresent(System.out::println);
present.ifPresentOrElse(
    System.out::println,
    () -> System.out.println("Empty!")
);

// or() - chain Optionals (Java 9+)
Optional<String> value = primary
    .or(() -> secondary)
    .or(() -> Optional.of("default"));

// stream() - convert to Stream (Java 9+)
List<String> values = optionals.stream()
    .flatMap(Optional::stream)  // Filter out empties
    .toList();

// Best practices:
// ✅ Use for return types
public Optional<User> findUser(long id) {
    return Optional.ofNullable(userMap.get(id));
}

// ❌ Don't use for parameters
public void process(Optional<String> input) {  // Bad!
    // Use method overloading instead
}

// ❌ Don't use for fields
public class Bad {
    private Optional<String> name;  // Bad! Just use nullable field
}

// ❌ Don't use Optional.get() without checking
optional.get();  // May throw NoSuchElementException!

// ✅ Use safe methods
optional.orElse("default");
optional.ifPresent(value -> process(value));
```

### 12) Date and Time API (java.time)

```java
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.time.temporal.ChronoUnit;

// Current date and time
LocalDate today = LocalDate.now();              // 2024-01-15
LocalTime now = LocalTime.now();                // 14:30:45.123456789
LocalDateTime dateTime = LocalDateTime.now();   // 2024-01-15T14:30:45
Instant instant = Instant.now();                // Timestamp (UTC)
ZonedDateTime zoned = ZonedDateTime.now();      // With timezone

// Creating specific dates/times
LocalDate date = LocalDate.of(2024, 1, 15);
LocalDate date2 = LocalDate.of(2024, Month.JANUARY, 15);
LocalTime time = LocalTime.of(14, 30, 45);
LocalDateTime dt = LocalDateTime.of(2024, 1, 15, 14, 30);

// Parsing
LocalDate parsed = LocalDate.parse("2024-01-15");
LocalDateTime parsed2 = LocalDateTime.parse("2024-01-15T14:30:45");
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate custom = LocalDate.parse("15/01/2024", formatter);

// Formatting
String formatted = today.format(DateTimeFormatter.ISO_DATE);
String custom2 = dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm"));

// Date manipulation (immutable - returns new instance)
LocalDate tomorrow = today.plusDays(1);
LocalDate nextMonth = today.plusMonths(1);
LocalDate lastYear = today.minusYears(1);
LocalDate adjusted = today.withMonth(6).withDayOfMonth(15);

// Getting date parts
int year = today.getYear();
Month month = today.getMonth();
int day = today.getDayOfMonth();
DayOfWeek dayOfWeek = today.getDayOfWeek();
int dayOfYear = today.getDayOfYear();
boolean isLeapYear = today.isLeapYear();

// Comparing
boolean isBefore = date1.isBefore(date2);
boolean isAfter = date1.isAfter(date2);
boolean isEqual = date1.isEqual(date2);

// Period (date-based) and Duration (time-based)
Period period = Period.between(date1, date2);
int years = period.getYears();
int months = period.getMonths();
int days = period.getDays();

Duration duration = Duration.between(time1, time2);
long hours = duration.toHours();
long minutes = duration.toMinutes();
long seconds = duration.getSeconds();

// ChronoUnit for differences
long daysBetween = ChronoUnit.DAYS.between(date1, date2);
long hoursBetween = ChronoUnit.HOURS.between(dateTime1, dateTime2);

// Time zones
ZoneId zone = ZoneId.of("America/New_York");
ZonedDateTime nyTime = ZonedDateTime.now(zone);
ZonedDateTime utc = ZonedDateTime.now(ZoneId.of("UTC"));

// Converting between zones
ZonedDateTime tokyo = nyTime.withZoneSameInstant(ZoneId.of("Asia/Tokyo"));

// Instant (machine timestamp)
Instant now2 = Instant.now();
long epochSecond = now2.getEpochSecond();
long epochMilli = now2.toEpochMilli();
Instant fromEpoch = Instant.ofEpochMilli(1705312245000L);

// Converting from legacy Date
Date legacyDate = new Date();
Instant fromLegacy = legacyDate.toInstant();
LocalDateTime fromLegacy2 = LocalDateTime.ofInstant(fromLegacy, ZoneId.systemDefault());

// Avoid legacy classes!
// java.util.Date - mutable, confusing API
// java.util.Calendar - complex, error-prone
// java.text.SimpleDateFormat - not thread-safe
```

---

## Common Pitfalls

### 1. String Comparison with ==

```java
// ❌ Wrong
String s1 = "Hello";
String s2 = new String("Hello");
if (s1 == s2) {  // false! Different objects
    // ...
}

// ✅ Correct
if (s1.equals(s2)) {  // true! Same content
    // ...
}

// Or null-safe:
if (Objects.equals(s1, s2)) {
    // ...
}
```

### 2. Integer Cache Confusion

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);  // true (cached -128 to 127)

Integer c = 200;
Integer d = 200;
System.out.println(c == d);  // false (not cached!)

// Always use equals() for wrapper types
System.out.println(c.equals(d));  // true
```

### 3. Mutable hashCode Fields

```java
// ❌ Problem: Key lost after mutation
Set<Person> set = new HashSet<>();
Person p = new Person("John");
set.add(p);
p.setName("Jane");  // Changes hashCode!
set.contains(p);    // false! Can't find it

// ✅ Solution: Use immutable fields for equals/hashCode
public record Person(String name) { }  // Immutable by design
```

### 4. StringBuilder vs String in Loops

```java
// ❌ Creates many String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // Creates new String each iteration!
}

// ✅ Use StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

---

## Interview Questions

### Basic Questions

1. **What is the difference between primitive types and reference types?**
   - Primitives store values directly, references store addresses to heap objects
   - Primitives have default values (0, false), references default to null
   - Primitives cannot be null, references can be
   - Primitives live on stack, objects live on heap

2. **Why is Java considered pass-by-value?**
   - Java always passes a copy of the value
   - For primitives: copy of the value itself
   - For objects: copy of the reference (address)
   - You can mutate objects, but cannot reassign caller's reference

3. **Why is String immutable?**
   - Security: safe for classloading, network connections
   - Thread safety: can be shared without synchronization
   - Caching: hashCode can be cached
   - String pool: identical strings can share memory

4. **What is the String pool?**
   - Memory area storing unique String literals
   - Saves memory through interning
   - `"Hello" == "Hello"` is true (same pooled reference)
   - `new String("Hello")` creates new object outside pool

5. **Explain the equals() and hashCode() contract.**
   - If `a.equals(b)`, then `a.hashCode() == b.hashCode()` (must)
   - If `a.hashCode() != b.hashCode()`, then `!a.equals(b)` (must)
   - Same hashCode doesn't guarantee equals (collisions allowed)
   - Both must use same fields for consistency

### Intermediate Questions

6. **What is the difference between Comparable and Comparator?**
   - Comparable: Natural ordering, implemented IN the class, one per class
   - Comparator: External ordering, separate class, multiple per class
   - Comparable: `compareTo(T other)`, Comparator: `compare(T o1, T o2)`

7. **Explain static keyword with examples.**
   - Static fields: shared across all instances (class-level)
   - Static methods: called on class, no `this` reference
   - Static blocks: run once when class is loaded
   - Static nested classes: no reference to outer instance

8. **What is the initialization order of a class?**
   - 1) Static fields and static blocks (class loading)
   - 2) Instance fields and instance blocks (object creation)
   - 3) Constructor body

9. **What is autoboxing and unboxing?**
   - Autoboxing: primitive → wrapper (`int` → `Integer`)
   - Unboxing: wrapper → primitive (`Integer` → `int`)
   - Pitfall: Unboxing null throws NullPointerException
   - Integer cache: -128 to 127 cached (== may work unexpectedly)

10. **What are the different types of nested classes?**
    - Static nested: no outer reference, like regular class
    - Inner (non-static): has outer reference via `Outer.this`
    - Local: defined in method, captures effectively final locals
    - Anonymous: inline class without name

### Advanced Questions

11. **What are Records and when would you use them?**
    - Immutable data carriers (Java 14+)
    - Auto-generates constructor, accessors, equals, hashCode, toString
    - Use for DTOs, value objects, simple data holders
    - Cannot extend other classes, implicitly final

12. **What are Sealed classes and why use them?**
    - Restrict which classes can extend (Java 17+)
    - Permits clause lists allowed subclasses
    - Enables exhaustive pattern matching
    - Better domain modeling and API control

13. **Explain Pattern Matching in Java.**
    - instanceof with variable: `if (obj instanceof String s)`
    - Switch patterns: type patterns, guarded patterns
    - Record patterns: deconstruct records in switch/instanceof
    - Enables cleaner, more type-safe code

14. **What are best practices for Optional?**
    - Use for return types to indicate absence
    - Don't use for fields or parameters
    - Prefer orElse/orElseGet over get()
    - Don't overuse - null is still valid for private fields

15. **Why prefer java.time over java.util.Date?**
    - Immutable and thread-safe
    - Clear API (LocalDate, LocalTime, ZonedDateTime)
    - Better timezone handling
    - Cleaner formatting and parsing

---

## Quick Reference

### Primitive Types

| Type | Size | Range | Default | Wrapper |
|------|------|-------|---------|---------|
| byte | 8 bits | -128 to 127 | 0 | Byte |
| short | 16 bits | -32,768 to 32,767 | 0 | Short |
| int | 32 bits | -2³¹ to 2³¹-1 | 0 | Integer |
| long | 64 bits | -2⁶³ to 2⁶³-1 | 0L | Long |
| float | 32 bits | ±3.4E38 | 0.0f | Float |
| double | 64 bits | ±1.7E308 | 0.0d | Double |
| char | 16 bits | 0 to 65,535 | '\u0000' | Character |
| boolean | 1 bit | true/false | false | Boolean |

### Access Modifiers

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| public | ✓ | ✓ | ✓ | ✓ |
| protected | ✓ | ✓ | ✓ | ✗ |
| (default) | ✓ | ✓ | ✗ | ✗ |
| private | ✓ | ✗ | ✗ | ✗ |

### String Methods Cheat Sheet

```java
// Query
s.length()              s.isEmpty()             s.isBlank()
s.charAt(i)             s.indexOf(str)          s.lastIndexOf(str)
s.contains(str)         s.startsWith(str)       s.endsWith(str)

// Transform
s.substring(start, end) s.toLowerCase()         s.toUpperCase()
s.trim()                s.strip()               s.replace(old, new)
s.split(regex)          s.repeat(n)

// Compare
s.equals(other)         s.equalsIgnoreCase(other)
s.compareTo(other)      s.compareToIgnoreCase(other)

// Convert
String.valueOf(x)       s.toCharArray()
Integer.parseInt(s)     s.getBytes()
```
