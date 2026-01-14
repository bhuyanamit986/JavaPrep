# Java Interview Preparation (Comprehensive)

This repository is a structured, detailed preparation guide for **cracking Java interviews**—from core language fundamentals to **Streams**, **Concurrency**, and **Multithreading**, plus the JVM, performance, testing, and commonly asked system/design topics.

## How to use this repo

- Start with the **Study plan** below if you’re new or rusty.
- Use the **Table of Contents** to jump to topics.
- Each chapter includes: explanations, common pitfalls, interview questions, and code examples.

## Study plan (suggested)

- **Week 1**: Core Java + OOP + Exceptions
- **Week 2**: Collections + Generics + JVM basics
- **Week 3**: Functional programming + Streams
- **Week 4**: Concurrency + Multithreading (deep dive)
- **Week 5**: Testing + Build tools + Design patterns + Spring basics (if relevant)
- Ongoing: DSA practice + revise the cheat sheets

## Table of Contents

1. **Core Java fundamentals** — [`docs/core-java.md`](docs/core-java.md)  
   - Syntax & types, references vs primitives, pass-by-value
   - `String`/immutability, `equals`/`hashCode`, `Comparable`/`Comparator`
   - `static`, initialization order, `final`, nested classes
   - Date/Time API, Optional, annotations (basics)

2. **Object-Oriented Programming (OOP) + SOLID** — [`docs/oop-solid.md`](docs/oop-solid.md)  
   - Encapsulation, inheritance, polymorphism, abstraction
   - Composition vs inheritance, LSP pitfalls
   - Interfaces vs abstract classes, sealed classes (modern Java)

3. **Exceptions, error handling & logging** — [`docs/exceptions-logging.md`](docs/exceptions-logging.md)  
   - Checked vs unchecked, `try-with-resources`, suppression
   - Custom exceptions, best practices, logging basics

4. **Collections & Generics** — [`docs/collections-generics.md`](docs/collections-generics.md)  
   - `List`/`Set`/`Map`, complexity, common implementations
   - Fail-fast iterators, immutability, `Collections` vs `Collectors`
   - Generics, invariance, wildcards (`extends`/`super`), type erasure

5. **Functional programming in Java** — [`docs/functional-java.md`](docs/functional-java.md)  
   - Lambdas, method references, functional interfaces
   - Effectively final, closure capture, best practices

6. **Streams (including parallel streams)** — [`docs/streams.md`](docs/streams.md)  
   - Stream pipeline model, laziness, intermediate vs terminal ops
   - `map`/`flatMap`/`filter`/`reduce`/`collect`
   - `Collectors` deep dive, grouping, partitioning
   - Parallel streams: when to use, when to avoid

7. **Concurrency & Multithreading (deep dive)** — [`docs/concurrency-multithreading.md`](docs/concurrency-multithreading.md)  
   - Threads, `Runnable`/`Callable`, thread lifecycle
   - `synchronized`, monitors, `wait/notify`, deadlocks
   - Java Memory Model, `volatile`, happens-before
   - Executors, thread pools, Futures, `CompletableFuture`
   - Locks, atomics, concurrent collections, fork/join
   - Common pitfalls (race conditions, visibility bugs, contention)

8. **JVM, memory model & garbage collection** — [`docs/jvm-memory-gc.md`](docs/jvm-memory-gc.md)  
   - Stack vs heap, object layout (conceptual), class loading
   - GC concepts, tuning basics, avoiding leaks

9. **I/O, NIO.2, and networking basics** — [`docs/io-nio-networking.md`](docs/io-nio-networking.md)  
   - Streams, buffers, channels, files, serialization pitfalls
   - HTTP basics, sockets overview (interview-level)

10. **Design patterns (practical interview set)** — [`docs/design-patterns.md`](docs/design-patterns.md)  
   - Singleton pitfalls, Factory, Strategy, Builder, Adapter
   - Observer, Decorator, Template Method; when to apply

11. **Testing & quality** — [`docs/testing.md`](docs/testing.md)  
   - Unit vs integration tests, mocking, test design
   - JUnit basics, parameterized tests, test anti-patterns

12. **Build tools & dependency management** — [`docs/build-tools.md`](docs/build-tools.md)  
   - Maven/Gradle basics, dependency scopes, BOMs, reproducibility

13. **Spring / Spring Boot (optional but common)** — [`docs/spring-boot.md`](docs/spring-boot.md)  
   - DI, beans, scopes, lifecycle; REST controllers
   - Transactions, common annotations, pitfalls

14. **DSA for Java interviews (minimal but essential)** — [`docs/dsa-for-java.md`](docs/dsa-for-java.md)  
   - Big-O, arrays/strings, hashmaps, stacks/queues, trees/graphs
   - Patterns: sliding window, two pointers, BFS/DFS, DP basics

15. **Cheat sheets & last-minute revision** — [`docs/cheat-sheets.md`](docs/cheat-sheets.md)  
   - Rapid review for Collections, Streams, Concurrency, JVM

## Contributing / extending

Add new notes under `docs/` and link them from this README.
