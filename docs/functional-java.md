# Functional Programming in Java (Interview Essentials)

Java is multi-paradigm. Functional-style code in Java relies on:

- **Lambdas**
- **Method references**
- **Functional interfaces**
- **Streams** (covered in the next chapter)

This chapter focuses on the language features and correctness/performance pitfalls interviewers probe.

## Definitions

- **Functional interface**: an interface with exactly one abstract method (SAM).
- **Lambda**: an inline function implementation that can be passed as a value.
- **Method reference**: shorthand for a lambda that simply calls a method.
- **Effectively final**: a local variable that is not reassigned, so it can be captured in a lambda.
- **Higher-order function**: a method that accepts a function or returns a function.
- **Side effects**: observable changes outside a function (e.g., mutating state, I/O).

## Illustrations

- **Lambda**: like writing a tiny unnamed function on a sticky note and handing it to an API.
- **Method reference**: pointing at an existing method instead of rewriting it.
- **Effectively final**: a snapshot of a variable captured at the moment the lambda is created.

## Code Examples

```java
java.util.List<String> names = java.util.List.of("Ann", "", "Bob");

java.util.List<String> cleaned = names.stream()
    .filter(s -> !s.isBlank())
    .map(String::trim)
    .toList();
```

## Interview Questions

1. What makes an interface "functional"?
2. Why must captured variables be effectively final?
3. How do lambdas differ from anonymous inner classes?
4. When should you avoid side effects in lambdas?
5. Explain method references and their common forms.

---

## 1) Lambdas and functional interfaces

### Functional interface

A **functional interface** has exactly **one abstract method** (SAM). It may have any number of `default`/`static` methods.

Common built-ins:

- `Predicate<T>`: `boolean test(T t)`
- `Function<T,R>`: `R apply(T t)`
- `Consumer<T>`: `void accept(T t)`
- `Supplier<T>`: `T get()`
- `UnaryOperator<T>`: `T apply(T t)` (same input/output)
- `BinaryOperator<T>`: `T apply(T a, T b)`

### Lambda syntax

```java
Predicate<String> nonEmpty = s -> !s.isEmpty();
Function<String, Integer> len = String::length;
Consumer<String> printer = System.out::println;
```

Interview point: lambdas are **not** anonymous inner classes, though they look similar; they compile differently and can be optimized (invokedynamic).

## 2) Method references

Four common forms:

- `ClassName::staticMethod`
- `instance::instanceMethod`
- `ClassName::instanceMethod` (receiver becomes first parameter)
- `ClassName::new` (constructor reference)

Example:

```java
Function<String, Integer> f1 = Integer::parseInt;      // static
Consumer<String> c1 = System.out::println;             // instance
BiPredicate<String, String> eq = String::equalsIgnoreCase; // Class::instanceMethod
Supplier<ArrayList<String>> listFactory = ArrayList::new;  // constructor
```

## 3) “Effectively final” and variable capture

Lambdas can capture:

- instance fields (`this.x`)
- static fields
- local variables that are **final or effectively final**

Effectively final means: assigned once, never reassigned.

```java
int base = 10;
Function<Integer, Integer> addBase = x -> x + base;
// base = 11; // would break: not effectively final
```

Interview pitfall: capturing a **mutable object** is allowed, but can create subtle bugs in concurrency.

## 4) `this` behavior (lambda vs anonymous class)

In a lambda:

- `this` refers to the **enclosing instance**

In an anonymous inner class:

- `this` refers to the **anonymous class instance**

Interviewers sometimes ask this to check deep understanding.

## 5) Default methods and diamond problem

Interfaces can have `default` methods. If a class implements two interfaces with the same default method signature, you must resolve it:

```java
class C implements A, B {
  @Override public void m() { A.super.m(); }
}
```

## 6) Functional composition

### `Function` composition

```java
Function<String, String> trim = String::trim;
Function<String, Integer> parse = Integer::parseInt;

Function<String, Integer> trimThenParse = trim.andThen(parse);
Function<String, Integer> parseAfterTrim = parse.compose(trim);
```

### Predicates

```java
Predicate<String> nonEmpty = s -> !s.isEmpty();
Predicate<String> shortStr = s -> s.length() < 5;

Predicate<String> ok = nonEmpty.and(shortStr.negate());
```

## 7) Side effects, referential transparency, and best practice

Interview-friendly rules:

- Prefer **pure functions** (same input → same output, no side effects) in stream pipelines.
- Avoid mutating external state inside lambdas.

Bad (race-prone with parallel streams):

```java
List<Integer> out = new ArrayList<>();
xs.stream().forEach(out::add); // side effect + not thread-safe
```

Better:

```java
List<Integer> out = xs.stream().collect(Collectors.toList());
```

## 8) Common functional pitfalls in Java

- **Autoboxing**: `Stream<Integer>` vs primitive streams (`IntStream`, `LongStream`, `DoubleStream`).
- **Accidental statefulness**: lambdas that depend on mutable captured variables.
- **Overuse**: functional style is great for transformations; don’t force it for complex imperative flows.

## 9) Interview questions

- What is a functional interface? How is it different from an interface with multiple abstract methods?
- Explain “effectively final”. Why does Java restrict captured locals?
- Difference between `this` in a lambda vs anonymous inner class?
- Why are side effects inside stream operations discouraged?

