# 1️⃣ Java Fundamentals (Must-Master)

This chapter builds the base for everything else. Most interview failures happen not because a candidate doesn’t know “advanced topics”, but because fundamentals (types, equality, object model, OOP basics) are shaky.

---

## 1.1 Java Basics

### History & evolution of Java (what interviewers expect)

- **1995**: Java launched by Sun Microsystems with the goal of “write once, run anywhere”.
- Major milestones (high ROI):
  - **Java 5**: Generics, annotations, enhanced for-loop, autoboxing.
  - **Java 8**: Lambdas, Streams, default methods, new Date/Time (`java.time`).
  - **Java 9**: Modules (JPMS).
  - **Java 10+**: `var` (local inference), records, sealed classes (modern modeling), pattern matching improvements.

Interview framing: newer Java adds language ergonomics and safer modeling, but the core runtime concepts (JVM, GC, memory visibility, collections) remain essential.

### Features of Java (WORA, platform independence)

- **Platform independent**: Java compiles to **bytecode** (`.class`) which runs on any JVM.
- **WORA**: same bytecode can run across OS/CPU architectures (given a compatible JVM).
- **Object-oriented** (mostly), strongly typed, garbage collected.
- **Rich standard library**: collections, concurrency, IO, networking, crypto, etc.

### JDK vs JRE vs JVM

- **JVM (Java Virtual Machine)**: executes bytecode; provides runtime services (GC, JIT, threads).
- **JRE (Java Runtime Environment)**: JVM + core libraries needed to *run* Java apps.
- **JDK (Java Development Kit)**: JRE + compiler (`javac`) + dev tools (e.g., `javadoc`, `jstack`, `jmap`, etc.).

Illustration:

```
Your .java code
   |  (javac)           (runs on OS)
   v
.class bytecode  --->  JVM  +  standard libs  ---> executing program
```

### Compilation & execution flow (important)

1. `javac` compiles `.java` → `.class` bytecode
2. ClassLoader loads classes into JVM
3. Bytecode verified (safety checks)
4. Execution engine runs bytecode
5. Hot code paths may be **JIT-compiled** to native machine code

Interview keywords: **class loading**, **bytecode verification**, **JIT**, **GC**, **runtime data areas**.

### Java memory model (high-level overview)

At a high level (not the deep JMM yet):

- **Heap**: objects live here (garbage collected)
- **Stack**: per-thread frames (locals, return addresses, references)
- **Method area / metaspace**: class metadata

High-level mental model:

```
Thread stack (per thread)          Heap (shared)
--------------------------        ----------------
frame: local ref -->  -------->   object fields...
frame: primitives               (GC manages)
```

Deep JMM (happens-before/volatile) is covered in Concurrency.

### Keywords, identifiers, literals

- **Keywords**: reserved words like `class`, `public`, `static`, `final`, `try`, `catch`, `volatile`, `synchronized`.
- **Identifiers**: names you define (variables, methods, classes). Must follow Java naming rules.
- **Literals**: fixed values in code:
  - numeric: `123`, `123L`, `0xFF`, `0b1010`, `1_000_000`
  - char: `'a'`, `'\n'`
  - string: `"hello"`
  - boolean: `true`, `false`
  - null: `null`

### Data types (primitive vs non-primitive)

#### Primitives

- `byte`, `short`, `int`, `long`
- `float`, `double`
- `char`
- `boolean`

Properties:

- store raw values
- cannot be `null`
- default values exist for fields (e.g., `0` for `int`, `false` for `boolean`)

#### Non-primitive (reference) types

- classes (`String`, `ArrayList`, your classes)
- arrays (`int[]`, `String[]`)
- interfaces, enums, records

Properties:

- variables hold references (pointers)
- can be `null`

### Type casting (implicit, explicit)

#### Widening (implicit)

Safe conversion to a larger range type:

```java
int a = 10;
long b = a;     // widening
double c = b;   // widening
```

#### Narrowing (explicit)

May lose information; requires cast:

```java
long x = 1_000_000_000_000L;
int y = (int) x; // overflow risk
```

Interview trap: casting doesn’t “fix” overflow; it just truncates bits.

### Operators (arithmetic, logical, bitwise, ternary)

- **Arithmetic**: `+ - * / %`
  - integer division truncates: `5 / 2 == 2`
- **Logical**: `&& || !`
  - `&&` and `||` are short-circuit operators
- **Bitwise**: `& | ^ ~ << >> >>>`
  - `>>` arithmetic right shift (sign-extended)
  - `>>>` logical right shift (zero-filled)
- **Ternary**: `cond ? a : b`

Interview trap: `&` and `|` also work on booleans but do **not** short-circuit.

### Control statements (if, switch, loops)

- `if/else` for branching
- `switch` supports (modern Java): strings, enums, and enhanced switch expressions
- loops: `for`, enhanced `for-each`, `while`, `do-while`

Enhanced switch example:

```java
String dayType = switch (day) {
  case SAT, SUN -> "weekend";
  default -> "weekday";
};
```

### Wrapper classes

Wrappers represent primitives as objects:

- `Integer`, `Long`, `Double`, `Character`, `Boolean`, etc.

Why wrappers matter:

- Collections store objects, not primitives: `List<Integer>` not `List<int>`
- `null` can represent “missing”
- methods/utilities exist (`Integer.parseInt`, caching, etc.)

### Autoboxing & unboxing

- **Autoboxing**: primitive → wrapper
- **Unboxing**: wrapper → primitive

```java
Integer a = 10; // autobox
int b = a;      // unbox
```

Interview traps:

- `NullPointerException` on unboxing `null`:

```java
Integer x = null;
int y = x; // NPE
```

- `Integer` caching: small values may be cached, so `==` behaves unexpectedly:

```java
Integer a = 127, b = 127;
System.out.println(a == b); // often true (cached)

Integer c = 128, d = 128;
System.out.println(c == d); // often false (different objects)
```

Correct comparison: use `.equals()` for wrappers.

### Immutability basics

An object is **immutable** if its state cannot change after construction.

Why it matters:

- thread-safety by design
- easier to reason about
- safe sharing and caching

Examples:

- `String` is immutable
- `LocalDate` is immutable

Pitfall: `final` reference ≠ immutable object:

```java
final List<Integer> xs = new ArrayList<>();
xs.add(1); // allowed, object is mutable; reference is final
```

---

## 1.2 Object-Oriented Programming (OOP)

### Classes & objects

- **Class**: blueprint (fields + methods)
- **Object**: instance of a class at runtime

Example mental model:

```
Class: Car { color, speed, accelerate() }
Objects: car1 (red), car2 (blue)
```

### Constructors

- Initializes object state
- Can be overloaded
- If you define no constructor, compiler adds a default no-arg constructor.

Best practice: validate invariants inside constructors.

### `this` and `super`

- `this`: current instance
  - disambiguates fields vs parameters
  - can call another constructor: `this(...)` (must be first line)
- `super`: parent class constructor/method
  - `super(...)` must be first line in constructor if used

### Inheritance

Used for “is-a” relationships:

```java
class Animal { void speak() {} }
class Dog extends Animal { @Override void speak() { ... } }
```

Interview rule: don’t use inheritance only to reuse code—prefer composition when the relationship isn’t truly “is-a”.

### Method overriding vs overloading

#### Overriding (runtime polymorphism)

- same signature in subclass
- decided at runtime (dynamic dispatch)

```java
class A { void m() { System.out.println("A"); } }
class B extends A { @Override void m() { System.out.println("B"); } }
```

#### Overloading (compile-time)

- same method name, different parameter list
- chosen by compiler based on parameter types

Pitfall: overload resolution can be surprising with `null`, varargs, boxing.

### Polymorphism (compile-time vs runtime)

Interview-friendly breakdown:

- **Compile-time polymorphism**: method overloading
- **Runtime polymorphism**: method overriding + dynamic dispatch

Illustration:

```java
A x = new B();
x.m(); // prints "B" (runtime dispatch)
```

### Abstraction (abstract class vs interface)

Abstraction means exposing **what** an object can do, not **how** it does it.

#### Abstract class

- Can have:
  - abstract methods (no body)
  - concrete methods (shared implementation)
  - state (fields)
- Single inheritance only.

Use when:

- you control the hierarchy
- you want to share state/implementation across closely related classes

#### Interface

- Defines a **contract**.
- Supports multiple inheritance of type: a class can implement many interfaces.
- Can have `default` and `static` methods (Java 8+), but no instance state (except constants).

Use when:

- you want to model capabilities/roles (`Serializable`, `Comparable`, `Runnable`)
- you want flexible composition of behaviors

Key interview nuance:

- Interfaces are best for **polymorphic APIs**.
- Abstract classes are best for **shared base code** when inheritance is truly appropriate.

### Encapsulation

Encapsulation is not just “private fields + getters/setters”.
It is about:

- hiding internal representation
- protecting invariants
- exposing a minimal API

Example: enforcing invariants

```java
final class BankAccount {
  private long balance;

  public void deposit(long amount) {
    if (amount <= 0) throw new IllegalArgumentException("amount must be > 0");
    balance += amount;
  }
}
```

### SOLID principles (interview-ready)

#### S — Single Responsibility Principle

- One reason to change.
- Separate concerns: validation, persistence, business logic, IO.

#### O — Open/Closed Principle

- Extend behavior without modifying existing code.
- Achieved via interfaces + composition (e.g., Strategy pattern).

#### L — Liskov Substitution Principle

- Subtypes must not break assumptions of base type.
- Classic trap: `Square extends Rectangle` when setters exist.

#### I — Interface Segregation Principle

- Many small interfaces > one big “god” interface.

#### D — Dependency Inversion Principle

- Depend on abstractions, not concretions.
- Enables testing and swapping implementations.

### Composition vs inheritance

**Composition**: “has-a” relationship (assemble objects).

Prefer composition when:

- you need flexibility (swap behavior)
- inheritance would be used only for code reuse
- you want to avoid fragile base-class coupling

Illustration:

```
Inheritance:  Car extends Engine   (usually wrong)
Composition:  Car has Engine       (usually right)
```

### Coupling & cohesion

- **Coupling**: how dependent modules/classes are on each other.
  - low coupling is better (changes don’t ripple)
- **Cohesion**: how strongly related responsibilities inside a class/module are.
  - high cohesion is better (focused responsibility)

Interview heuristic:

- If a class has unrelated methods, cohesion is low → split it.
- If two classes change together frequently, coupling is high → redesign boundaries.

### Marker interfaces

A marker interface has **no methods** and signals metadata to the runtime/framework.

Classic examples:

- `Serializable`
- `Cloneable` (legacy)

Modern note: annotations are often preferred as markers, but marker interfaces are still used in JDK and some frameworks.

### Common interview traps (quick list)

- Using `==` instead of `.equals()` for objects (especially wrappers/strings).
- Confusing overloading (compile-time) with overriding (runtime).
- Misusing inheritance where composition is needed.
- Exposing internal mutable state from getters (breaks encapsulation).

### Practice interview questions

- Explain the difference between abstraction and encapsulation with an example.
- When would you choose an interface over an abstract class?
- Give a real-world LSP violation example.
- Explain coupling vs cohesion in the context of maintainable design.

