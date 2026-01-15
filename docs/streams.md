# Streams (including Parallel Streams)

Streams are a **declarative, functional** way to process data in Java. They provide a powerful abstraction for working with sequences of elements, supporting both sequential and parallel operations. This guide covers everything from basics to advanced patterns commonly asked in interviews.

---

## Definitions

- **Stream**: A sequence of elements supporting sequential and parallel aggregate operations. Unlike collections, streams do not store elements; they carry values from a source through a pipeline.

- **Stream pipeline**: A chain of operations consisting of a source, zero or more intermediate operations, and a terminal operation.

- **Source**: The origin of stream elements (collection, array, generator, I/O channel, etc.).

- **Intermediate operation**: A lazy transformation that returns a new stream (e.g., `map`, `filter`, `sorted`). These operations don't execute until a terminal operation is invoked.

- **Terminal operation**: An operation that triggers the pipeline execution and produces a result or side effect (e.g., `collect`, `forEach`, `reduce`). A stream can only have one terminal operation.

- **Lazy evaluation**: Intermediate operations are not executed until a terminal operation is called. This enables optimizations like short-circuiting.

- **Short-circuiting**: An operation that can produce a result without processing all elements (e.g., `findFirst`, `anyMatch`, `limit`).

- **Collector**: A mutable reduction operation that accumulates elements into a container (e.g., List, Map, String).

- **Parallel stream**: A stream that divides elements into multiple chunks and processes them concurrently using the ForkJoin framework.

- **Stateless operation**: An operation where the result for any element doesn't depend on other elements (e.g., `map`, `filter`).

- **Stateful operation**: An operation that may need to see all elements before producing a result (e.g., `sorted`, `distinct`).

---

## Illustrations

### Stream Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                        STREAM PIPELINE                               │
│                                                                      │
│   ┌──────────┐     ┌─────────────────┐     ┌────────────────────┐   │
│   │  SOURCE  │────▶│  INTERMEDIATE   │────▶│    TERMINAL       │   │
│   │          │     │   OPERATIONS    │     │   OPERATION       │   │
│   │ • List   │     │   (0 or more)   │     │   (exactly 1)     │   │
│   │ • Array  │     │                 │     │                   │   │
│   │ • File   │     │ • filter        │     │ • collect         │   │
│   │ • Range  │     │ • map           │     │ • forEach         │   │
│   │ • Stream │     │ • flatMap       │     │ • reduce          │   │
│   │   .of()  │     │ • sorted        │     │ • count           │   │
│   └──────────┘     │ • distinct      │     │ • findFirst       │   │
│                    │ • limit/skip    │     │ • anyMatch        │   │
│                    └─────────────────┘     └────────────────────┘   │
│                           │                        │                 │
│                           │                        │                 │
│                     LAZY (deferred)          EAGER (triggers         │
│                     Returns Stream           execution)              │
│                                              Returns result          │
└─────────────────────────────────────────────────────────────────────┘
```

### Lazy Evaluation Visualization

```
Without terminal operation:
┌─────────┐    ┌────────┐    ┌─────┐
│ List    │───▶│ filter │───▶│ map │───▶ Nothing happens!
└─────────┘    └────────┘    └─────┘     (Pipeline not executed)

With terminal operation:
┌─────────┐    ┌────────┐    ┌─────┐    ┌─────────┐
│ List    │───▶│ filter │───▶│ map │───▶│ collect │───▶ Results!
└─────────┘    └────────┘    └─────┘    └─────────┘
                                         (Triggers execution)

Processing order (vertical slicing, not horizontal):
┌───────────────────────────────────────────────────┐
│ Element 1: source → filter → map → collect        │
│ Element 2: source → filter → (filtered out)       │
│ Element 3: source → filter → map → collect        │
│ ...                                               │
└───────────────────────────────────────────────────┘
```

### map vs flatMap

```
map: One-to-One transformation
┌─────────┐        ┌─────────┐
│ ["Hello │   map  │ [5, 5]  │
│  World"]│───────▶│         │
└─────────┘ length └─────────┘

flatMap: One-to-Many (then flattened)
┌─────────┐        ┌──────────────┐        ┌─────────────────┐
│ ["Hello │ split  │ [['H','e',   │flatten │ ['H','e','l',   │
│  World"]│───────▶│  'l','l','o']│───────▶│ 'l','o','W',    │
└─────────┘        │ ['W','o',    │        │ 'o','r','l','d']│
                   │  'r','l','d']│        └─────────────────┘
                   └──────────────┘
```

### Parallel Stream Processing

```
Sequential:
┌───────────────────────────────────────────────┐
│ Thread 1: [1, 2, 3, 4, 5, 6, 7, 8] ──────────▶│
└───────────────────────────────────────────────┘

Parallel:
┌─────────────────────────────────────────────────────────────┐
│ ForkJoinPool splits work:                                   │
│                                                             │
│ Thread 1: [1, 2] ───────▶ ┐                                 │
│ Thread 2: [3, 4] ───────▶ │                                 │
│ Thread 3: [5, 6] ───────▶ ├───▶ Combine Results             │
│ Thread 4: [7, 8] ───────▶ ┘                                 │
│                                                             │
│ Uses ForkJoinPool.commonPool() by default                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Code Examples

### 1) Creating Streams

```java
import java.util.stream.*;
import java.util.*;
import java.nio.file.*;

// From Collection
List<String> list = List.of("a", "b", "c");
Stream<String> fromList = list.stream();
Stream<String> parallelFromList = list.parallelStream();

// From Array
String[] array = {"a", "b", "c"};
Stream<String> fromArray = Arrays.stream(array);
Stream<String> fromArrayPartial = Arrays.stream(array, 0, 2);  // [0, 2)

// From values
Stream<String> fromValues = Stream.of("a", "b", "c");
Stream<Integer> singleElement = Stream.of(1);

// Empty stream
Stream<String> empty = Stream.empty();

// Primitive streams (avoid boxing)
IntStream intStream = IntStream.of(1, 2, 3);
IntStream range = IntStream.range(0, 10);         // 0 to 9
IntStream rangeClosed = IntStream.rangeClosed(1, 10);  // 1 to 10
LongStream longStream = LongStream.of(1L, 2L, 3L);
DoubleStream doubleStream = DoubleStream.of(1.0, 2.0, 3.0);

// Infinite streams
Stream<Integer> infinite = Stream.iterate(0, n -> n + 2);  // 0, 2, 4, 6...
Stream<Integer> boundedIterate = Stream.iterate(0, n -> n < 100, n -> n + 2);  // Java 9+
Stream<Double> random = Stream.generate(Math::random);
Stream<UUID> uuids = Stream.generate(UUID::randomUUID);

// From file
Stream<String> lines = Files.lines(Path.of("file.txt"));  // Remember to close!

// From BufferedReader
BufferedReader reader = new BufferedReader(new FileReader("file.txt"));
Stream<String> readerLines = reader.lines();

// From String (chars)
IntStream chars = "Hello".chars();  // IntStream of char codes
Stream<String> splitStream = Pattern.compile(",").splitAsStream("a,b,c");

// From Optional
Optional<String> opt = Optional.of("value");
Stream<String> fromOptional = opt.stream();  // Java 9+

// Builder pattern
Stream<String> built = Stream.<String>builder()
    .add("a")
    .add("b")
    .add("c")
    .build();

// Concatenating streams
Stream<String> combined = Stream.concat(stream1, stream2);
```

### 2) Intermediate Operations

```java
List<String> names = List.of("Alice", "Bob", "Charlie", "David", "Eve");

// filter: Keep elements matching predicate
List<String> longNames = names.stream()
    .filter(name -> name.length() > 3)
    .toList();  // [Alice, Charlie, David]

// map: Transform each element
List<Integer> lengths = names.stream()
    .map(String::length)
    .toList();  // [5, 3, 7, 5, 3]

// mapToInt/Long/Double: Map to primitive stream
int totalLength = names.stream()
    .mapToInt(String::length)
    .sum();  // 23

// flatMap: Transform each element to stream, then flatten
List<String> words = List.of("Hello World", "Good Morning");
List<String> allWords = words.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .toList();  // [Hello, World, Good, Morning]

// flatMapToInt/Long/Double: flatMap to primitive stream
int[] numbers = {{1, 2}, {3, 4}, {5, 6}};
IntStream flattened = Arrays.stream(numbers)
    .flatMapToInt(Arrays::stream);

// distinct: Remove duplicates (uses equals/hashCode)
List<Integer> unique = List.of(1, 2, 2, 3, 3, 3).stream()
    .distinct()
    .toList();  // [1, 2, 3]

// sorted: Natural order
List<String> sorted = names.stream()
    .sorted()
    .toList();  // [Alice, Bob, Charlie, David, Eve]

// sorted: Custom comparator
List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparingInt(String::length))
    .toList();  // [Bob, Eve, Alice, David, Charlie]

// Reverse order
List<String> descending = names.stream()
    .sorted(Comparator.reverseOrder())
    .toList();

// limit: First n elements
List<String> firstTwo = names.stream()
    .limit(2)
    .toList();  // [Alice, Bob]

// skip: Skip first n elements
List<String> afterTwo = names.stream()
    .skip(2)
    .toList();  // [Charlie, David, Eve]

// Pagination pattern
int page = 2;
int pageSize = 10;
List<String> pageResults = allItems.stream()
    .skip((long) page * pageSize)
    .limit(pageSize)
    .toList();

// peek: Debug/inspect (don't use for side effects in production)
List<String> result = names.stream()
    .peek(n -> System.out.println("Before filter: " + n))
    .filter(n -> n.length() > 3)
    .peek(n -> System.out.println("After filter: " + n))
    .toList();

// takeWhile: Take while predicate is true (Java 9+)
List<Integer> taken = List.of(1, 2, 3, 4, 5, 1, 2).stream()
    .takeWhile(n -> n < 4)
    .toList();  // [1, 2, 3]

// dropWhile: Drop while predicate is true (Java 9+)
List<Integer> dropped = List.of(1, 2, 3, 4, 5, 1, 2).stream()
    .dropWhile(n -> n < 4)
    .toList();  // [4, 5, 1, 2]

// Chaining multiple operations
List<String> processed = names.stream()
    .filter(n -> n.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .distinct()
    .limit(3)
    .toList();
```

### 3) Terminal Operations

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

// forEach: Process each element (void return)
numbers.stream().forEach(System.out::println);

// forEachOrdered: Maintains encounter order (important for parallel)
numbers.parallelStream().forEachOrdered(System.out::println);

// collect: Mutable reduction to container
List<Integer> list = numbers.stream().collect(Collectors.toList());
Set<Integer> set = numbers.stream().collect(Collectors.toSet());

// toList(): Unmodifiable list (Java 16+)
List<Integer> immutableList = numbers.stream().filter(n -> n > 2).toList();

// toArray: Convert to array
Integer[] array = numbers.stream().toArray(Integer[]::new);
int[] primitiveArray = numbers.stream().mapToInt(i -> i).toArray();

// reduce: Combine elements
// With identity
int sum = numbers.stream().reduce(0, Integer::sum);
int product = numbers.stream().reduce(1, (a, b) -> a * b);

// Without identity (returns Optional)
Optional<Integer> max = numbers.stream().reduce(Integer::max);

// With combiner (for parallel)
int parallelSum = numbers.parallelStream()
    .reduce(0, Integer::sum, Integer::sum);

// count: Number of elements
long count = numbers.stream().filter(n -> n > 2).count();

// min/max: Smallest/largest element
Optional<Integer> min = numbers.stream().min(Comparator.naturalOrder());
Optional<Integer> maxValue = numbers.stream().max(Integer::compare);

// findFirst: First element (respects encounter order)
Optional<Integer> first = numbers.stream().filter(n -> n > 2).findFirst();

// findAny: Any matching element (may be faster for parallel)
Optional<Integer> any = numbers.parallelStream().filter(n -> n > 2).findAny();

// anyMatch: At least one matches
boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);  // true

// allMatch: All elements match
boolean allPositive = numbers.stream().allMatch(n -> n > 0);  // true

// noneMatch: No elements match
boolean noNegative = numbers.stream().noneMatch(n -> n < 0);  // true

// Primitive stream terminal operations
IntStream intStream = IntStream.of(1, 2, 3, 4, 5);
int intSum = intStream.sum();
OptionalDouble average = IntStream.of(1, 2, 3, 4, 5).average();
IntSummaryStatistics stats = IntStream.of(1, 2, 3, 4, 5).summaryStatistics();
// stats.getCount(), getSum(), getMin(), getMax(), getAverage()

// iterator: Get Iterator from stream
Iterator<Integer> iterator = numbers.stream().iterator();

// spliterator: Get Spliterator (for parallel decomposition)
Spliterator<Integer> spliterator = numbers.stream().spliterator();
```

### 4) Collectors Deep Dive

```java
import java.util.stream.Collectors;

List<Person> people = List.of(
    new Person("Alice", 30, "Engineering"),
    new Person("Bob", 25, "Sales"),
    new Person("Charlie", 35, "Engineering"),
    new Person("Diana", 28, "Marketing"),
    new Person("Eve", 32, "Sales")
);

// toList, toSet, toCollection
List<String> nameList = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

Set<String> nameSet = people.stream()
    .map(Person::getName)
    .collect(Collectors.toSet());

TreeSet<String> sortedNames = people.stream()
    .map(Person::getName)
    .collect(Collectors.toCollection(TreeSet::new));

// toMap: Key-value pairs
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(
        Person::getName,      // Key mapper
        Person::getAge        // Value mapper
    ));

// toMap with merge function (handles duplicates)
Map<String, Person> byDept = people.stream()
    .collect(Collectors.toMap(
        Person::getDept,
        p -> p,
        (existing, replacement) -> existing  // Keep first on duplicate
    ));

// toMap with specific Map implementation
Map<String, Integer> linkedMap = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge,
        (a, b) -> a,
        LinkedHashMap::new
    ));

// joining: Concatenate strings
String names = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining());  // "AliceBobCharlieDianaEve"

String namesComma = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", "));  // "Alice, Bob, Charlie, Diana, Eve"

String namesFormatted = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", ", "[", "]"));  // "[Alice, Bob, Charlie, Diana, Eve]"

// counting, summingInt, averagingInt
long count = people.stream().collect(Collectors.counting());
int totalAge = people.stream().collect(Collectors.summingInt(Person::getAge));
double avgAge = people.stream().collect(Collectors.averagingInt(Person::getAge));

// summarizingInt: All stats in one pass
IntSummaryStatistics ageStats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));
// ageStats.getCount(), getSum(), getMin(), getMax(), getAverage()

// maxBy, minBy
Optional<Person> oldest = people.stream()
    .collect(Collectors.maxBy(Comparator.comparingInt(Person::getAge)));

Optional<Person> youngest = people.stream()
    .collect(Collectors.minBy(Comparator.comparingInt(Person::getAge)));

// groupingBy: Group into Map<K, List<V>>
Map<String, List<Person>> byDepartment = people.stream()
    .collect(Collectors.groupingBy(Person::getDept));
// {Engineering=[Alice, Charlie], Sales=[Bob, Eve], Marketing=[Diana]}

// groupingBy with downstream collector
Map<String, Long> countByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDept,
        Collectors.counting()
    ));

Map<String, Double> avgAgeByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDept,
        Collectors.averagingInt(Person::getAge)
    ));

Map<String, Optional<Person>> oldestByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDept,
        Collectors.maxBy(Comparator.comparingInt(Person::getAge))
    ));

// groupingBy with Map factory
Map<String, List<Person>> sortedByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDept,
        TreeMap::new,  // Sorted keys
        Collectors.toList()
    ));

// Multi-level grouping
Map<String, Map<Integer, List<Person>>> byDeptThenAge = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDept,
        Collectors.groupingBy(Person::getAge)
    ));

// partitioningBy: Split into true/false groups
Map<Boolean, List<Person>> seniorJunior = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() >= 30));
// {true=[Alice, Charlie, Eve], false=[Bob, Diana]}

// partitioningBy with downstream
Map<Boolean, Long> seniorJuniorCount = people.stream()
    .collect(Collectors.partitioningBy(
        p -> p.getAge() >= 30,
        Collectors.counting()
    ));

// mapping: Transform before collecting
Map<String, List<String>> namesByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDept,
        Collectors.mapping(Person::getName, Collectors.toList())
    ));

// flatMapping: flatMap before collecting (Java 9+)
Map<String, Set<String>> skillsByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDept,
        Collectors.flatMapping(
            p -> p.getSkills().stream(),
            Collectors.toSet()
        )
    ));

// filtering: Filter before collecting (Java 9+)
Map<String, List<Person>> seniorsByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDept,
        Collectors.filtering(p -> p.getAge() >= 30, Collectors.toList())
    ));

// collectingAndThen: Post-process collected result
List<Person> unmodifiablePeople = people.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));

// reducing: Similar to Stream.reduce
Optional<Person> oldestPerson = people.stream()
    .collect(Collectors.reducing(
        BinaryOperator.maxBy(Comparator.comparingInt(Person::getAge))
    ));

// teeing: Two collectors, combine results (Java 12+)
record AgeRange(int min, int max) {}
AgeRange range = people.stream()
    .collect(Collectors.teeing(
        Collectors.minBy(Comparator.comparingInt(Person::getAge)),
        Collectors.maxBy(Comparator.comparingInt(Person::getAge)),
        (minOpt, maxOpt) -> new AgeRange(
            minOpt.map(Person::getAge).orElse(0),
            maxOpt.map(Person::getAge).orElse(0)
        )
    ));
```

### 5) Custom Collector

```java
// Collector interface: <T, A, R>
// T = input type, A = accumulator type, R = result type

// Simple custom collector: Join to string with custom delimiter
Collector<String, StringBuilder, String> customJoiner = Collector.of(
    StringBuilder::new,                    // Supplier: Create accumulator
    (sb, s) -> {                          // Accumulator: Add element
        if (sb.length() > 0) sb.append("|");
        sb.append(s);
    },
    (sb1, sb2) -> {                       // Combiner: Merge accumulators (parallel)
        if (sb1.length() > 0) sb1.append("|");
        sb1.append(sb2);
        return sb1;
    },
    StringBuilder::toString               // Finisher: Final transformation
);

String result = Stream.of("a", "b", "c")
    .collect(customJoiner);  // "a|b|c"

// Collector with characteristics
Collector<Integer, List<Integer>, List<Integer>> toUnmodifiableList = Collector.of(
    ArrayList::new,
    List::add,
    (left, right) -> { left.addAll(right); return left; },
    Collections::unmodifiableList,
    Collector.Characteristics.UNORDERED   // Characteristics
);

// Available characteristics:
// CONCURRENT - accumulator can be modified concurrently
// UNORDERED - collection doesn't preserve encounter order
// IDENTITY_FINISH - finisher is identity function (can be omitted)
```

### 6) Parallel Streams

```java
// Creating parallel streams
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8);

// From collection
Stream<Integer> parallel1 = numbers.parallelStream();

// Convert sequential to parallel
Stream<Integer> parallel2 = numbers.stream().parallel();

// Convert parallel to sequential
Stream<Integer> sequential = numbers.parallelStream().sequential();

// Check if parallel
boolean isParallel = numbers.parallelStream().isParallel();  // true

// Parallel processing
long count = numbers.parallelStream()
    .filter(n -> expensiveComputation(n))
    .count();

// When parallel streams help:
// ✅ Large datasets (thousands of elements)
// ✅ CPU-bound operations
// ✅ Stateless, independent operations
// ✅ Easy-to-split sources (ArrayList, arrays)

// When to avoid parallel streams:
// ❌ Small datasets (overhead > benefit)
// ❌ I/O-bound operations (network, file)
// ❌ Operations with side effects
// ❌ Stateful operations (sorted, distinct)
// ❌ Linked data structures (LinkedList)
// ❌ Order-dependent operations

// Custom thread pool for parallel streams
ForkJoinPool customPool = new ForkJoinPool(4);
try {
    List<Integer> result = customPool.submit(() ->
        numbers.parallelStream()
            .map(this::expensiveOperation)
            .toList()
    ).get();
} finally {
    customPool.shutdown();
}

// Ordered vs unordered in parallel
// Ordered parallel (maintains encounter order - slower)
numbers.parallelStream()
    .filter(n -> n > 2)
    .findFirst();  // Deterministic: 3

// Unordered parallel (any matching element - faster)
numbers.parallelStream()
    .unordered()
    .filter(n -> n > 2)
    .findAny();  // Non-deterministic: 3, 4, 5, 6, 7, or 8

// forEachOrdered: Maintains order in parallel
numbers.parallelStream()
    .forEachOrdered(System.out::println);  // 1, 2, 3, 4, 5, 6, 7, 8

// Reduction in parallel - associativity matters!
// ✅ Associative operations (safe for parallel)
int sum = numbers.parallelStream().reduce(0, Integer::sum);

// ❌ Non-associative operations (unsafe!)
// Subtraction is not associative: (a-b)-c ≠ a-(b-c)
// Don't use: numbers.parallelStream().reduce(0, (a, b) -> a - b);
```

### 7) Primitive Streams

```java
// IntStream
IntStream intStream = IntStream.range(1, 6);  // 1, 2, 3, 4, 5
IntStream intStreamClosed = IntStream.rangeClosed(1, 5);  // 1, 2, 3, 4, 5
IntStream fromArray = Arrays.stream(new int[]{1, 2, 3});

// IntStream operations
int sum = IntStream.of(1, 2, 3, 4, 5).sum();
OptionalDouble avg = IntStream.of(1, 2, 3, 4, 5).average();
OptionalInt max = IntStream.of(1, 2, 3, 4, 5).max();
OptionalInt min = IntStream.of(1, 2, 3, 4, 5).min();
long count = IntStream.of(1, 2, 3, 4, 5).count();

IntSummaryStatistics stats = IntStream.of(1, 2, 3, 4, 5).summaryStatistics();
// stats.getSum(), getAverage(), getMin(), getMax(), getCount()

// Boxing/Unboxing
Stream<Integer> boxed = IntStream.of(1, 2, 3).boxed();
IntStream unboxed = Stream.of(1, 2, 3).mapToInt(Integer::intValue);

// mapToObj: IntStream to Stream<T>
Stream<String> strings = IntStream.range(1, 6)
    .mapToObj(i -> "Number: " + i);

// LongStream
LongStream longStream = LongStream.rangeClosed(1, 1_000_000_000);
long longSum = longStream.sum();

// DoubleStream
DoubleStream doubleStream = DoubleStream.of(1.0, 2.0, 3.0);
double doubleSum = doubleStream.sum();

// Convert between primitive streams
IntStream ints = IntStream.of(1, 2, 3);
LongStream longs = ints.asLongStream();
DoubleStream doubles = IntStream.of(1, 2, 3).asDoubleStream();

// Useful patterns
// Character frequency
"hello".chars()  // IntStream of char codes
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(c -> c, Collectors.counting()));

// Generate index-value pairs
String[] array = {"a", "b", "c"};
IntStream.range(0, array.length)
    .mapToObj(i -> i + ": " + array[i])
    .forEach(System.out::println);
// 0: a, 1: b, 2: c
```

### 8) Common Patterns and Use Cases

```java
// Find first matching element
Optional<Person> alice = people.stream()
    .filter(p -> p.getName().equals("Alice"))
    .findFirst();

// Check if any/all/none match
boolean hasAdult = people.stream().anyMatch(p -> p.getAge() >= 18);
boolean allAdults = people.stream().allMatch(p -> p.getAge() >= 18);
boolean noMinors = people.stream().noneMatch(p -> p.getAge() < 18);

// Get max/min by property
Optional<Person> oldest = people.stream()
    .max(Comparator.comparingInt(Person::getAge));

// Top N elements
List<Person> top3Oldest = people.stream()
    .sorted(Comparator.comparingInt(Person::getAge).reversed())
    .limit(3)
    .toList();

// Distinct by property
List<Person> uniqueByDept = people.stream()
    .collect(Collectors.toMap(
        Person::getDept,
        p -> p,
        (p1, p2) -> p1  // Keep first
    ))
    .values()
    .stream()
    .toList();

// Or using filter with seen set
Set<String> seen = new HashSet<>();
List<Person> uniqueByDept2 = people.stream()
    .filter(p -> seen.add(p.getDept()))  // Warning: stateful!
    .toList();

// Flatten nested collections
List<List<String>> nested = List.of(
    List.of("a", "b"),
    List.of("c", "d"),
    List.of("e", "f")
);
List<String> flat = nested.stream()
    .flatMap(Collection::stream)
    .toList();  // [a, b, c, d, e, f]

// Zip two lists (no built-in, use IntStream)
List<String> list1 = List.of("a", "b", "c");
List<Integer> list2 = List.of(1, 2, 3);
List<String> zipped = IntStream.range(0, Math.min(list1.size(), list2.size()))
    .mapToObj(i -> list1.get(i) + list2.get(i))
    .toList();  // [a1, b2, c3]

// Batch processing
int batchSize = 10;
List<List<Integer>> batches = IntStream.range(0, (numbers.size() + batchSize - 1) / batchSize)
    .mapToObj(i -> numbers.subList(
        i * batchSize,
        Math.min((i + 1) * batchSize, numbers.size())
    ))
    .toList();

// Running total (cumulative sum)
List<Integer> runningTotal = new ArrayList<>();
numbers.stream()
    .reduce(0, (acc, n) -> {
        int sum = acc + n;
        runningTotal.add(sum);
        return sum;
    });

// Map with index
List<String> indexed = IntStream.range(0, names.size())
    .mapToObj(i -> i + ": " + names.get(i))
    .toList();

// Null-safe stream
List<String> nullableList = null;
Stream<String> safeStream = Optional.ofNullable(nullableList)
    .map(Collection::stream)
    .orElse(Stream.empty());

// Or in Java 9+
Stream<String> safeStream2 = Optional.ofNullable(nullableList)
    .stream()
    .flatMap(Collection::stream);

// Handling exceptions
List<String> results = paths.stream()
    .map(path -> {
        try {
            return Files.readString(path);
        } catch (IOException e) {
            return "ERROR: " + e.getMessage();
        }
    })
    .toList();
```

### 9) Performance Considerations

```java
// ❌ Creating unnecessary intermediate collections
List<String> bad = list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList())  // Unnecessary intermediate list
    .stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// ✅ Chain operations in single pipeline
List<String> good = list.stream()
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// ❌ Using boxed streams for primitives
int bad = Stream.of(1, 2, 3, 4, 5)
    .reduce(0, Integer::sum);  // Autoboxing overhead

// ✅ Using primitive streams
int good = IntStream.of(1, 2, 3, 4, 5).sum();

// ❌ Sorting large stream then taking few elements
list.stream()
    .sorted()  // Sorts entire list
    .limit(3)
    .toList();

// ✅ Use min/max or partial sort structures
list.stream()
    .collect(Collectors.toCollection(
        () -> new TreeSet<>(Comparator.naturalOrder())
    ))
    .stream()
    .limit(3)
    .toList();

// Or use PriorityQueue for top-k
PriorityQueue<String> topK = list.stream()
    .collect(Collectors.toCollection(() -> 
        new PriorityQueue<>(3, Comparator.naturalOrder())
    ));

// ❌ count() after filter in Java 8
long count = list.stream().filter(predicate).collect(Collectors.counting());

// ✅ Direct count()
long count = list.stream().filter(predicate).count();

// Stream source performance
// Fast random access: ArrayList, arrays - good for parallel
// Sequential access: LinkedList - poor for parallel, avoid
// Infinite/lazy: Stream.iterate, generate - good for on-demand
```

---

## Common Pitfalls

### 1. Reusing Streams

```java
// ❌ Streams can only be used once
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
stream.forEach(System.out::println);  // IllegalStateException!

// ✅ Create new stream for each use
list.stream().forEach(System.out::println);
list.stream().forEach(System.out::println);

// ✅ Or use Supplier
Supplier<Stream<String>> streamSupplier = list::stream;
streamSupplier.get().forEach(System.out::println);
streamSupplier.get().forEach(System.out::println);
```

### 2. Side Effects in peek()

```java
// ❌ Don't use peek for side effects
List<String> collected = new ArrayList<>();
list.stream()
    .peek(collected::add)  // Bad: side effect
    .filter(s -> s.length() > 3)
    .toList();

// ✅ Use proper terminal operations
List<String> collected = list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());
```

### 3. sorted() with Parallel Streams

```java
// Caution: sorted() is a stateful intermediate operation
// It needs to see all elements before producing output
// This can eliminate parallel benefits
list.parallelStream()
    .sorted()  // Bottleneck: synchronization point
    .map(expensive)
    .toList();
```

### 4. toMap() with Duplicate Keys

```java
// ❌ Throws IllegalStateException if duplicate keys
Map<String, Person> map = people.stream()
    .collect(Collectors.toMap(Person::getDept, p -> p));

// ✅ Provide merge function
Map<String, Person> map = people.stream()
    .collect(Collectors.toMap(
        Person::getDept,
        p -> p,
        (existing, replacement) -> existing  // Keep first
    ));
```

### 5. Infinite Streams Without limit()

```java
// ❌ This will never terminate
Stream.iterate(0, n -> n + 1).forEach(System.out::println);

// ✅ Always limit infinite streams
Stream.iterate(0, n -> n + 1).limit(10).forEach(System.out::println);

// ✅ Or use bounded iterate (Java 9+)
Stream.iterate(0, n -> n < 10, n -> n + 1).forEach(System.out::println);
```

### 6. Resource Leaks with I/O Streams

```java
// ❌ Resource leak: stream not closed
Files.lines(path).forEach(System.out::println);

// ✅ Use try-with-resources
try (Stream<String> lines = Files.lines(path)) {
    lines.forEach(System.out::println);
}
```

---

## Interview Questions

### Basic Questions

1. **What is the difference between intermediate and terminal operations?**
   - Intermediate: Lazy, returns Stream, can be chained (map, filter, sorted)
   - Terminal: Eager, triggers execution, returns result/void (collect, forEach, count)

2. **Why are streams single-use?**
   - Stream pipeline maintains internal state
   - Terminal operation consumes the stream
   - Prevents undefined behavior from reuse

3. **What is the difference between map() and flatMap()?**
   - `map`: One-to-one transformation (T → R)
   - `flatMap`: One-to-many transformation, then flattens (T → Stream<R> → R)

4. **How do you handle checked exceptions in streams?**
   - Wrap in try-catch inside lambda
   - Create wrapper utility method
   - Use sneaky throws (not recommended)

5. **What is short-circuiting in streams?**
   - Operations that can terminate without processing all elements
   - Examples: `findFirst`, `findAny`, `anyMatch`, `allMatch`, `limit`

### Intermediate Questions

6. **What is the difference between Stream.toList() and Collectors.toList()?**
   - `toList()` (Java 16+): Returns unmodifiable list
   - `Collectors.toList()`: No guarantee about mutability (usually ArrayList)

7. **Explain groupingBy vs partitioningBy.**
   - `groupingBy`: Groups into Map<K, List<V>> by classifier function
   - `partitioningBy`: Special case, splits into Map<Boolean, List<V>>

8. **When should you avoid parallel streams?**
   - Small datasets (overhead exceeds benefit)
   - I/O-bound operations (network, file)
   - Operations with side effects
   - LinkedList sources (poor splitting)
   - Order-dependent operations

9. **How does reduce() work with parallel streams?**
   - Identity value is used for each partition
   - Combiner merges partial results
   - Operation must be associative for correct results

10. **What is the difference between findFirst() and findAny()?**
    - `findFirst`: Returns first element in encounter order (deterministic)
    - `findAny`: Returns any element (may be faster in parallel, non-deterministic)

### Advanced Questions

11. **How would you implement distinct() by a property?**
    ```java
    people.stream()
        .collect(Collectors.toMap(
            Person::getDept,
            p -> p,
            (p1, p2) -> p1
        ))
        .values()
        .stream()
        .toList();
    ```

12. **Explain the Collector interface components.**
    - Supplier: Creates accumulator container
    - Accumulator: Adds element to container
    - Combiner: Merges two containers (for parallel)
    - Finisher: Final transformation
    - Characteristics: Hints for optimization

13. **Why must reduce operations be associative for parallel streams?**
    - Parallel splits work into chunks
    - Each chunk produces partial result
    - Combiner merges results in arbitrary order
    - Non-associative: (a op b) op c ≠ a op (b op c) → incorrect results

14. **How do parallel streams use threads?**
    - Uses ForkJoinPool.commonPool() by default
    - Pool size = Runtime.availableProcessors() - 1
    - Can use custom ForkJoinPool

15. **What is the difference between forEach and forEachOrdered?**
    - `forEach`: No order guarantee (parallel may be faster)
    - `forEachOrdered`: Maintains encounter order (serial processing)

---

## Quick Reference

### Operation Classification

| Type | Operations |
|------|-----------|
| **Intermediate (Lazy)** | filter, map, flatMap, distinct, sorted, peek, limit, skip, takeWhile, dropWhile |
| **Terminal (Eager)** | forEach, forEachOrdered, toArray, reduce, collect, min, max, count, anyMatch, allMatch, noneMatch, findFirst, findAny |
| **Short-circuiting** | limit, findFirst, findAny, anyMatch, allMatch, noneMatch |
| **Stateful** | distinct, sorted, limit, skip |
| **Stateless** | filter, map, flatMap, peek |

### Common Collectors

| Collector | Result |
|-----------|--------|
| `toList()` | List<T> |
| `toSet()` | Set<T> |
| `toMap(k, v)` | Map<K, V> |
| `joining(delim)` | String |
| `counting()` | Long |
| `summingInt(f)` | Integer |
| `averagingInt(f)` | Double |
| `groupingBy(f)` | Map<K, List<V>> |
| `partitioningBy(p)` | Map<Boolean, List<V>> |
| `maxBy(cmp)` | Optional<T> |
| `minBy(cmp)` | Optional<T> |
| `mapping(f, d)` | Apply mapper then downstream |
| `filtering(p, d)` | Apply filter then downstream |
| `collectingAndThen(c, f)` | Collect then transform |
| `teeing(c1, c2, f)` | Two collectors, merge results |

### Primitive Stream Methods

| Method | IntStream | LongStream | DoubleStream |
|--------|-----------|------------|--------------|
| Range | `range(a, b)` | `range(a, b)` | - |
| Statistics | `summaryStatistics()` | `summaryStatistics()` | `summaryStatistics()` |
| Sum | `sum()` | `sum()` | `sum()` |
| Average | `average()` | `average()` | `average()` |
| Box | `boxed()` | `boxed()` | `boxed()` |
| To Object | `mapToObj(f)` | `mapToObj(f)` | `mapToObj(f)` |
