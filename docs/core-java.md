# Core Java Fundamentals

This chapter covers fundamental Java concepts that form the foundation of the language. Understanding these concepts is essential for any Java developer and is frequently tested in interviews.

---

## 1) Java Compilation Model

### Definition

Java uses a two-stage compilation model: source code is first compiled to platform-independent **bytecode**, which is then executed by the **Java Virtual Machine (JVM)** on the target platform.

### Illustration

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Java Compilation and Execution                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Source Code        Compile         Bytecode          Execute          │
│                                                                         │
│   ┌─────────┐       ┌───────┐       ┌─────────┐       ┌─────────┐      │
│   │ .java   │ ────► │ javac │ ────► │ .class  │ ────► │   JVM   │      │
│   │  file   │       │       │       │  file   │       │         │      │
│   └─────────┘       └───────┘       └─────────┘       └────┬────┘      │
│                                                             │          │
│                        Platform-Independent                 │          │
│                              ▼                              │          │
│                                                             ▼          │
│                         ┌───────────────────────────────────────┐      │
│                         │    Runs on ANY platform with JVM      │      │
│                         │  Windows | macOS | Linux | Android    │      │
│                         └───────────────────────────────────────┘      │
│                                                                         │
│   JVM Execution Modes:                                                  │
│   ┌──────────────────────────────────────────────────────────────┐     │
│   │  Interpreter: Executes bytecode line-by-line (slow start)    │     │
│   │  JIT Compiler: Compiles hot code to native (fast execution)  │     │
│   └──────────────────────────────────────────────────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Points

| Aspect | Description |
|--------|-------------|
| **Bytecode** | Intermediate representation, not machine code |
| **JVM** | Abstract machine that executes bytecode |
| **JIT (Just-In-Time)** | Compiles frequently executed bytecode to native code at runtime |
| **Platform Independence** | "Write once, run anywhere" - same bytecode runs on any JVM |

### Code Example

```java
// HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

// Compile: javac HelloWorld.java → produces HelloWorld.class
// Run: java HelloWorld → JVM executes bytecode
```

---

## 2) Primitives vs References

### Definition

Java has two categories of data types:
- **Primitives**: Store actual values directly in memory
- **References**: Store addresses (references) to objects in heap memory

### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Primitives vs References                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   PRIMITIVE (Stack)                 REFERENCE (Stack → Heap)            │
│                                                                         │
│   ┌─────────────┐                   ┌─────────────┐                     │
│   │  int x = 5  │                   │   name      │──────┐              │
│   │  ┌───────┐  │                   │  ┌───────┐  │      │              │
│   │  │   5   │  │                   │  │ 0x123 │  │      │              │
│   │  └───────┘  │                   │  └───────┘  │      │              │
│   │   (value)   │                   │  (address)  │      │              │
│   └─────────────┘                   └─────────────┘      │              │
│                                                          │              │
│                                                          ▼              │
│                                             ┌─────────────────────┐     │
│                                             │  Heap Memory        │     │
│                                             │  ┌───────────────┐  │     │
│                                             │  │ String Object │  │     │
│                                             │  │   "Alice"     │  │     │
│                                             │  │   length: 5   │  │     │
│                                             │  └───────────────┘  │     │
│                                             └─────────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Primitive Types

| Type | Size | Range | Default | Wrapper |
|------|------|-------|---------|---------|
| `byte` | 8 bits | -128 to 127 | 0 | `Byte` |
| `short` | 16 bits | -32,768 to 32,767 | 0 | `Short` |
| `int` | 32 bits | -2³¹ to 2³¹-1 | 0 | `Integer` |
| `long` | 64 bits | -2⁶³ to 2⁶³-1 | 0L | `Long` |
| `float` | 32 bits | ±3.4E38 | 0.0f | `Float` |
| `double` | 64 bits | ±1.7E308 | 0.0d | `Double` |
| `char` | 16 bits | '\u0000' to '\uffff' | '\u0000' | `Character` |
| `boolean` | 1 bit* | true/false | false | `Boolean` |

*JVM implementation-dependent

### Reference Types

```java
// Reference types include:
String name = "Alice";          // String
int[] numbers = {1, 2, 3};      // Arrays
List<String> list = new ArrayList<>();  // Collections
User user = new User();         // Custom objects

// References can be null
String nullRef = null;          // Valid - no object referenced
int primitive = null;           // ❌ Compile error - primitives can't be null
```

### Key Differences

| Aspect | Primitive | Reference |
|--------|-----------|-----------|
| Stores | Actual value | Address to object |
| Can be null? | No | Yes |
| Memory location | Stack | Stack (reference) + Heap (object) |
| Default value | Type-specific (0, false) | null |
| Pass to method | Copy of value | Copy of reference |
| Comparison `==` | Compares values | Compares addresses |

---

## 3) == vs equals()

### Definition

- `==` compares **primitive values** or **reference identity** (memory addresses)
- `equals()` compares **logical equality** (content) if properly overridden

### Visual Illustration

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        == vs equals()                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   String s1 = new String("hello");                                      │
│   String s2 = new String("hello");                                      │
│   String s3 = s1;                                                       │
│                                                                         │
│   Stack                              Heap                               │
│   ┌─────┐                           ┌──────────────┐                    │
│   │ s1  │───────────────────────────│► "hello"     │ Object A          │
│   │0x100│                           │  (0x100)     │                    │
│   ├─────┤                           └──────────────┘                    │
│   │ s2  │───────────────────────────│► "hello"     │ Object B          │
│   │0x200│                           │  (0x200)     │                    │
│   ├─────┤                           └──────────────┘                    │
│   │ s3  │───────────────────────────┘                                   │
│   │0x100│  (same as s1)                                                 │
│   └─────┘                                                               │
│                                                                         │
│   s1 == s2    → false (different addresses: 0x100 vs 0x200)             │
│   s1 == s3    → true  (same address: 0x100)                             │
│   s1.equals(s2) → true  (same content: "hello")                         │
│   s1.equals(s3) → true  (same content: "hello")                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Code Examples

```java
// Primitives: == compares values
int a = 5;
int b = 5;
System.out.println(a == b);  // true - same value

// References: == compares addresses
String str1 = new String("hello");
String str2 = new String("hello");
System.out.println(str1 == str2);       // false - different objects
System.out.println(str1.equals(str2));  // true - same content

// String literals: Use String pool
String lit1 = "hello";
String lit2 = "hello";
System.out.println(lit1 == lit2);       // true - same pooled reference
System.out.println(lit1.equals(lit2));  // true - same content

// Integer caching (-128 to 127)
Integer i1 = 100;
Integer i2 = 100;
System.out.println(i1 == i2);  // true - cached

Integer i3 = 200;
Integer i4 = 200;
System.out.println(i3 == i4);  // false - not cached
System.out.println(i3.equals(i4));  // true - always use equals for objects!
```

---

## 4) Pass-By-Value

### Definition

**Java is ALWAYS pass-by-value**. When passing arguments to methods:
- For **primitives**: A copy of the value is passed
- For **references**: A copy of the reference (address) is passed, NOT the object itself

### Visual Illustration

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Pass-By-Value                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   PRIMITIVE:                                                            │
│   ┌─────────────────────────────────────────────────────────────┐      │
│   │  int x = 10;                                                 │      │
│   │  modify(x);        // Passes COPY of 10                      │      │
│   │  // x is still 10                                            │      │
│   │                                                              │      │
│   │  void modify(int n) {   // n = copy = 10                     │      │
│   │      n = 20;            // Changes only local copy           │      │
│   │  }                                                           │      │
│   └─────────────────────────────────────────────────────────────┘      │
│                                                                         │
│   REFERENCE:                                                            │
│   ┌─────────────────────────────────────────────────────────────┐      │
│   │  StringBuilder sb = new StringBuilder("hello");              │      │
│   │  modify(sb);       // Passes COPY of reference               │      │
│   │  // sb.toString() → "hellox" (object was mutated)            │      │
│   │                                                              │      │
│   │  void modify(StringBuilder str) {                            │      │
│   │      str.append("x");  // Mutates the SAME object ✓          │      │
│   │      str = new StringBuilder("new");  // Rebinds local copy  │      │
│   │  }                      // Original 'sb' unchanged           │      │
│   └─────────────────────────────────────────────────────────────┘      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Code Examples

```java
// Primitive: Value is copied
public void changePrimitive(int x) {
    x = 100;  // Only changes local copy
}

int num = 10;
changePrimitive(num);
System.out.println(num);  // Still 10

// Reference: Reference is copied (but points to same object)
public void modifyObject(StringBuilder sb) {
    sb.append(" world");  // Modifies the SAME object
    sb = new StringBuilder("new");  // Rebinds local reference only
}

StringBuilder builder = new StringBuilder("hello");
modifyObject(builder);
System.out.println(builder);  // "hello world" (not "new")

// Proof that Java is not pass-by-reference
public void tryToSwap(Integer a, Integer b) {
    Integer temp = a;
    a = b;
    b = temp;  // Only swaps local copies!
}

Integer x = 1, y = 2;
tryToSwap(x, y);
System.out.println(x + ", " + y);  // "1, 2" (not swapped!)
```

---

## 5) String and Immutability

### Definition

**String** in Java is **immutable** - once created, its value cannot be changed. Any operation that appears to modify a String actually creates a new String object.

### Why Strings are Immutable

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Why String is Immutable?                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   1. SECURITY                                                           │
│      - Strings used in class loading, file paths, network connections   │
│      - If mutable, security checks could be bypassed                    │
│                                                                         │
│   2. THREAD SAFETY                                                      │
│      - Immutable objects are inherently thread-safe                     │
│      - Can be shared between threads without synchronization            │
│                                                                         │
│   3. CACHING (String Pool)                                              │
│      - JVM can cache and reuse String literals                          │
│      - Saves memory when same string used multiple times                │
│                                                                         │
│   4. HASHCODE CACHING                                                   │
│      - String caches its hashCode (computed once)                       │
│      - Essential for HashMap/HashSet performance                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### String Pool

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        String Pool (String Intern Pool)                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   String Literals are interned:                                         │
│                                                                         │
│   String s1 = "hello";    // Goes to pool                               │
│   String s2 = "hello";    // Reuses from pool                           │
│   String s3 = new String("hello");  // New object on heap               │
│   String s4 = s3.intern();          // Gets pooled version              │
│                                                                         │
│   ┌────────────────────────────────────────────────────────────┐       │
│   │                     HEAP MEMORY                             │       │
│   │  ┌─────────────────────────────────────────────────────┐   │       │
│   │  │   STRING POOL (special area in heap since Java 7)   │   │       │
│   │  │   ┌─────────────────────────────────────────────┐   │   │       │
│   │  │   │  "hello" ← s1, s2, s4 (all point here)      │   │   │       │
│   │  │   │  "world"                                     │   │   │       │
│   │  │   │  "java"                                      │   │   │       │
│   │  │   └─────────────────────────────────────────────┘   │   │       │
│   │  └─────────────────────────────────────────────────────┘   │       │
│   │                                                            │       │
│   │  Regular Heap:                                             │       │
│   │  ┌─────────────────────────────────────────────────────┐   │       │
│   │  │  String object "hello" ← s3 (separate object)       │   │       │
│   │  └─────────────────────────────────────────────────────┘   │       │
│   └────────────────────────────────────────────────────────────┘       │
│                                                                         │
│   s1 == s2   → true  (same pooled object)                              │
│   s1 == s3   → false (different objects)                               │
│   s1 == s4   → true  (intern() returns pooled object)                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### String Operations

```java
// All operations create new String objects
String original = "hello";
String upper = original.toUpperCase();  // New String "HELLO"
System.out.println(original);  // Still "hello"

// Concatenation creates new objects
String s = "a";
s = s + "b";  // Creates new String "ab", original "a" is eligible for GC

// Use StringBuilder for repeated modifications
StringBuilder sb = new StringBuilder("hello");
sb.append(" world");  // Modifies same object
sb.insert(0, "say ");
String result = sb.toString();  // "say hello world"

// String methods (all return new String)
String str = "  Hello World  ";
str.trim();           // "Hello World" (new)
str.substring(2, 7);  // "Hello" (new)
str.replace('o', '0'); // "Hell0 W0rld" (new)
str.toLowerCase();    // "  hello world  " (new)
```

---

## 6) equals() and hashCode() Contract

### Definition

The `equals()` and `hashCode()` methods have a strict contract that must be followed for objects to work correctly in hash-based collections (HashMap, HashSet, etc.).

### The Contract

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   equals() and hashCode() Contract                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   RULE 1: Consistency with equals()                                     │
│   If a.equals(b) is TRUE, then a.hashCode() MUST equal b.hashCode()     │
│                                                                         │
│   RULE 2: hashCode doesn't imply equals                                 │
│   If a.hashCode() == b.hashCode(), a.equals(b) may be TRUE or FALSE     │
│   (Hash collisions are allowed)                                         │
│                                                                         │
│   RULE 3: Consistency                                                   │
│   Multiple calls to hashCode() on unchanged object return same value    │
│   Multiple calls to equals() with unchanged objects return same result  │
│                                                                         │
│   RULE 4: equals() properties                                           │
│   - Reflexive:  a.equals(a) is TRUE                                     │
│   - Symmetric:  a.equals(b) ↔ b.equals(a)                               │
│   - Transitive: a.equals(b) && b.equals(c) → a.equals(c)                │
│   - null-safe:  a.equals(null) is FALSE                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Why This Matters

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   How HashMap Uses These Methods                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   put(key, value):                                                      │
│   1. Calculate bucket: hashCode() → bucket index                        │
│   2. In bucket, use equals() to find existing key                       │
│                                                                         │
│   get(key):                                                             │
│   1. Calculate bucket: hashCode() → bucket index                        │
│   2. In bucket, use equals() to find matching key                       │
│                                                                         │
│   IF CONTRACT IS BROKEN:                                                │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  Scenario: equals() overridden, hashCode() NOT overridden       │  │
│   │                                                                 │  │
│   │  Person p1 = new Person("Alice", 30);                           │  │
│   │  Person p2 = new Person("Alice", 30);  // Same logical person   │  │
│   │                                                                 │  │
│   │  map.put(p1, "value");                                          │  │
│   │  map.get(p2);  → NULL! (different hashCode → different bucket)  │  │
│   │                                                                 │  │
│   │  Even though p1.equals(p2) is TRUE!                             │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Proper Implementation

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
        // 1. Check for same reference
        if (this == o) return true;
        
        // 2. Check for null and type
        if (o == null || getClass() != o.getClass()) return false;
        
        // 3. Cast and compare fields
        Person person = (Person) o;
        return age == person.age &&
               Objects.equals(name, person.name) &&
               Objects.equals(email, person.email);
    }
    
    @Override
    public int hashCode() {
        // Use same fields as equals()
        return Objects.hash(name, age, email);
    }
}

// Usage
Person p1 = new Person("Alice", 30, "alice@email.com");
Person p2 = new Person("Alice", 30, "alice@email.com");

System.out.println(p1.equals(p2));  // true
System.out.println(p1.hashCode() == p2.hashCode());  // true

Map<Person, String> map = new HashMap<>();
map.put(p1, "Employee");
System.out.println(map.get(p2));  // "Employee" ✓
```

### Common Pitfalls

```java
// ❌ BAD: Using mutable fields in hashCode
public class BadKey {
    private String name;  // Mutable!
    
    @Override
    public int hashCode() {
        return name.hashCode();
    }
}

Set<BadKey> set = new HashSet<>();
BadKey key = new BadKey("original");
set.add(key);
key.setName("changed");  // hashCode changes!
set.contains(key);  // FALSE! Key is "lost" in wrong bucket

// ✓ GOOD: Use immutable fields or immutable objects
public class GoodKey {
    private final String name;  // Immutable!
    
    @Override
    public int hashCode() {
        return name.hashCode();
    }
}
```

---

## 7) Comparable vs Comparator

### Definition

Both interfaces define ordering for objects, but serve different purposes:
- **Comparable<T>**: Defines natural ordering (built into the class)
- **Comparator<T>**: Defines external ordering (separate from the class)

### Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Comparable vs Comparator                              │
├────────────────────────┬────────────────────────────────────────────────┤
│      Comparable        │              Comparator                         │
├────────────────────────┼────────────────────────────────────────────────┤
│ java.lang              │ java.util                                       │
│ compareTo(T o)         │ compare(T o1, T o2)                             │
│ Natural ordering       │ Custom ordering                                 │
│ Single sort order      │ Multiple sort orders possible                   │
│ Modifies original class│ Separate class/lambda                           │
│ "this" vs "other"      │ Two separate objects                            │
└────────────────────────┴────────────────────────────────────────────────┘
```

### Code Examples

```java
// Comparable: Natural ordering (built into class)
public class Employee implements Comparable<Employee> {
    private String name;
    private int salary;
    private LocalDate hireDate;
    
    @Override
    public int compareTo(Employee other) {
        // Natural order by name
        return this.name.compareTo(other.name);
    }
}

// Usage
List<Employee> employees = new ArrayList<>();
Collections.sort(employees);  // Uses natural order (by name)
employees.stream().sorted();  // Same

// Comparator: External ordering (multiple options)
Comparator<Employee> bySalary = Comparator.comparingInt(Employee::getSalary);
Comparator<Employee> bySalaryDesc = bySalary.reversed();
Comparator<Employee> byHireDate = Comparator.comparing(Employee::getHireDate);

// Chained comparisons
Comparator<Employee> byDeptThenSalary = Comparator
    .comparing(Employee::getDepartment)
    .thenComparingInt(Employee::getSalary)
    .reversed();

// Null-safe comparators
Comparator<Employee> nullSafe = Comparator
    .nullsLast(Comparator.comparing(Employee::getName));

// Usage with comparator
Collections.sort(employees, bySalary);
employees.stream().sorted(bySalaryDesc);
employees.sort(byDeptThenSalary);
```

### Return Value Rules

```java
// compareTo() / compare() return value:
// NEGATIVE: this/first < other/second  (comes before)
// ZERO:     this/first == other/second (equal)
// POSITIVE: this/first > other/second  (comes after)

// Example implementations:
@Override
public int compareTo(Employee other) {
    // Compare integers
    return Integer.compare(this.salary, other.salary);
    
    // Or for simple cases:
    // return this.salary - other.salary;  // Avoid! Can overflow
}

// TreeSet/TreeMap: compareTo determines equality!
// If compareTo returns 0, elements are considered duplicates
TreeSet<Employee> set = new TreeSet<>(bySalary);
set.add(new Employee("Alice", 50000));
set.add(new Employee("Bob", 50000));  // Might not be added if same salary!
```

---

## 8) static Keyword

### Definition

The `static` keyword denotes that a member (field, method, block, or nested class) belongs to the class itself rather than to instances of the class.

### Visual Illustration

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Static vs Instance                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   CLASS (shared)                    INSTANCES (separate)                │
│   ┌─────────────────────┐                                               │
│   │  Counter class      │                                               │
│   │  ─────────────────  │                                               │
│   │  static int count=3 │◄────────┬───────┬───────┐                    │
│   │  static void inc()  │         │       │       │                    │
│   └─────────────────────┘         │       │       │                    │
│                                   │       │       │                    │
│                             ┌─────┴─┐ ┌───┴───┐ ┌─┴─────┐              │
│                             │ obj1  │ │ obj2  │ │ obj3  │              │
│                             │ ───── │ │ ───── │ │ ───── │              │
│                             │ id=1  │ │ id=2  │ │ id=3  │              │
│                             └───────┘ └───────┘ └───────┘              │
│                                (each has own 'id')                      │
│                                                                         │
│   Counter.count → 3  (accessed via class)                               │
│   obj1.count    → 3  (same value, bad practice)                         │
│   Counter.inc() → increments shared count                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Types of Static Members

```java
public class StaticExample {
    
    // 1. STATIC FIELD - Shared across all instances
    private static int instanceCount = 0;
    public static final String APP_NAME = "MyApp";  // Constant
    
    // 2. INSTANCE FIELD - Unique per instance
    private int id;
    
    // 3. STATIC BLOCK - Runs once when class is loaded
    static {
        System.out.println("Class loaded!");
        // Initialize static fields, load resources, etc.
    }
    
    // 4. INSTANCE BLOCK - Runs for each new instance
    {
        this.id = ++instanceCount;
        System.out.println("Instance " + id + " created");
    }
    
    // 5. STATIC METHOD - Called on class, no 'this'
    public static int getInstanceCount() {
        return instanceCount;
        // Cannot access 'this' or instance members directly
    }
    
    // 6. INSTANCE METHOD - Called on object, has 'this'
    public int getId() {
        return this.id;  // Can access 'this' and instance members
    }
    
    // 7. STATIC NESTED CLASS - No reference to outer instance
    public static class StaticNested {
        // Can only access outer class's static members
    }
}

// Usage
StaticExample.APP_NAME;        // Access static field via class
StaticExample.getInstanceCount();  // Access static method via class

StaticExample obj = new StaticExample();
obj.getId();                   // Access instance method via object
// obj.APP_NAME;               // Works but bad practice - use class name
```

---

## 9) Initialization Order

### Definition

When a Java class is loaded and instances are created, initialization happens in a specific order that's important to understand.

### Visual Order

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Initialization Order                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   CLASS LOADING (happens once):                                         │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  1. Static fields (default values: 0, null, false)              │  │
│   │  2. Static fields (explicit initializers) + Static blocks       │  │
│   │     → In order of appearance, top to bottom                     │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│   INSTANCE CREATION (each new object):                                  │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  3. Instance fields (default values: 0, null, false)            │  │
│   │  4. Instance fields (explicit initializers) + Instance blocks   │  │
│   │     → In order of appearance, top to bottom                     │  │
│   │  5. Constructor body                                            │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│   WITH INHERITANCE:                                                     │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │  1. Parent static init                                          │  │
│   │  2. Child static init                                           │  │
│   │  3. Parent instance init + constructor                          │  │
│   │  4. Child instance init + constructor                           │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Code Example

```java
public class InitOrder {
    // Step 1 (class loading): static field default = 0
    // Step 2 (class loading): static field = 10
    private static int staticField = printAndReturn("Static field init", 10);
    
    // Step 2 (class loading): static block
    static {
        System.out.println("Static block: staticField = " + staticField);
    }
    
    // Step 3 (instance creation): instance field default = 0
    // Step 4 (instance creation): instance field = 20
    private int instanceField = printAndReturn("Instance field init", 20);
    
    // Step 4 (instance creation): instance block
    {
        System.out.println("Instance block: instanceField = " + instanceField);
    }
    
    // Step 5 (instance creation): constructor
    public InitOrder() {
        System.out.println("Constructor");
    }
    
    private static int printAndReturn(String msg, int value) {
        System.out.println(msg + ": " + value);
        return value;
    }
    
    public static void main(String[] args) {
        System.out.println("Creating first instance:");
        new InitOrder();
        System.out.println("\nCreating second instance:");
        new InitOrder();
    }
}

/* Output:
Static field init: 10
Static block: staticField = 10
Creating first instance:
Instance field init: 20
Instance block: instanceField = 20
Constructor

Creating second instance:
Instance field init: 20
Instance block: instanceField = 20
Constructor
*/
```

---

## 10) final Keyword

### Definition

The `final` keyword can be applied to variables, methods, and classes with different meanings:
- **Final variable**: Cannot be reassigned (constant)
- **Final method**: Cannot be overridden
- **Final class**: Cannot be subclassed

### Behavior Summary

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         final Keyword Usage                              │
├────────────────────┬────────────────────────────────────────────────────┤
│      Context       │                    Effect                          │
├────────────────────┼────────────────────────────────────────────────────┤
│ final variable     │ Value cannot be changed after initialization       │
│ (primitive)        │ int x = 5; x = 10; // ❌ Compile error            │
├────────────────────┼────────────────────────────────────────────────────┤
│ final variable     │ Reference cannot be changed, but object can be     │
│ (reference)        │ mutated: final List list = new ArrayList();        │
│                    │ list.add("x"); // ✓   list = new List(); // ❌    │
├────────────────────┼────────────────────────────────────────────────────┤
│ final method       │ Cannot be overridden by subclasses                 │
│                    │ Allows JVM to inline the method (performance)      │
├────────────────────┼────────────────────────────────────────────────────┤
│ final class        │ Cannot be extended (no subclasses allowed)         │
│                    │ Examples: String, Integer, System                  │
├────────────────────┼────────────────────────────────────────────────────┤
│ final parameter    │ Parameter cannot be reassigned within method       │
│                    │ void process(final int x) { x = 10; // ❌ }        │
└────────────────────┴────────────────────────────────────────────────────┘
```

### Code Examples

```java
// Final variables
public class FinalExamples {
    // Final static = compile-time constant
    public static final double PI = 3.14159;
    
    // Final instance field - must be initialized by constructor
    private final String name;
    
    public FinalExamples(String name) {
        this.name = name;  // Can only assign once
    }
    
    public void demonstrate() {
        // Final local variable
        final int x = 10;
        // x = 20;  // ❌ Compile error
        
        // Final reference - object can be mutated!
        final List<String> list = new ArrayList<>();
        list.add("item");      // ✓ Mutating the object
        // list = new ArrayList<>();  // ❌ Reassigning reference
        
        // Effectively final (for lambdas)
        int y = 20;  // Not declared final, but never reassigned
        Runnable r = () -> System.out.println(y);  // ✓ OK
        // y = 30;  // Would break the lambda above
    }
}

// Final method
class Parent {
    public final void cannotOverride() {
        System.out.println("Parent implementation");
    }
}

class Child extends Parent {
    // @Override
    // public void cannotOverride() { }  // ❌ Compile error
}

// Final class
final class ImmutableClass {
    private final String value;
    
    public ImmutableClass(String value) {
        this.value = value;
    }
}

// class ExtendImmutable extends ImmutableClass { }  // ❌ Compile error
```

### final in Concurrency

```java
// Final fields have special JMM guarantees
public class SafePublication {
    private final Map<String, String> config;
    
    public SafePublication() {
        config = loadConfig();  // Safely published to other threads
    }
    
    public String get(String key) {
        return config.get(key);  // Safe without synchronization
    }
}
```

---

## 11) Wrapper Classes and Autoboxing

### Definition

**Wrapper Classes** are object representations of primitive types. **Autoboxing** is automatic conversion between primitives and their wrappers.

### Wrapper Class Mapping

| Primitive | Wrapper | Cache Range |
|-----------|---------|-------------|
| `byte` | `Byte` | -128 to 127 (all) |
| `short` | `Short` | -128 to 127 |
| `int` | `Integer` | -128 to 127 |
| `long` | `Long` | -128 to 127 |
| `float` | `Float` | None |
| `double` | `Double` | None |
| `char` | `Character` | 0 to 127 |
| `boolean` | `Boolean` | TRUE, FALSE |

### Autoboxing and Unboxing

```java
// Autoboxing: primitive → wrapper (automatic)
Integer boxed = 100;  // Compiler: Integer.valueOf(100)
List<Integer> list = new ArrayList<>();
list.add(5);          // Compiler: list.add(Integer.valueOf(5))

// Unboxing: wrapper → primitive (automatic)
int unboxed = boxed;  // Compiler: boxed.intValue()
int sum = boxed + 10; // Compiler: boxed.intValue() + 10

// ⚠️ PITFALL: NullPointerException on unboxing null!
Integer nullInt = null;
int value = nullInt;  // NullPointerException!

// Integer Cache: Integers from -128 to 127 are cached
Integer a = 100;
Integer b = 100;
System.out.println(a == b);  // true (same cached object)

Integer c = 200;
Integer d = 200;
System.out.println(c == d);  // false (different objects)
System.out.println(c.equals(d));  // true (always use equals!)
```

### Visual: Integer Cache

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Integer Cache (-128 to 127)                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Integer.valueOf(100):                                                 │
│   ┌───────────────────────────────────────────────────────────────┐    │
│   │  if (value >= -128 && value <= 127)                            │    │
│   │      return cache[value + 128];  // Return cached object       │    │
│   │  else                                                          │    │
│   │      return new Integer(value);  // Create new object          │    │
│   └───────────────────────────────────────────────────────────────┘    │
│                                                                         │
│   Integer a = 100; Integer b = 100;                                     │
│   a ──┐                                                                 │
│       └──► [Cached Integer(100)]   a == b → true                       │
│   b ──┘                                                                 │
│                                                                         │
│   Integer c = 200; Integer d = 200;                                     │
│   c ──────► [Integer(200)]         c == d → false                      │
│   d ──────► [Integer(200)]         (different objects)                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 12) Enums

### Definition

**Enums** are special classes that represent a fixed set of constants. They provide type safety, can have fields and methods, and are the best way to implement singletons.

### Basic Enum

```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// Usage
Day today = Day.MONDAY;
System.out.println(today);          // "MONDAY"
System.out.println(today.ordinal()); // 0 (position)
System.out.println(today.name());    // "MONDAY"

// Iterating
for (Day day : Day.values()) {
    System.out.println(day);
}

// Parsing
Day parsed = Day.valueOf("MONDAY");  // From string

// Switch statement
switch (today) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> System.out.println("Weekday");
    case SATURDAY, SUNDAY -> System.out.println("Weekend");
}
```

### Enum with Fields and Methods

```java
public enum Status {
    PENDING("P", 1),
    ACTIVE("A", 2),
    SUSPENDED("S", 3),
    CLOSED("C", 4);
    
    private final String code;
    private final int priority;
    
    // Constructor (always private)
    Status(String code, int priority) {
        this.code = code;
        this.priority = priority;
    }
    
    public String getCode() {
        return code;
    }
    
    public int getPriority() {
        return priority;
    }
    
    // Static factory method
    public static Status fromCode(String code) {
        for (Status s : values()) {
            if (s.code.equals(code)) {
                return s;
            }
        }
        throw new IllegalArgumentException("Unknown code: " + code);
    }
    
    // Instance method
    public boolean isTerminal() {
        return this == CLOSED;
    }
}

// Usage
Status status = Status.ACTIVE;
System.out.println(status.getCode());     // "A"
System.out.println(status.getPriority()); // 2
System.out.println(status.isTerminal());  // false

Status fromCode = Status.fromCode("P");   // PENDING
```

### Enum Singleton Pattern

```java
// Best way to implement singleton in Java
public enum DatabaseConnection {
    INSTANCE;
    
    private final Connection connection;
    
    DatabaseConnection() {
        // Initialize expensive resource once
        this.connection = createConnection();
    }
    
    public Connection getConnection() {
        return connection;
    }
    
    private Connection createConnection() {
        // Create actual connection
        return null;
    }
}

// Usage
Connection conn = DatabaseConnection.INSTANCE.getConnection();
```

---

## 13) Records (Java 14+)

### Definition

**Records** are special classes that act as transparent carriers for immutable data. They automatically provide constructors, getters, `equals()`, `hashCode()`, and `toString()`.

### Traditional Class vs Record

```java
// Traditional class (verbose)
public final class PersonOld {
    private final String name;
    private final int age;
    
    public PersonOld(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    @Override
    public boolean equals(Object o) { /* implementation */ }
    
    @Override
    public int hashCode() { /* implementation */ }
    
    @Override
    public String toString() { /* implementation */ }
}

// Record (concise) - generates all of the above!
public record Person(String name, int age) { }

// Usage
Person person = new Person("Alice", 30);
System.out.println(person.name());      // "Alice" (accessor, not getName!)
System.out.println(person.age());       // 30
System.out.println(person);             // Person[name=Alice, age=30]
```

### Record with Validation and Additional Methods

```java
public record Person(String name, int age) {
    
    // Compact constructor for validation
    public Person {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
        // Fields are automatically assigned after this block
    }
    
    // Static factory method
    public static Person of(String name, int age) {
        return new Person(name, age);
    }
    
    // Additional instance methods
    public boolean isAdult() {
        return age >= 18;
    }
    
    public String greeting() {
        return "Hello, " + name + "!";
    }
}
```

---

## 14) Pattern Matching

### instanceof Pattern Matching (Java 16+)

```java
// Old way
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// New way - pattern variable
if (obj instanceof String s) {
    System.out.println(s.length());  // s is already cast!
}

// With negation
if (!(obj instanceof String s)) {
    return;
}
// s is in scope here
System.out.println(s.length());

// In expressions
String result = (obj instanceof String s) ? s.toUpperCase() : "not a string";
```

### Switch Pattern Matching (Java 21+)

```java
// Pattern matching in switch
String describe(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + i;
        case Long l    -> "Long: " + l;
        case Double d  -> "Double: " + d;
        case String s  -> "String: " + s;
        case null      -> "null";
        default        -> "Unknown: " + obj.getClass();
    };
}

// With guards
String categorize(Integer num) {
    return switch (num) {
        case null -> "null";
        case Integer i when i < 0 -> "negative";
        case Integer i when i == 0 -> "zero";
        case Integer i -> "positive: " + i;
    };
}

// With records
record Point(int x, int y) {}

String describePoint(Object obj) {
    return switch (obj) {
        case Point(int x, int y) when x == 0 && y == 0 -> "origin";
        case Point(int x, int y) when x == 0 -> "on y-axis";
        case Point(int x, int y) when y == 0 -> "on x-axis";
        case Point(int x, int y) -> "point at (" + x + ", " + y + ")";
        default -> "not a point";
    };
}
```

---

## 15) Nested Classes

### Types of Nested Classes

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       Nested Classes in Java                             │
├────────────────────────┬────────────────────────────────────────────────┤
│         Type           │                Characteristics                  │
├────────────────────────┼────────────────────────────────────────────────┤
│ Static Nested Class    │ • Declared with 'static' keyword               │
│                        │ • No implicit reference to outer instance       │
│                        │ • Can only access outer's static members        │
├────────────────────────┼────────────────────────────────────────────────┤
│ Inner Class            │ • Non-static member class                       │
│ (Member Class)         │ • Has implicit reference to outer (Outer.this) │
│                        │ • Can access all outer members                  │
│                        │ • ⚠️ Can cause memory leaks                     │
├────────────────────────┼────────────────────────────────────────────────┤
│ Local Class            │ • Defined inside a method                       │
│                        │ • Can access effectively final local variables  │
│                        │ • Scope limited to the method                   │
├────────────────────────┼────────────────────────────────────────────────┤
│ Anonymous Class        │ • No name, declared and instantiated together   │
│                        │ • Used for one-time implementations             │
│                        │ • Can capture effectively final variables       │
│                        │ • Largely replaced by lambdas                   │
└────────────────────────┴────────────────────────────────────────────────┘
```

### Code Examples

```java
public class Outer {
    private int outerField = 10;
    private static int staticField = 20;
    
    // 1. STATIC NESTED CLASS
    public static class StaticNested {
        public void display() {
            // Can access static members only
            System.out.println(staticField);  // ✓
            // System.out.println(outerField);  // ❌ Cannot access
        }
    }
    
    // 2. INNER CLASS (non-static member class)
    public class Inner {
        public void display() {
            // Can access all outer members
            System.out.println(outerField);   // ✓
            System.out.println(staticField);  // ✓
            System.out.println(Outer.this.outerField);  // Explicit outer ref
        }
    }
    
    public void method() {
        final int localVar = 30;  // Must be effectively final
        
        // 3. LOCAL CLASS
        class LocalClass {
            public void display() {
                System.out.println(outerField);  // ✓
                System.out.println(localVar);    // ✓ (effectively final)
            }
        }
        
        // 4. ANONYMOUS CLASS
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(outerField);  // ✓
                System.out.println(localVar);    // ✓
            }
        };
    }
}

// Usage
Outer outer = new Outer();
Outer.StaticNested staticNested = new Outer.StaticNested();  // No outer instance
Outer.Inner inner = outer.new Inner();  // Requires outer instance
```

---

## 16) Common Interview Questions

### Primitives and References

**Q1: What is the difference between primitives and reference types?**

**Answer:** Primitives store actual values directly, while references store addresses to objects on the heap. Primitives cannot be null, have fixed sizes, and are compared by value with `==`. References can be null, point to heap objects, and `==` compares addresses while `equals()` compares content.

**Q2: Is Java pass-by-value or pass-by-reference?**

**Answer:** Java is always pass-by-value. For primitives, a copy of the value is passed. For objects, a copy of the reference (address) is passed. You can mutate the object through the copied reference, but you cannot change what the original reference points to.

### String

**Q3: Why is String immutable?**

**Answer:** String is immutable for:
1. **Security**: Strings used in class loading, network connections
2. **Thread safety**: Immutable objects are inherently thread-safe
3. **String pool**: Enables caching and sharing of string literals
4. **HashCode caching**: HashCode computed once, reused

**Q4: What is the String pool?**

**Answer:** The String pool is a special memory area where Java stores string literals. When you create a string literal, JVM checks if an identical string exists in the pool. If yes, it returns the existing reference; if no, it creates a new string in the pool.

### equals/hashCode

**Q5: What is the equals/hashCode contract?**

**Answer:** If `a.equals(b)` returns true, then `a.hashCode()` must equal `b.hashCode()`. The reverse is not required (hash collisions allowed). Breaking this contract causes objects to be "lost" in hash-based collections.

**Q6: Why must you override hashCode when overriding equals?**

**Answer:** HashMap/HashSet use `hashCode()` to determine the bucket and `equals()` to find the key within the bucket. If hashCode isn't overridden, logically equal objects may end up in different buckets and won't be found.

### Other Concepts

**Q7: What is the difference between `==` and `equals()`?**

**Answer:** `==` compares primitive values or reference addresses (memory locations). `equals()` compares logical equality based on the object's state (if properly overridden).

**Q8: What is autoboxing/unboxing?**

**Answer:** Autoboxing is automatic conversion from primitive to wrapper (`int` → `Integer`). Unboxing is the reverse. Pitfall: Unboxing null causes NPE. Performance: Use primitives in performance-critical code.

**Q9: What is the difference between `Comparable` and `Comparator`?**

**Answer:** `Comparable` defines natural ordering within the class (`compareTo`). `Comparator` defines external ordering separate from the class (`compare`). Use Comparable for single natural order, Comparator for multiple orderings.

**Q10: Explain the initialization order in Java.**

**Answer:** 
1. Static fields/blocks (parent first, then child, once per class)
2. Instance fields/blocks (parent first, then child, per instance)
3. Constructor (parent first, then child)

---

## 17) Quick Reference

### Primitives

```java
byte b = 127;         // 8 bits
short s = 32000;      // 16 bits
int i = 2000000000;   // 32 bits
long l = 9000000000L; // 64 bits (L suffix)
float f = 3.14f;      // 32 bits (f suffix)
double d = 3.14159;   // 64 bits
char c = 'A';         // 16 bits Unicode
boolean bool = true;  // true/false
```

### String Methods

```java
str.length();              // Length
str.charAt(i);             // Character at index
str.substring(start, end); // Substring
str.indexOf(s);            // First index
str.contains(s);           // Contains check
str.equals(s);             // Content equality
str.equalsIgnoreCase(s);   // Case-insensitive
str.toUpperCase();         // Upper case
str.toLowerCase();         // Lower case
str.trim();                // Remove whitespace
str.replace(old, new);     // Replace all
str.split(regex);          // Split to array
String.format(fmt, args);  // Format string
```

### Collections with Comparator

```java
// Sort with comparator
list.sort(Comparator.comparing(Person::getName));

// Chained comparators
Comparator.comparing(Person::getAge)
    .thenComparing(Person::getName)
    .reversed();

// Null-safe
Comparator.nullsLast(Comparator.naturalOrder());
```

---

*Master these fundamentals for a strong Java foundation!*
