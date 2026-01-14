# Collections & Generics (Deep Interview Coverage)

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

## 10) Classic interview questions

- Explain how `HashMap` works internally. What happens on collisions?
- Why must you override `hashCode` when overriding `equals`?
- Difference between `HashMap`, `LinkedHashMap`, `TreeMap`?
- Why is `List<Integer>` not a `List<Number>`? Explain invariance.
- Explain PECS with examples.
- What is type erasure? What limitations does it introduce?

