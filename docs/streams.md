# Streams (including Parallel Streams)

Streams are a **declarative, functional** way to process data. Interviewers care about:

- pipeline model (lazy vs eager)
- intermediate vs terminal operations
- `collect` and `Collectors`
- short-circuiting
- parallel streams trade-offs

## 1) Stream pipeline model

A stream pipeline typically looks like:

1. **Source**: collection, array, generator (`Stream.of`, `Files.lines`, etc.)
2. **Intermediate ops** (lazy): `map`, `filter`, `flatMap`, `sorted`, `distinct`, `peek`, ...
3. **Terminal op** (eager): `collect`, `forEach`, `reduce`, `count`, `findFirst`, `anyMatch`, ...

Key points:

- Intermediate operations are **lazy**: they don’t run until a terminal operation is invoked.
- Streams are **single-use**: once consumed, they can’t be reused.

## 2) Intermediate operations (high-signal set)

### `map`

Transforms each element:

```java
List<Integer> lengths = names.stream()
    .map(String::length)
    .toList();
```

### `filter`

Keeps elements that match a predicate:

```java
List<String> longNames = names.stream()
    .filter(s -> s.length() > 10)
    .toList();
```

### `flatMap`

Transforms each element into a stream and flattens:

```java
List<String> tokens = lines.stream()
    .flatMap(line -> Arrays.stream(line.split("\\s+")))
    .toList();
```

Interview pitfall: using `map` instead of `flatMap` yields `Stream<Stream<T>>`.

### `distinct`

Removes duplicates using `equals`/`hashCode`.

### `sorted`

Sorts elements (stateful). With custom comparator:

```java
List<Person> byAge = people.stream()
    .sorted(Comparator.comparingInt(Person::age))
    .toList();
```

### `limit`, `skip` (often paired)

Useful for pagination and sampling.

### `peek`

For debugging/inspection. Avoid using `peek` for side effects in production pipelines.

## 3) Terminal operations

### `collect`

Most important for interviews; see Collectors section below.

### `reduce`

Combines elements into one value. Must be associative for parallel use.

```java
int sum = xs.stream().reduce(0, Integer::sum);
```

### Short-circuiting

- `findFirst`, `findAny`
- `anyMatch`, `allMatch`, `noneMatch`
- `limit`

These may stop processing early.

## 4) Primitive streams

Prefer primitive streams to avoid boxing overhead:

- `IntStream`, `LongStream`, `DoubleStream`

Examples:

```java
int sum = ints.stream().mapToInt(Integer::intValue).sum();
double avg = ints.stream().mapToInt(Integer::intValue).average().orElse(0);
```

## 5) `Collectors` deep dive

### Common collectors

- `toList()`, `toSet()`, `toCollection(...)`
- `toMap(keyMapper, valueMapper, mergeFn, mapSupplier)`
- `joining(delimiter, prefix, suffix)`
- `counting()`, `summingInt(...)`, `averagingInt(...)`
- `groupingBy(classifier, downstream)`
- `partitioningBy(predicate, downstream)`
- `mapping(mapper, downstream)`

### `toList()` vs `Collectors.toList()`

- `Stream#toList()` (modern Java) returns an **unmodifiable** list.
- `Collectors.toList()` returns a list with **no strict mutability guarantee** (often mutable `ArrayList`), but don’t rely on that.

### `toMap` and duplicate keys

Default `toMap` throws `IllegalStateException` on duplicate keys.

```java
Map<String, Person> byId = people.stream().collect(
    Collectors.toMap(Person::id, p -> p)
);
```

To handle duplicates, provide a merge function:

```java
Map<String, Person> latestById = people.stream().collect(
    Collectors.toMap(Person::id, p -> p, (a, b) -> b)
);
```

### `groupingBy` vs `partitioningBy`

- `groupingBy`: groups into many buckets by key
- `partitioningBy`: groups into exactly two buckets (`true`/`false`)

```java
Map<Dept, List<Employee>> byDept =
    employees.stream().collect(Collectors.groupingBy(Employee::dept));

Map<Boolean, List<Employee>> byActive =
    employees.stream().collect(Collectors.partitioningBy(Employee::active));
```

Downstream collectors (power feature):

```java
Map<Dept, Long> headcount =
    employees.stream().collect(Collectors.groupingBy(Employee::dept, Collectors.counting()));
```

### `collect` vs manual mutation

Prefer collectors over side effects:

- safer (especially under parallelism)
- clearer intent

## 6) Ordering and determinism

Streams can be:

- **ordered** (e.g., from `List`)
- **unordered** (e.g., from `HashSet`)

Operations like `findFirst` depend on encounter order; `findAny` may return any element (and may be faster in parallel).

## 7) Parallel streams

Parallel streams split work across the common ForkJoinPool.

### When parallel streams can help

- Large data sets
- CPU-bound, independent computations
- Operations are stateless and associative
- Minimal contention and minimal shared mutable state

### When parallel streams are a bad idea

- I/O-bound work (threads block; pool starvation risk)
- Small data sets (parallel overhead dominates)
- Stateful lambdas or side effects
- Order-dependent operations (`sorted`, `findFirst`) can reduce parallel benefits
- Running inside app servers where shared common pool can interfere with other workloads

### Common ForkJoinPool gotcha

Parallel streams use `ForkJoinPool.commonPool()` by default.

Interview takeaway: treat parallel streams as an optimization tool, not a default. Benchmark and be aware of environment constraints.

## 8) Stream pitfalls and best practices

- Don’t reuse a stream after a terminal op.
- Avoid `peek` side effects.
- Prefer primitive streams when aggregating numeric data.
- Prefer `Collectors` to manual mutation.
- Be careful with `sorted()` (stateful and potentially expensive).

## 9) Interview questions

- Explain lazy evaluation in streams. What triggers execution?
- Difference between `map` and `flatMap`?
- How does `groupingBy` with downstream collectors work?
- Why can `toMap` throw at runtime? How do you resolve duplicate keys?
- When would you avoid parallel streams?

