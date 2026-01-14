# Streams (Including Parallel Streams)

Java Streams (introduced in Java 8) provide a declarative, functional approach to processing collections of data. They enable complex data processing pipelines that are readable, composable, and potentially parallelizable.

---

## 1) What is a Stream?

### Definition

A **Stream** is a sequence of elements that supports sequential and parallel aggregate operations. Unlike collections, streams:
- Do not store elements
- Are functional in nature (don't modify the source)
- Are lazy (computations happen only when terminal operation is invoked)
- Are potentially unbounded (can represent infinite sequences)
- Are consumable (can only be traversed once)

### Stream vs Collection

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Collection vs Stream                                  │
├───────────────────────┬─────────────────────────────────────────────────┤
│      Collection       │                   Stream                         │
├───────────────────────┼─────────────────────────────────────────────────┤
│ Data structure        │ Pipeline of computations                         │
│ Stores elements       │ Doesn't store elements                           │
│ Eager evaluation      │ Lazy evaluation                                  │
│ Iterable multiple     │ Can only be traversed ONCE                       │
│   times               │                                                  │
│ External iteration    │ Internal iteration                               │
│ Finite               │ Can be infinite                                  │
│ Mutates data          │ Functional (doesn't modify source)              │
└───────────────────────┴─────────────────────────────────────────────────┘
```

### Visual Representation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Stream Pipeline Anatomy                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────────┐      │
│   │ SOURCE  │ ──► │ filter  │ ──► │   map   │ ──► │  TERMINAL   │      │
│   │         │     │  (lazy) │     │  (lazy) │     │  (triggers) │      │
│   └─────────┘     └─────────┘     └─────────┘     └─────────────┘      │
│       │                                                  │              │
│   Collection      Intermediate Operations            Terminal Op        │
│   Array                (0 or more)                  (exactly 1)         │
│   Generator                                                             │
│   I/O channel                                                           │
│                                                                         │
│   Nothing happens until the terminal operation is called!               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2) Creating Streams

### From Collections

```java
// Most common: From Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();            // Sequential stream
Stream<String> stream2 = list.parallelStream();    // Parallel stream

// From Set
Set<Integer> set = Set.of(1, 2, 3);
Stream<Integer> stream3 = set.stream();

// From Map (via entrySet, keySet, or values)
Map<String, Integer> map = Map.of("a", 1, "b", 2);
Stream<Map.Entry<String, Integer>> stream4 = map.entrySet().stream();
Stream<String> stream5 = map.keySet().stream();
Stream<Integer> stream6 = map.values().stream();
```

### From Arrays

```java
// From array
String[] array = {"x", "y", "z"};
Stream<String> stream1 = Arrays.stream(array);
Stream<String> stream2 = Stream.of(array);
Stream<String> stream3 = Stream.of("x", "y", "z");  // Varargs

// Partial array
Stream<String> stream4 = Arrays.stream(array, 0, 2);  // "x", "y"
```

### Primitive Streams

```java
// Primitive streams avoid boxing overhead
IntStream intStream = IntStream.of(1, 2, 3, 4, 5);
IntStream intRange = IntStream.range(0, 10);       // 0 to 9 (exclusive)
IntStream intRangeClosed = IntStream.rangeClosed(1, 10);  // 1 to 10 (inclusive)

LongStream longStream = LongStream.of(1L, 2L, 3L);
DoubleStream doubleStream = DoubleStream.of(1.0, 2.0, 3.0);

// Converting between object and primitive streams
Stream<Integer> boxed = intStream.boxed();
IntStream unboxed = Stream.of(1, 2, 3).mapToInt(Integer::intValue);
```

### Generating Streams

```java
// Empty stream
Stream<String> empty = Stream.empty();

// Single element
Stream<String> single = Stream.of("hello");

// Infinite stream with generate (stateless supplier)
Stream<Double> randoms = Stream.generate(Math::random);  // Infinite!
Stream<String> constants = Stream.generate(() -> "constant");

// Infinite stream with iterate (stateful)
Stream<Integer> evens = Stream.iterate(0, n -> n + 2);  // 0, 2, 4, 6, ...

// Iterate with predicate (Java 9+) - finite
Stream<Integer> evensLimited = Stream.iterate(0, n -> n < 100, n -> n + 2);

// Concatenate streams
Stream<String> combined = Stream.concat(stream1, stream2);

// From other sources
Stream<String> lines = Files.lines(Paths.get("file.txt"));  // File lines
Stream<String> tokens = Pattern.compile(",").splitAsStream("a,b,c");  // Regex split
```

### Stream Creation Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Stream Creation Methods                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   FROM COLLECTIONS:                                                     │
│   collection.stream()         →  Sequential stream                      │
│   collection.parallelStream() →  Parallel stream                        │
│                                                                         │
│   FROM ARRAYS:                                                          │
│   Arrays.stream(array)        →  Stream from array                      │
│   Stream.of(values...)        →  Stream from varargs/array              │
│                                                                         │
│   PRIMITIVE STREAMS:                                                    │
│   IntStream.range(0, 10)      →  0, 1, 2, ..., 9                        │
│   IntStream.rangeClosed(1, 10)→  1, 2, 3, ..., 10                       │
│                                                                         │
│   GENERATORS:                                                           │
│   Stream.empty()              →  Empty stream                           │
│   Stream.generate(supplier)   →  Infinite from supplier                 │
│   Stream.iterate(seed, fn)    →  Infinite from seed + function          │
│   Stream.iterate(seed,pred,fn)→  Finite (Java 9+)                       │
│                                                                         │
│   FROM I/O:                                                             │
│   Files.lines(path)           →  Lines from file                        │
│   BufferedReader.lines()      →  Lines from reader                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3) Intermediate Operations (Lazy)

### Definition

**Intermediate Operations** transform a stream into another stream. They are **lazy** - they don't execute until a terminal operation is invoked. Multiple intermediate operations form a pipeline.

### Operation Categories

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Intermediate Operations                               │
├──────────────┬──────────────────────────────────────────────────────────┤
│   Category   │                    Operations                            │
├──────────────┼──────────────────────────────────────────────────────────┤
│ Filtering    │ filter, distinct, limit, skip, takeWhile*, dropWhile*   │
│ Mapping      │ map, mapToInt/Long/Double, flatMap, flatMapToInt/...    │
│ Sorting      │ sorted, sorted(Comparator)                              │
│ Peeking      │ peek (debugging only)                                   │
│ Stateless    │ filter, map, flatMap, peek, mapToXxx, flatMapToXxx      │
│ Stateful     │ distinct, sorted, limit, skip, takeWhile*, dropWhile*   │
└──────────────┴──────────────────────────────────────────────────────────┘
* Java 9+
```

### filter - Keep elements matching predicate

```java
// Syntax: filter(Predicate<T> predicate)

List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// Keep names starting with 'A' or 'C'
List<String> filtered = names.stream()
    .filter(name -> name.startsWith("A") || name.startsWith("C"))
    .collect(Collectors.toList());
// Result: ["Alice", "Charlie"]

// Multiple filters (equivalent to AND)
List<String> filtered2 = names.stream()
    .filter(name -> name.length() > 3)
    .filter(name -> name.contains("i"))
    .collect(Collectors.toList());
// Result: ["Alice", "Charlie", "David"]

// Filter with method reference
List<String> nonEmpty = strings.stream()
    .filter(s -> s != null)
    .filter(s -> !s.isEmpty())
    .collect(Collectors.toList());
// Or use Predicate composition
Predicate<String> notNullOrEmpty = Objects::nonNull;
notNullOrEmpty = notNullOrEmpty.and(s -> !s.isEmpty());
```

### map - Transform each element

```java
// Syntax: map(Function<T, R> mapper)

// Transform strings to their lengths
List<Integer> lengths = names.stream()
    .map(String::length)              // name -> name.length()
    .collect(Collectors.toList());
// Result: [5, 3, 7, 5]

// Transform objects
List<String> emails = users.stream()
    .map(User::getEmail)
    .collect(Collectors.toList());

// Chain transformations
List<String> processed = names.stream()
    .map(String::trim)
    .map(String::toUpperCase)
    .map(s -> s.substring(0, Math.min(3, s.length())))
    .collect(Collectors.toList());
```

### Visual: map vs flatMap

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        map vs flatMap                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   map:  Transform each element 1:1                                      │
│   ┌───┐   ┌───┐   ┌───┐                ┌───┐   ┌───┐   ┌───┐           │
│   │ A │ → │ B │   │ C │    map(f)  →   │f(A)│ → │f(B)│ → │f(C)│         │
│   └───┘   └───┘   └───┘                └───┘   └───┘   └───┘           │
│                                                                         │
│   flatMap:  Transform each element to stream, then flatten              │
│   ┌───┐   ┌───┐   ┌───┐              ┌──────────────────────┐          │
│   │ A │ → │ B │ → │ C │  flatMap(f)  │a1│a2│b1│b2│b3│c1│    │          │
│   └───┘   └───┘   └───┘      →       └──────────────────────┘          │
│     │       │       │                                                   │
│     ▼       ▼       ▼                                                   │
│  [a1,a2] [b1,b2,b3] [c1]                                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### flatMap - Flatten nested structures

```java
// Syntax: flatMap(Function<T, Stream<R>> mapper)

// Problem: map gives Stream<Stream<String>>
List<List<String>> listOfLists = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d", "e"),
    Arrays.asList("f")
);

// ❌ WRONG: Gives nested streams
Stream<Stream<String>> nested = listOfLists.stream()
    .map(List::stream);

// ✅ CORRECT: flatMap flattens to single stream
List<String> flat = listOfLists.stream()
    .flatMap(List::stream)    // Each list becomes a stream, then flattened
    .collect(Collectors.toList());
// Result: ["a", "b", "c", "d", "e", "f"]

// Real-world: Get all orders from all customers
List<Order> allOrders = customers.stream()
    .flatMap(customer -> customer.getOrders().stream())
    .collect(Collectors.toList());

// Split strings and flatten
List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split("\\s+")))
    .collect(Collectors.toList());

// Flatten optional values
List<String> emails = users.stream()
    .map(User::getOptionalEmail)     // Stream<Optional<String>>
    .flatMap(Optional::stream)        // Java 9+: Filter and flatten
    .collect(Collectors.toList());
```

### distinct - Remove duplicates

```java
// Syntax: distinct()
// Uses equals() to determine uniqueness

List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4);
List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// Result: [1, 2, 3, 4]

// For objects, ensure proper equals/hashCode implementation
List<User> uniqueUsers = users.stream()
    .distinct()                       // Uses User.equals()
    .collect(Collectors.toList());

// Alternative: Distinct by property
List<User> uniqueByEmail = users.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.toMap(
            User::getEmail,
            Function.identity(),
            (existing, replacement) -> existing  // Keep first
        ),
        m -> new ArrayList<>(m.values())
    ));
```

### sorted - Order elements

```java
// Syntax: sorted() or sorted(Comparator<T> comparator)

// Natural order (Comparable)
List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList());

// Custom comparator
List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());

// Reverse order
List<String> reversed = names.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());

// Multiple sort criteria
List<Employee> sorted = employees.stream()
    .sorted(Comparator
        .comparing(Employee::getDepartment)
        .thenComparing(Employee::getSalary).reversed()
        .thenComparing(Employee::getName))
    .collect(Collectors.toList());

// Null-safe sorting
List<String> sortedNullSafe = strings.stream()
    .sorted(Comparator.nullsLast(Comparator.naturalOrder()))
    .collect(Collectors.toList());
```

### limit and skip - Truncate and skip elements

```java
// Syntax: limit(long maxSize), skip(long n)

// First 5 elements
List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());

// Skip first 3, take next 5
List<Integer> page2 = numbers.stream()
    .skip(3)
    .limit(5)
    .collect(Collectors.toList());

// Pagination pattern
int pageSize = 10;
int pageNumber = 2;  // 0-indexed
List<Item> page = items.stream()
    .skip((long) pageNumber * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());
```

### takeWhile and dropWhile (Java 9+)

```java
// Syntax: takeWhile(Predicate), dropWhile(Predicate)

// Take while condition is true (stops at first false)
List<Integer> taken = Stream.of(1, 2, 3, 4, 5, 1, 2)
    .takeWhile(n -> n < 4)
    .collect(Collectors.toList());
// Result: [1, 2, 3]  - Stops when it hits 4

// Drop while condition is true (starts after first false)
List<Integer> dropped = Stream.of(1, 2, 3, 4, 5, 1, 2)
    .dropWhile(n -> n < 4)
    .collect(Collectors.toList());
// Result: [4, 5, 1, 2]  - Starts from 4

// Useful for sorted data
List<Integer> positives = sortedNumbers.stream()
    .dropWhile(n -> n <= 0)    // Skip non-positive
    .collect(Collectors.toList());
```

### peek - Debug intermediate results

```java
// Syntax: peek(Consumer<T> action)
// ⚠️ Only for debugging, not for side effects!

List<String> result = names.stream()
    .filter(s -> s.length() > 3)
    .peek(s -> System.out.println("After filter: " + s))
    .map(String::toUpperCase)
    .peek(s -> System.out.println("After map: " + s))
    .collect(Collectors.toList());

// ❌ BAD: Using peek for side effects
List<String> modified = new ArrayList<>();
names.stream()
    .peek(modified::add)          // DON'T do this!
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

## 4) Terminal Operations (Eager)

### Definition

**Terminal Operations** trigger the stream pipeline execution and produce a result or side effect. A stream can only have ONE terminal operation, after which the stream is consumed.

### Operation Categories

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     Terminal Operations                                  │
├──────────────────┬──────────────────────────────────────────────────────┤
│    Category      │                   Operations                         │
├──────────────────┼──────────────────────────────────────────────────────┤
│ Collection       │ collect, toList*, toArray                            │
│ Reduction        │ reduce, count, sum*, min, max, average*              │
│ Finding          │ findFirst, findAny                                   │
│ Matching         │ anyMatch, allMatch, noneMatch                        │
│ Iteration        │ forEach, forEachOrdered                              │
│ Short-circuiting │ findFirst, findAny, anyMatch, allMatch, noneMatch    │
└──────────────────┴──────────────────────────────────────────────────────┘
* Some operations are specific to primitive streams or Java 16+
```

### collect - Gather elements

```java
// Syntax: collect(Collector<T, A, R> collector)

// To List (Java 16+: can use toList())
List<String> list = stream.collect(Collectors.toList());
List<String> list2 = stream.toList();  // Java 16+ (unmodifiable)

// To Set
Set<String> set = stream.collect(Collectors.toSet());

// To specific collection
ArrayList<String> arrayList = stream
    .collect(Collectors.toCollection(ArrayList::new));
TreeSet<String> treeSet = stream
    .collect(Collectors.toCollection(TreeSet::new));

// To Map
Map<Long, User> userById = users.stream()
    .collect(Collectors.toMap(User::getId, Function.identity()));

// To Map with merge function (handle duplicates)
Map<String, User> userByEmail = users.stream()
    .collect(Collectors.toMap(
        User::getEmail,
        Function.identity(),
        (existing, replacement) -> existing  // Keep first on duplicate
    ));

// To String
String joined = strings.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// Result: "[a, b, c]"
```

### reduce - Combine elements into single value

```java
// Syntax: reduce(identity, accumulator) or reduce(accumulator)

// Sum with identity
int sum = numbers.stream()
    .reduce(0, Integer::sum);

// Product
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);

// Without identity (returns Optional)
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);

// Concatenate strings
String concatenated = strings.stream()
    .reduce("", String::concat);

// Complex reduction
// Sum of all order totals
BigDecimal totalRevenue = orders.stream()
    .map(Order::getTotal)
    .reduce(BigDecimal.ZERO, BigDecimal::add);
```

### Visual: How reduce works

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     reduce(identity, accumulator)                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Stream: [1, 2, 3, 4, 5]                                               │
│   reduce(0, (a, b) -> a + b)                                            │
│                                                                         │
│   Step 1:  0 + 1 = 1      (identity + first element)                    │
│   Step 2:  1 + 2 = 3      (result + second element)                     │
│   Step 3:  3 + 3 = 6      (result + third element)                      │
│   Step 4:  6 + 4 = 10     (result + fourth element)                     │
│   Step 5: 10 + 5 = 15     (result + fifth element)                      │
│                                                                         │
│   Final result: 15                                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Finding and Matching

```java
// findFirst - First element (respects encounter order)
Optional<String> first = names.stream()
    .filter(s -> s.startsWith("A"))
    .findFirst();

// findAny - Any element (may be faster in parallel)
Optional<String> any = names.parallelStream()
    .filter(s -> s.startsWith("A"))
    .findAny();

// anyMatch - Check if any element matches
boolean hasAdult = users.stream()
    .anyMatch(u -> u.getAge() >= 18);

// allMatch - Check if all elements match
boolean allAdults = users.stream()
    .allMatch(u -> u.getAge() >= 18);

// noneMatch - Check if no elements match
boolean noMinors = users.stream()
    .noneMatch(u -> u.getAge() < 18);
```

### count, min, max

```java
// count - Number of elements
long count = stream.count();

// min/max - Smallest/largest element
Optional<Integer> min = numbers.stream()
    .min(Comparator.naturalOrder());

Optional<Integer> max = numbers.stream()
    .max(Comparator.naturalOrder());

Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));
```

### forEach - Perform action on each element

```java
// forEach - Process each element (no guarantee of order in parallel)
names.stream()
    .forEach(System.out::println);

// forEachOrdered - Maintain encounter order (even in parallel)
names.parallelStream()
    .forEachOrdered(System.out::println);

// ⚠️ Don't modify external state in forEach with parallel streams!
```

### toArray

```java
// To Object array
Object[] array1 = stream.toArray();

// To typed array
String[] array2 = stream.toArray(String[]::new);
Integer[] array3 = numbers.stream().toArray(Integer[]::new);

// For primitives
int[] intArray = IntStream.range(0, 10).toArray();
```

---

## 5) Collectors Deep Dive

### Common Collectors

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Collectors Summary                                │
├─────────────────────────────┬───────────────────────────────────────────┤
│        Collector            │              Result                        │
├─────────────────────────────┼───────────────────────────────────────────┤
│ toList()                    │ List<T>                                   │
│ toSet()                     │ Set<T>                                    │
│ toCollection(supplier)      │ Custom collection                         │
│ toMap(keyFn, valueFn)       │ Map<K,V>                                  │
│ joining(delimiter)          │ String                                    │
│ counting()                  │ Long                                      │
│ summingInt/Long/Double(fn)  │ Sum                                       │
│ averagingInt/Long/Double(fn)│ Average                                   │
│ summarizingInt/Long/Double()│ Statistics                                │
│ maxBy(comparator)           │ Optional<T>                               │
│ minBy(comparator)           │ Optional<T>                               │
│ groupingBy(classifier)      │ Map<K, List<T>>                           │
│ partitioningBy(predicate)   │ Map<Boolean, List<T>>                     │
│ mapping(mapper, downstream) │ Transformed collection                    │
│ filtering(predicate, down)  │ Filtered collection (Java 9+)            │
│ flatMapping(mapper, down)   │ Flattened collection (Java 9+)           │
│ collectingAndThen(down, fn) │ Transform final result                    │
│ reducing(identity, op)      │ Reduction                                 │
└─────────────────────────────┴───────────────────────────────────────────┘
```

### groupingBy - Group elements

```java
// Basic grouping
Map<Department, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Grouping with downstream collector
Map<Department, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// Grouping with sum
Map<Department, Double> salaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summingDouble(Employee::getSalary)
    ));

// Grouping to different map type
Map<Department, List<Employee>> byDeptSorted = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        TreeMap::new,
        Collectors.toList()
    ));

// Multi-level grouping
Map<Department, Map<String, List<Employee>>> byDeptAndCity = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.groupingBy(Employee::getCity)
    ));

// Group and transform
Map<Department, Set<String>> namesByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(Employee::getName, Collectors.toSet())
    ));

// Group and find max
Map<Department, Optional<Employee>> highestPaidByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))
    ));
```

### Visual: groupingBy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     groupingBy Operation                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Input:  [E1(Sales), E2(IT), E3(Sales), E4(HR), E5(IT)]               │
│                                                                         │
│   groupingBy(Employee::getDepartment)                                   │
│                    ↓                                                    │
│   ┌─────────────────────────────────────────────────────────┐          │
│   │  Sales  → [E1, E3]                                      │          │
│   │  IT     → [E2, E5]                                      │          │
│   │  HR     → [E4]                                          │          │
│   └─────────────────────────────────────────────────────────┘          │
│                                                                         │
│   groupingBy(getDepartment, counting())                                 │
│                    ↓                                                    │
│   ┌─────────────────────────────────────────────────────────┐          │
│   │  Sales  → 2                                             │          │
│   │  IT     → 2                                             │          │
│   │  HR     → 1                                             │          │
│   └─────────────────────────────────────────────────────────┘          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### partitioningBy - Split into two groups

```java
// Partition by predicate (always creates exactly 2 groups)
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 50000));

List<Employee> highEarners = partitioned.get(true);
List<Employee> others = partitioned.get(false);

// Partition with downstream
Map<Boolean, Long> counts = employees.stream()
    .collect(Collectors.partitioningBy(
        e -> e.getSalary() > 50000,
        Collectors.counting()
    ));
```

### toMap - Create maps

```java
// Basic toMap
Map<Long, String> idToName = users.stream()
    .collect(Collectors.toMap(
        User::getId,        // Key mapper
        User::getName       // Value mapper
    ));

// toMap with merge function
Map<String, Integer> wordCount = words.stream()
    .collect(Collectors.toMap(
        Function.identity(),    // Key: the word itself
        word -> 1,              // Value: count 1
        Integer::sum            // Merge: sum counts for duplicates
    ));

// toMap with specific map type
Map<Long, User> idToUser = users.stream()
    .collect(Collectors.toMap(
        User::getId,
        Function.identity(),
        (existing, replacement) -> existing,
        LinkedHashMap::new      // Maintain insertion order
    ));
```

### joining - Concatenate strings

```java
// Simple join
String names = users.stream()
    .map(User::getName)
    .collect(Collectors.joining());
// Result: "AliceBobCharlie"

// Join with delimiter
String namesCsv = users.stream()
    .map(User::getName)
    .collect(Collectors.joining(", "));
// Result: "Alice, Bob, Charlie"

// Join with delimiter, prefix, and suffix
String namesJson = users.stream()
    .map(User::getName)
    .map(n -> "\"" + n + "\"")
    .collect(Collectors.joining(", ", "[", "]"));
// Result: ["Alice", "Bob", "Charlie"]
```

### collectingAndThen - Post-process result

```java
// Make result unmodifiable
List<String> unmodifiable = stream.collect(
    Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    )
);

// Get size after collecting
int size = stream.collect(
    Collectors.collectingAndThen(
        Collectors.toList(),
        List::size
    )
);

// Convert to different type
String[] array = stream.collect(
    Collectors.collectingAndThen(
        Collectors.toList(),
        list -> list.toArray(new String[0])
    )
);
```

---

## 6) Primitive Streams

### Why Use Primitive Streams?

```
┌─────────────────────────────────────────────────────────────────────────┐
│              Why Primitive Streams?                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Stream<Integer>:  ┌───┐   Boxing/Unboxing overhead!                   │
│                     │ 5 │ → Integer.valueOf(5) → int 5                  │
│                     └───┘   Every operation boxes/unboxes               │
│                                                                         │
│   IntStream:        ┌───┐   Native int, no boxing                       │
│                     │ 5 │   Direct primitive operations                 │
│                     └───┘   Better performance, less GC                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Primitive Stream Types

```java
// IntStream - for int primitives
IntStream intStream = IntStream.of(1, 2, 3, 4, 5);
IntStream range = IntStream.range(0, 10);
IntStream rangeClosed = IntStream.rangeClosed(1, 10);

// LongStream - for long primitives
LongStream longStream = LongStream.of(1L, 2L, 3L);
LongStream longRange = LongStream.range(0, 1_000_000);

// DoubleStream - for double primitives
DoubleStream doubleStream = DoubleStream.of(1.0, 2.0, 3.0);
```

### Conversion Between Streams

```java
// Object stream to primitive stream
Stream<String> strings = Stream.of("1", "2", "3");
IntStream ints = strings.mapToInt(Integer::parseInt);
LongStream longs = strings.mapToLong(Long::parseLong);
DoubleStream doubles = strings.mapToDouble(Double::parseDouble);

// Primitive stream to object stream
IntStream intStream = IntStream.range(0, 10);
Stream<Integer> boxed = intStream.boxed();
Stream<String> stringified = IntStream.range(0, 10)
    .mapToObj(Integer::toString);
```

### Primitive-Specific Operations

```java
IntStream numbers = IntStream.rangeClosed(1, 100);

// Aggregation methods
int sum = numbers.sum();
OptionalDouble average = numbers.average();
OptionalInt min = numbers.min();
OptionalInt max = numbers.max();
long count = numbers.count();

// Statistics (all at once)
IntSummaryStatistics stats = IntStream.rangeClosed(1, 100)
    .summaryStatistics();
System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());

// Range-based loops
IntStream.range(0, array.length)
    .forEach(i -> process(array[i], i));
```

---

## 7) Parallel Streams

### Definition

**Parallel Streams** split the data into multiple chunks and process them concurrently using the common ForkJoinPool. They can significantly speed up processing for CPU-intensive operations on large datasets.

### How Parallel Streams Work

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   Parallel Stream Execution                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   Source: [1, 2, 3, 4, 5, 6, 7, 8]                                      │
│                    │                                                    │
│                    ▼                                                    │
│            ┌───────┴───────┐                                           │
│            │    SPLIT      │                                           │
│            └───────┬───────┘                                           │
│        ┌───────────┼───────────┐                                       │
│        ▼           ▼           ▼                                       │
│    [1,2,3]      [4,5]       [6,7,8]                                    │
│        │           │           │                                       │
│   ┌────┴────┐ ┌────┴────┐ ┌────┴────┐                                 │
│   │Thread 1 │ │Thread 2 │ │Thread 3 │   ForkJoinPool                   │
│   │ process │ │ process │ │ process │                                  │
│   └────┬────┘ └────┬────┘ └────┬────┘                                 │
│        │           │           │                                       │
│        └───────────┼───────────┘                                       │
│                    ▼                                                    │
│            ┌───────┴───────┐                                           │
│            │   COMBINE     │                                           │
│            └───────┬───────┘                                           │
│                    ▼                                                    │
│               Final Result                                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Creating Parallel Streams

```java
// From collection
Stream<String> parallel1 = list.parallelStream();

// From existing stream
Stream<String> parallel2 = list.stream().parallel();

// Check if parallel
boolean isParallel = stream.isParallel();

// Convert back to sequential
Stream<String> sequential = parallel1.sequential();
```

### When to Use Parallel Streams

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 When to Use Parallel Streams                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ✅ USE when:                                                          │
│   • Large dataset (typically > 10,000 elements)                         │
│   • CPU-intensive operations (computation, not I/O)                     │
│   • Operations are stateless and independent                            │
│   • No shared mutable state                                             │
│   • Source collection supports efficient splitting (ArrayList, arrays)  │
│   • Accumulator/combiner in reduce are associative                      │
│                                                                         │
│   ❌ AVOID when:                                                        │
│   • Small dataset (parallel overhead dominates)                         │
│   • I/O-bound operations (network, database, file)                      │
│   • Operations have side effects or shared mutable state                │
│   • Order matters (findFirst, sorted, limit)                            │
│   • Source is LinkedList or ordered streams                             │
│   • Running in a web server (common pool contention)                    │
│   • Operations are blocking                                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Parallel Stream Pitfalls

```java
// ❌ BAD: Shared mutable state
List<Integer> results = new ArrayList<>();  // Not thread-safe!
numbers.parallelStream()
    .filter(n -> n > 0)
    .forEach(results::add);  // Race condition!

// ✅ GOOD: Use collect instead
List<Integer> results = numbers.parallelStream()
    .filter(n -> n > 0)
    .collect(Collectors.toList());

// ❌ BAD: Non-associative accumulator
// This won't work correctly in parallel!
int result = numbers.parallelStream()
    .reduce(0, (a, b) -> a - b);  // Subtraction is NOT associative!

// ✅ GOOD: Associative operations
int sum = numbers.parallelStream()
    .reduce(0, Integer::sum);  // Addition IS associative

// ❌ BAD: I/O in parallel streams
users.parallelStream()
    .forEach(user -> saveToDatabase(user));  // Thread pool starvation!

// ❌ BAD: Order-dependent with findFirst
Optional<Integer> first = numbers.parallelStream()
    .filter(n -> n > 100)
    .findFirst();  // May be slower than sequential due to ordering requirement

// ✅ BETTER for parallel: findAny (if order doesn't matter)
Optional<Integer> any = numbers.parallelStream()
    .filter(n -> n > 100)
    .findAny();  // Can return any matching element
```

### Custom Thread Pool

```java
// By default, parallel streams use ForkJoinPool.commonPool()
// You can use a custom pool for isolation

ForkJoinPool customPool = new ForkJoinPool(4);  // 4 threads
try {
    List<String> result = customPool.submit(() ->
        largeList.parallelStream()
            .map(this::expensiveOperation)
            .collect(Collectors.toList())
    ).get();
} finally {
    customPool.shutdown();
}
```

---

## 8) Stream Operation Characteristics

### Stateful vs Stateless Operations

```
┌─────────────────────────────────────────────────────────────────────────┐
│              Stateless vs Stateful Operations                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   STATELESS (Preferred):                                                │
│   • Process each element independently                                  │
│   • No need to remember previous elements                               │
│   • Examples: filter, map, flatMap, peek                                │
│   • Better parallel performance                                         │
│                                                                         │
│   STATEFUL (Use with care):                                             │
│   • May need to see other elements                                      │
│   • May need to process entire stream first                             │
│   • Examples: distinct, sorted, limit, skip                             │
│   • Can impact parallel performance                                     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Short-Circuiting Operations

```java
// Short-circuiting operations don't process all elements

// findFirst/findAny - Stops at first match
Optional<Integer> found = numbers.stream()
    .filter(n -> n > 100)
    .findFirst();  // Stops when found

// anyMatch/allMatch/noneMatch - Stops when result is determined
boolean hasNegative = numbers.stream()
    .anyMatch(n -> n < 0);  // Stops at first negative

// limit - Stops after n elements
List<Integer> first10 = numbers.stream()
    .filter(n -> n > 0)
    .limit(10)  // Short-circuits after 10 positive numbers
    .collect(Collectors.toList());
```

---

## 9) Common Stream Patterns

### Pattern: Processing with Index

```java
// Using IntStream.range
List<String> list = Arrays.asList("a", "b", "c");
IntStream.range(0, list.size())
    .forEach(i -> System.out.println(i + ": " + list.get(i)));

// Using AtomicInteger (for sequential streams only)
AtomicInteger index = new AtomicInteger(0);
list.stream()
    .map(s -> index.getAndIncrement() + ": " + s)
    .forEach(System.out::println);
```

### Pattern: Collecting to Different Types

```java
// To Array
String[] array = stream.toArray(String[]::new);

// To LinkedHashMap (maintain order)
Map<Long, User> orderedMap = users.stream()
    .collect(Collectors.toMap(
        User::getId,
        Function.identity(),
        (a, b) -> a,
        LinkedHashMap::new
    ));

// To ConcurrentHashMap
ConcurrentMap<Long, User> concurrentMap = users.stream()
    .collect(Collectors.toConcurrentMap(
        User::getId,
        Function.identity()
    ));
```

### Pattern: Handling Checked Exceptions

```java
// Wrapper for checked exceptions
public static <T, R> Function<T, R> wrap(CheckedFunction<T, R> fn) {
    return t -> {
        try {
            return fn.apply(t);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    };
}

@FunctionalInterface
interface CheckedFunction<T, R> {
    R apply(T t) throws Exception;
}

// Usage
List<String> contents = files.stream()
    .map(wrap(Files::readString))
    .collect(Collectors.toList());
```

### Pattern: Running Totals / Scan

```java
// Cumulative sum
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
AtomicInteger running = new AtomicInteger(0);
List<Integer> cumulative = numbers.stream()
    .map(n -> running.addAndGet(n))
    .collect(Collectors.toList());
// Result: [1, 3, 6, 10, 15]
```

### Pattern: Distinct by Property

```java
// Distinct by specific property
public static <T> Predicate<T> distinctByKey(Function<? super T, ?> keyExtractor) {
    Set<Object> seen = ConcurrentHashMap.newKeySet();
    return t -> seen.add(keyExtractor.apply(t));
}

// Usage
List<User> uniqueByEmail = users.stream()
    .filter(distinctByKey(User::getEmail))
    .collect(Collectors.toList());
```

---

## 10) Common Interview Questions

### Stream Basics

**Q1: What is the difference between intermediate and terminal operations?**

**Answer:**
- **Intermediate operations** (filter, map, sorted) transform a stream into another stream. They are lazy and don't execute until a terminal operation is invoked.
- **Terminal operations** (collect, forEach, reduce) produce a result or side effect and consume the stream. A stream can only have one terminal operation.

**Q2: Why are streams lazy? What are the benefits?**

**Answer:**
- Streams are lazy because intermediate operations don't execute until a terminal operation is invoked.
- Benefits:
  - **Efficiency**: Avoids unnecessary computation
  - **Short-circuiting**: Operations like `findFirst` can stop early
  - **Memory efficiency**: Process one element at a time
  - **Optimization**: Stream can optimize the pipeline

**Q3: Can a stream be reused? Why or why not?**

**Answer:**
No, streams can only be traversed once. After a terminal operation, the stream is considered consumed. Attempting to reuse it throws `IllegalStateException`. This is by design - streams represent a single pass over data.

```java
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
stream.forEach(System.out::println);  // IllegalStateException!
```

### map vs flatMap

**Q4: What is the difference between map and flatMap?**

**Answer:**
- `map`: Transforms each element 1:1. If transformation returns a stream, you get `Stream<Stream<T>>`.
- `flatMap`: Transforms each element to a stream, then flattens all streams into one. Avoids nested streams.

```java
// map: [[1,2], [3,4]] -> Stream<Stream<Integer>>
// flatMap: [[1,2], [3,4]] -> Stream<Integer>: [1,2,3,4]
```

### Collectors

**Q5: What is the difference between groupingBy and partitioningBy?**

**Answer:**
- `groupingBy`: Groups elements by a classifier function into multiple groups (Map<K, List<T>>)
- `partitioningBy`: Splits elements by a predicate into exactly two groups (Map<Boolean, List<T>>)

**Q6: What happens with duplicate keys in toMap?**

**Answer:**
By default, `toMap` throws `IllegalStateException` on duplicate keys. To handle duplicates, provide a merge function:
```java
Collectors.toMap(keyMapper, valueMapper, (existing, replacement) -> existing)
```

### Parallel Streams

**Q7: When should you use parallel streams?**

**Answer:**
Use parallel streams when:
- Large dataset (> 10,000 elements)
- CPU-intensive, independent operations
- No shared mutable state
- Source supports efficient splitting (ArrayList, arrays)
- Not running in a thread-constrained environment

Avoid when:
- Small datasets
- I/O-bound operations
- Operations with side effects
- Order matters

**Q8: Why is this code problematic?**
```java
List<Integer> results = new ArrayList<>();
numbers.parallelStream().forEach(results::add);
```

**Answer:**
`ArrayList` is not thread-safe. Multiple threads adding concurrently causes race conditions. Use `collect(Collectors.toList())` instead.

### reduce

**Q9: What does the identity parameter in reduce do?**

**Answer:**
The identity is the initial value and also the default return value for empty streams. It must be a value that when combined with any element, returns that element (e.g., 0 for addition, 1 for multiplication, empty string for concatenation).

**Q10: What does "associative" mean for reduce operations?**

**Answer:**
An operation is associative if `(a op b) op c == a op (b op c)`. This is required for parallel reduce to work correctly, as the order of combining partial results shouldn't affect the final result. Addition and multiplication are associative; subtraction and division are not.

---

## 11) Quick Reference

### Creating Streams

```java
collection.stream()              // From collection
Arrays.stream(array)             // From array
Stream.of(a, b, c)               // From values
Stream.empty()                   // Empty stream
Stream.iterate(seed, fn)         // Infinite iterate
Stream.generate(supplier)        // Infinite generate
IntStream.range(0, 10)           // 0-9
IntStream.rangeClosed(1, 10)     // 1-10
Files.lines(path)                // From file
```

### Intermediate Operations

```java
.filter(predicate)               // Keep matching
.map(function)                   // Transform 1:1
.flatMap(function)               // Transform and flatten
.distinct()                      // Remove duplicates
.sorted()                        // Natural order
.sorted(comparator)              // Custom order
.limit(n)                        // First n elements
.skip(n)                         // Skip first n
.peek(consumer)                  // Debug
.takeWhile(predicate)            // Take while true (Java 9)
.dropWhile(predicate)            // Drop while true (Java 9)
```

### Terminal Operations

```java
.collect(collector)              // Collect to container
.toList()                        // To unmodifiable list (Java 16)
.toArray(generator)              // To array
.forEach(consumer)               // Iterate
.reduce(identity, op)            // Reduce to single value
.count()                         // Count elements
.findFirst()                     // First element
.findAny()                       // Any element
.anyMatch(predicate)             // Any match?
.allMatch(predicate)             // All match?
.noneMatch(predicate)            // None match?
.min(comparator)                 // Minimum
.max(comparator)                 // Maximum
```

### Common Collectors

```java
Collectors.toList()              // To List
Collectors.toSet()               // To Set
Collectors.toMap(k, v)           // To Map
Collectors.joining(delim)        // Join strings
Collectors.counting()            // Count
Collectors.groupingBy(fn)        // Group
Collectors.partitioningBy(pred)  // Partition
Collectors.mapping(fn, down)     // Map then collect
```

---

*Streams enable powerful, expressive data processing in Java!*
