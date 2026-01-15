# Collections & Generics (Deep Interview Coverage)

## Definitions

- **Collection**: a container of elements with a common API for add/remove/iterate.
- **List**: ordered collection that allows duplicates and index-based access.
- **Set**: collection that enforces uniqueness based on `equals()` and `hashCode()`.
- **Map**: key-to-value structure; keys are unique and not part of `Collection`.
- **Big-O**: how performance grows as data size grows (e.g., `ArrayList` get is O(1)).
- **Generics**: compile-time type safety for containers and APIs.
- **Variance**: how subtyping works with generics (`extends` for producer, `super` for consumer).
- **Type erasure**: generic type info is removed at runtime; only compile-time checks remain.

## Illustrations

- **List**: an ordered notebook where each page number is an index.
- **Set**: a stamp collection where duplicates are rejected.
- **Map**: a phone book where each name maps to a phone number.
- **Wildcards**: `List<? extends Number>` is like a read-only view of numbers; you can read, not safely write.

## Code Examples

```java
static double sumNumbers(java.util.List<? extends Number> nums) {
    double sum = 0;
    for (Number n : nums) sum += n.doubleValue();
    return sum;
}

static void addIntegers(java.util.List<? super Integer> dest) {
    dest.add(1);
    dest.add(2);
}
```

## Interview Questions

1. When do you use `ArrayList` vs `LinkedList`?
2. Why must `equals()` and `hashCode()` be consistent for `HashSet`?
3. Explain `List<? extends T>` vs `List<? super T>`.
4. What is type erasure and why does it matter?
5. What is the difference between `HashMap` and `TreeMap`?
6. What does "fail-fast iterator" mean?

---

## 1) Collections framework overview

Core interfaces:

- `Collection<E>`
  - `List<E>`: ordered, allows duplicates
  - `Set<E>`: no duplicates (as defined by equality)
  - `Queue<E>` / `Deque<E>`: FIFO / double-ended
- `Map<K,V>`: key→value mapping (not a `Collection`)

Choosing the right one is frequently tested; focus on **semantics** and **complexity**.

## 2) `List` implementations

### `ArrayList`

- Backed by a resizable array.
- Random access: **O(1)**
- Append amortized **O(1)** (occasional resize)
- Insert/remove in middle: **O(n)** (shifts elements)

### `LinkedList`

- Doubly-linked list.
- Random access: **O(n)**
- Insert/remove at ends: **O(1)**
- As a general-purpose list, often worse than `ArrayList` due to cache locality and overhead.

Interview tip: default to `ArrayList` unless you have a clear reason.

## 3) `Set` implementations

### `HashSet`

- Backed by `HashMap`.
- Expected **O(1)** add/contains/remove.
- Requires correct `equals`/`hashCode`.

### `LinkedHashSet`

- Preserves insertion order (iteration order is predictable).

### `TreeSet`

- Sorted set backed by `TreeMap` (Red-Black tree).
- Operations **O(log n)**.
- Requires consistent comparator ordering; comparator defines equality for the set.

## 4) `Map` implementations

### `HashMap`

- Expected **O(1)** for get/put/remove.
- Since Java 8, bins can treeify under high collision (improves worst-case).

Important behaviors:

- Allows one `null` key and multiple `null` values.
- Iteration order is not guaranteed.

### `LinkedHashMap`

- Predictable iteration order: insertion or access order (LRU-like).
- Interview classic: implement LRU cache using `LinkedHashMap`.

### `TreeMap`

- Sorted by key (natural ordering or comparator).
- **O(log n)** operations.

### `ConcurrentHashMap` (preview)

- Thread-safe map designed for high concurrency (details in concurrency chapter).

## 5) Equality, hashing, and buckets (must know)

When `put(k, v)` in a `HashMap`:

- Compute `hash(k)` → bucket index.
- If bucket empty: store entry.
- Else compare existing keys using `equals` to find match and replace/update.

If `hashCode()` changes after insertion, the key becomes “lost”.

## 6) Fail-fast iterators and `ConcurrentModificationException`

Many collections are **fail-fast**:

- If you structurally modify a collection while iterating (outside iterator methods), you may get `ConcurrentModificationException`.
- This is a best-effort detection; it’s not guaranteed for correctness.

Safe approaches:

- Use iterator’s `remove()`.
- Iterate over a snapshot copy.
- Use concurrent collections (when appropriate).

## 7) Immutability vs unmodifiable vs immutable

- **Immutable**: cannot change state (by design).
- **Unmodifiable view**: wrapper that throws on modification, but underlying collection may still change elsewhere.

Example: `Collections.unmodifiableList(list)` is a view; if `list` changes, the view reflects it.

## 8) Generics fundamentals

### Why generics exist

- Compile-time type safety
- Avoid casts
- Self-documenting APIs

### Invariance

In Java, generics are **invariant**:

- `List<Integer>` is not a subtype of `List<Number>`.

### Wildcards: PECS

**PECS**: Producer Extends, Consumer Super

- `? extends T`: you can read `T` (producer), but cannot safely add (except `null`)
- `? super T`: you can add `T` (consumer), but reads come out as `Object`

Examples:

```java
void readNumbers(List<? extends Number> xs) {
  Number n = xs.get(0);
  // xs.add(1); // not allowed (could be List<Double>)
}

void addIntegers(List<? super Integer> xs) {
  xs.add(1);
  Object o = xs.get(0);
}
```

### Type erasure (big interview topic)

- Generic type parameters are erased at runtime.
- You can’t do `new T()` or `new List<String>()` directly.
- Overloads that differ only by generic type are illegal due to erasure.

### Bounded type parameters

```java
static <T extends Comparable<? super T>> T max(List<T> xs) { ... }
```

Interview signal: understanding bounds and why `Comparable<? super T>` is used (handles comparable declared on supertypes).

## 9) `Collections` vs `Collectors`

- `java.util.Collections`: utilities for **collections** (sort, unmodifiable wrappers, etc.)
- `java.util.stream.Collectors`: utilities for **stream reduction** (toList, groupingBy, joining, etc.)

## 10) HashMap Internal Working (Deep Dive)

### Structure

```
HashMap = Array of Buckets (Nodes/TreeNodes)
          [0] → Node → Node → Node (linked list)
          [1] → null
          [2] → TreeNode (tree if > 8 elements)
          ...
          [n-1]
```

### put() Operation

```java
// Simplified logic
int hash = hash(key);           // Spread bits
int index = hash & (n - 1);     // Bucket index

if (bucket[index] == null) {
    bucket[index] = new Node(key, value);
} else {
    // Traverse list/tree to find key
    // If found: update value
    // If not found: add new node
}

// If size > threshold (capacity * loadFactor):
//   resize (double capacity, rehash all)
```

### Treeification (Java 8+)

- When bucket has > 8 entries: convert to Red-Black tree (O(log n))
- When bucket has < 6 entries: convert back to linked list

### Thread Safety Note

```java
// HashMap is NOT thread-safe
// Use ConcurrentHashMap or Collections.synchronizedMap()
Map<K, V> syncMap = Collections.synchronizedMap(new HashMap<>());
```

---

## 11) Implementing Comparable and Comparator

```java
public class Employee implements Comparable<Employee> {
    private String name;
    private int salary;
    private LocalDate hireDate;
    
    // Natural ordering by name
    @Override
    public int compareTo(Employee other) {
        return this.name.compareTo(other.name);
    }
}

// External comparators
Comparator<Employee> bySalary = Comparator.comparingInt(Employee::getSalary);

Comparator<Employee> bySalaryDesc = bySalary.reversed();

Comparator<Employee> byHireDate = Comparator.comparing(Employee::getHireDate);

// Chained comparison
Comparator<Employee> bySalaryThenName = Comparator
    .comparingInt(Employee::getSalary)
    .thenComparing(Employee::getName);

// Null-safe
Comparator<Employee> nullSafe = Comparator
    .nullsLast(Comparator.comparing(Employee::getName));
```

---

## 12) Useful Collection Operations

```java
// Creating immutable collections (Java 9+)
List<String> list = List.of("a", "b", "c");
Set<String> set = Set.of("a", "b", "c");
Map<String, Integer> map = Map.of("a", 1, "b", 2);
Map<String, Integer> mapEntries = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2)
);

// Collections utilities
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
Collections.binarySearch(sortedList, key);
Collections.frequency(collection, element);
Collections.disjoint(c1, c2);  // true if no common elements
Collections.min(collection);
Collections.max(collection);
Collections.unmodifiableList(list);  // Read-only view
Collections.synchronizedList(list);  // Thread-safe wrapper

// Arrays utilities
Arrays.sort(array);
Arrays.binarySearch(sortedArray, key);
Arrays.fill(array, value);
Arrays.copyOf(array, newLength);
Arrays.equals(arr1, arr2);
Arrays.asList(array);  // Fixed-size list backed by array
```

---

## 13) Advanced Generic Patterns

### Generic Methods

```java
public static <T> T firstNonNull(T first, T second) {
    return first != null ? first : second;
}

public static <T extends Comparable<? super T>> T max(List<T> list) {
    return Collections.max(list);
}
```

### Generic Classes

```java
public class Pair<K, V> {
    private final K key;
    private final V value;
    
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    public K getKey() { return key; }
    public V getValue() { return value; }
}
```

### Type Tokens

```java
// Can't do: new T() or T.class due to erasure
// Solution: Pass Class<T> as parameter
public <T> T create(Class<T> type) throws Exception {
    return type.getDeclaredConstructor().newInstance();
}

// Usage
String s = create(String.class);
```

---

## 14) Classic interview questions

- Explain how `HashMap` works internally. What happens on collisions?
- Why must you override `hashCode` when overriding `equals`?
- Difference between `HashMap`, `LinkedHashMap`, `TreeMap`?
- Why is `List<Integer>` not a `List<Number>`? Explain invariance.
- Explain PECS with examples.
- What is type erasure? What limitations does it introduce?
- How does ConcurrentHashMap achieve thread-safety?
- What is the difference between fail-fast and fail-safe iterators?
- How would you implement an LRU cache?
- Why can't you create a generic array like `new T[]`?
- What happens if two objects have the same hashCode but are not equal?
- When would you use TreeMap over HashMap?

