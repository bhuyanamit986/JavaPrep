# 5️⃣ Collections Framework (Most Asked Topic)

Collections are asked in almost every Java interview because they test:

- big-O reasoning
- equality + hashing correctness
- API knowledge (List/Set/Map/Queue)
- internal workings (HashMap resize/treeify, fail-fast)
- concurrency awareness (`ConcurrentHashMap`, `CopyOnWriteArrayList`)

---

## 5.1 Core Interfaces

### Collection vs Collections

- **`Collection`**: interface (root of List/Set/Queue hierarchies)
- **`Collections`**: utility class (`sort`, `unmodifiableList`, `singletonList`, etc.)

### List, Set, Queue, Deque, Map

- **`List<E>`**: ordered, allows duplicates, positional access
- **`Set<E>`**: no duplicates (as defined by `equals`/`hashCode`)
- **`Queue<E>`**: typically FIFO; supports `offer/poll/peek`
- **`Deque<E>`**: double-ended queue; can be used as stack or queue
- **`Map<K,V>`**: key→value mapping (not a `Collection`)

Quick “choose what”:

- Need order + duplicates → `List`
- Need uniqueness → `Set`
- Need key-based lookup → `Map`
- Need scheduling/producer-consumer → `Queue`/`Deque`

### Iterator vs ListIterator

- `Iterator<E>`:
  - forward-only traversal
  - supports `remove()` during iteration
- `ListIterator<E>` (only for lists):
  - forward + backward traversal
  - supports `add`, `set`, `previousIndex`, etc.

### Fail-fast vs fail-safe

#### Fail-fast

- Most standard collections (`ArrayList`, `HashMap`) have fail-fast iterators.
- If the collection is structurally modified while iterating (outside iterator methods), you may get `ConcurrentModificationException`.

Key nuance: fail-fast is best-effort and not a synchronization mechanism.

#### Fail-safe

- Iterators that iterate over a snapshot or are designed for concurrency.
- Example: `CopyOnWriteArrayList` iterator (snapshot).

---

## 5.2 List Implementations

### ArrayList

- Backed by resizable array
- `get(i)` is O(1)
- append amortized O(1)
- insert/remove in middle O(n) due to shifting

### LinkedList

- doubly-linked list
- `get(i)` is O(n)
- insert/remove at ends O(1)
- often slower than ArrayList for general use (poor locality + overhead)

### Vector (legacy)

- Like ArrayList but synchronized
- Usually avoided; prefer `ArrayList` + external synchronization if needed

### CopyOnWriteArrayList

- Thread-safe list optimized for **many reads, few writes**
- On mutation, it copies the entire underlying array
- Iteration is snapshot-based (fail-safe)

When to use:

- mostly-read configuration lists, listeners, etc.

When not to use:

- frequent writes (high allocation/copy cost)

### Performance comparison (intuition)

```
Operation          ArrayList         LinkedList
--------------------------------------------------------
Random access      O(1)              O(n)
Append            amortized O(1)     O(1)
Insert middle     O(n)              O(n) (need traversal)
Memory overhead   low               higher (node objects)
```

---

## 5.3 Set Implementations

### HashSet

- Backed by HashMap
- expected O(1) add/contains/remove
- depends on correct `equals` + `hashCode`

### LinkedHashSet

- Like HashSet but preserves insertion order (predictable iteration)

### TreeSet

- Sorted set backed by TreeMap (Red-Black tree)
- O(log n) operations

### Comparator vs Comparable

- `Comparable<T>`: natural order via `compareTo`
- `Comparator<T>`: external order passed into sorting/TreeSet/TreeMap

Very important:

- In sorted sets/maps, **comparator defines equality**.
  - If `compare(a,b) == 0`, the structure treats them as “duplicates” even if `!a.equals(b)`.

### equals() & hashCode() importance

If `equals` is overridden, `hashCode` must also be overridden.

Typical bug:

- Put mutable key in a HashSet/HashMap
- mutate fields used in hashCode
- lookup fails because bucket changes

---

## 5.4 Map Implementations

### HashMap (internal working, resizing, treeification)

#### High-level algorithm (put)

1. compute hash for key
2. find bucket index
3. if bucket empty: insert
4. if bucket has entries: compare keys with `equals`
   - if match: replace value
   - else: add to bucket chain

Illustration:

```
table[bucket] -> Entry(k1,v1) -> Entry(k2,v2) -> ...
```

#### Resizing

- HashMap grows when `size > capacity * loadFactor`.
- resizing rehashes entries into a larger table.

Interview signal: mention that resize is expensive (rehash) but amortized across operations.

#### Treeification (Java 8+)

If a bucket becomes too large (many collisions), it can convert from a linked list to a balanced tree to improve worst-case.

Why this exists:

- protects against pathological collision scenarios (performance attacks, poor hash functions)

### LinkedHashMap

- Predictable iteration order (insertion order or access order)
- Often used to build LRU cache behavior (access order + eviction)

### TreeMap

- Sorted map (by key)
- Red-Black tree
- O(log n) get/put/remove

### ConcurrentHashMap (brief here; details in concurrency chapter)

- Designed for concurrent access with high throughput
- Avoids a single global lock (internals changed across Java versions)

Rule: prefer `ConcurrentHashMap` over synchronizing a `HashMap` in concurrent code.

### Hashtable (legacy)

- synchronized, legacy map
- generally replaced by `ConcurrentHashMap` or `Collections.synchronizedMap(new HashMap<>())` depending on needs

---

## High-frequency interview Q&A

- Explain how HashMap handles collisions. What triggers resize?
- Why do we need equals/hashCode for HashSet/HashMap?
- Difference between ArrayList and LinkedList? When would you use each?
- What is fail-fast and why is `ConcurrentModificationException` not a concurrency guarantee?
- Why is `CopyOnWriteArrayList` good for readers but bad for writers?
- Difference between HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap?

---

## Extra references

- Generics deep dive: `docs/collections-generics.md`
- Streams collectors (`groupingBy`, `toMap`): `docs/streams.md`

