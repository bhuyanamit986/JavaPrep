# Java Interview Preparation (Comprehensive)

This repository is a **complete, interview-focused Java preparation handbook**. It is organized exactly like a strong interview syllabus and covers every topic with explanations, pitfalls, and code illustrations.

## How to use this repo

- If you have <2–4 weeks, go **top-to-bottom** from Fundamentals → Strings → JVM/GC → Collections → Concurrency → Java 8+.
- If you have an interview in <7 days, focus on **Strings + Collections + Concurrency + JVM/GC** and revise the last-minute sections in each chapter.

## Table of Contents (requested structure)

### **1️⃣ Java Fundamentals (Must-Master)** — [`docs/01-java-fundamentals.md`](docs/01-java-fundamentals.md)

#### **1.1 Java Basics**

- History & evolution of Java
- Features of Java (WORA, platform independence)
- JDK vs JRE vs JVM
- Compilation & execution flow
- Java memory model (high-level overview)
- Keywords, identifiers, literals
- Data types (primitive vs non-primitive)
- Type casting (implicit, explicit)
- Operators (arithmetic, logical, bitwise, ternary)
- Control statements (if, switch, loops)
- Wrapper classes
- Autoboxing & unboxing
- Immutability basics

#### **1.2 Object-Oriented Programming (OOP)**

- Classes & objects
- Constructors
- `this` and `super`
- Inheritance
- Method overriding vs overloading
- Polymorphism (compile-time vs runtime)
- Abstraction (abstract class vs interface)
- Encapsulation
- SOLID principles
- Composition vs inheritance
- Coupling & cohesion
- Marker interfaces

### **2️⃣ String Handling (High-Frequency Area)** — [`docs/02-string-handling.md`](docs/02-string-handling.md)

#### **2.1 String Class Deep Dive**

- String immutability
- String pool & `intern()`
- `==` vs `equals()`
- `hashCode()` contract
- `StringBuilder` vs `StringBuffer`
- Performance implications
- Common string interview problems

### **3️⃣ Java Memory Management & JVM (Very Important)** — [`docs/03-jvm-memory-gc.md`](docs/03-jvm-memory-gc.md)

#### **3.1 JVM Architecture**

- ClassLoader subsystem
- Runtime data areas
  - Heap
  - Stack
  - Method area
  - PC register
  - Native method stack
- Execution engine
- JIT compiler

#### **3.2 Garbage Collection**

- GC basics
- Types of GC (Serial, Parallel, CMS, G1, ZGC)
- Young vs Old generation
- Minor vs Major GC
- Stop-the-world events
- Memory leaks & profiling
- `finalize()` (deprecated concepts)
- Soft, Weak, Phantom references

### **4️⃣ Exception Handling** — [`docs/04-exception-handling.md`](docs/04-exception-handling.md)

- Checked vs unchecked exceptions
- Error vs Exception
- try-catch-finally
- try-with-resources
- Custom exceptions
- Exception propagation
- Best practices
- Common interview traps

### **5️⃣ Collections Framework (Most Asked Topic)** — [`docs/05-collections-framework.md`](docs/05-collections-framework.md)

#### **5.1 Core Interfaces**

- Collection vs Collections
- List, Set, Queue, Deque, Map
- Iterator vs ListIterator
- Fail-fast vs fail-safe

#### **5.2 List Implementations**

- ArrayList
- LinkedList
- Vector
- CopyOnWriteArrayList
- Performance comparison

#### **5.3 Set Implementations**

- HashSet
- LinkedHashSet
- TreeSet
- Comparator vs Comparable
- equals() & hashCode() importance

#### **5.4 Map Implementations**

- HashMap (internal working, resizing, treeification)
- LinkedHashMap
- TreeMap
- ConcurrentHashMap
- Hashtable (legacy)

### **6️⃣ Multithreading & Concurrency (Critical for SDE Roles)** — [`docs/06-multithreading-concurrency.md`](docs/06-multithreading-concurrency.md)

#### **6.1 Thread Basics**

- Process vs thread
- Thread lifecycle
- Creating threads (Thread vs Runnable vs Callable)
- Thread priorities
- Daemon threads

#### **6.2 Synchronization & Locks**

- synchronized keyword
- Intrinsic locks
- Deadlock, livelock, starvation
- Volatile keyword
- Atomic classes
- ReentrantLock
- ReadWriteLock

#### **6.3 Executor Framework**

- ThreadPoolExecutor
- Fixed vs Cached vs Scheduled thread pool
- ForkJoinPool
- CompletableFuture
- Future vs CompletableFuture

### **7️⃣ Java 8+ Features (Very High ROI)** — [`docs/07-java8-plus.md`](docs/07-java8-plus.md)

#### **7.1 Lambda Expressions**

- Functional interfaces
- Method references
- Effectively final variables

#### **7.2 Streams API**

- Stream pipeline
- Intermediate vs terminal operations
- filter, map, flatMap
- reduce, collect
- Parallel streams
- Common stream problems

#### **7.3 Optional**

- Purpose & usage
- Best practices
- Anti-patterns

#### **7.4 Date & Time API**

- LocalDate, LocalTime, LocalDateTime
- ZonedDateTime
- Period vs Duration

### **8️⃣ File Handling & I/O** — [`docs/08-file-io.md`](docs/08-file-io.md)

- File class
- Streams vs Readers/Writers
- Serialization & deserialization
- transient keyword
- Externalization
- NIO basics
- Path, Files

### **9️⃣ Design Patterns (Interview Gold)** — [`docs/09-design-patterns.md`](docs/09-design-patterns.md)

#### **9.1 Creational**

- Singleton (all variations + pitfalls)
- Factory
- Abstract Factory
- Builder
- Prototype

#### **9.2 Structural**

- Adapter
- Decorator
- Proxy
- Composite

#### **9.3 Behavioral**

- Strategy
- Observer
- Template Method
- Command

### **🔟 Java Performance & Best Practices** — [`docs/10-performance-best-practices.md`](docs/10-performance-best-practices.md)

- Object creation cost
- Memory leaks
- Caching strategies
- Immutable objects
- Effective Java guidelines
- Common performance mistakes

### **1️⃣1️⃣ JVM, GC & Production Issues** — [`docs/11-production-issues.md`](docs/11-production-issues.md)

- Heap dump analysis
- Thread dump analysis
- OutOfMemoryError scenarios
- CPU spikes
- Memory leaks in collections
- Logging best practices

## Notes

- The chapters above are the “main path.”  
- Some earlier split-out notes also exist under `docs/` (e.g., `core-java.md`, `streams.md`) and are referenced from the main chapters where useful.
