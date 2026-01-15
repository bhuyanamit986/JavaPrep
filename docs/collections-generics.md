# Collections & Generics

This chapter covers the Java Collections Framework and Generics - essential knowledge for any Java developer. Understanding these concepts is crucial for writing efficient code and succeeding in technical interviews.

---

## Definitions

### Collection Interfaces

- **Collection**: Root interface for most collection types. Defines basic operations like `add()`, `remove()`, `contains()`, `size()`, `iterator()`.

- **List**: Ordered collection (sequence) that allows duplicates. Elements accessible by index. Implementations: ArrayList, LinkedList, Vector.

- **Set**: Collection that contains no duplicate elements. Implementations: HashSet, LinkedHashSet, TreeSet.

- **Queue**: Collection designed for holding elements prior to processing. Typically FIFO order. Implementations: LinkedList, PriorityQueue, ArrayDeque.

- **Deque**: Double-ended queue that supports insertion and removal at both ends. Implementations: ArrayDeque, LinkedList.

- **Map**: Object that maps keys to values. Not a true Collection (doesn't extend Collection interface). Implementations: HashMap, LinkedHashMap, TreeMap, Hashtable.

### Generics Concepts

- **Generics**: Feature that allows type parameterization, enabling type-safe collections and code reuse.

- **Type Erasure**: Process where generic type information is removed at compile time, replaced with bounds or Object.

- **Invariance**: `List<Dog>` is NOT a subtype of `List<Animal>`. Default behavior of generics.

- **Covariance**: Upper-bounded wildcard `List<? extends Animal>` - can read as Animal, cannot add.

- **Contravariance**: Lower-bounded wildcard `List<? super Dog>` - can add Dog, reads as Object.

- **PECS**: Producer Extends, Consumer Super - guideline for wildcard usage.

- **Raw Type**: Generic type used without type parameter. Avoid - loses type safety.

---

## Illustrations

### Collection Interface Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Java Collections Framework                            │
│                                                                          │
│                         ┌─────────────┐                                  │
│                         │  Iterable   │                                  │
│                         └──────┬──────┘                                  │
│                                │                                         │
│                         ┌──────▼──────┐                                  │
│                         │ Collection  │                                  │
│                         └──────┬──────┘                                  │
│              ┌─────────────────┼─────────────────┐                       │
│              │                 │                 │                       │
│       ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐               │
│       │    List     │   │    Set      │   │   Queue     │               │
│       │  (ordered,  │   │ (unique     │   │  (FIFO/     │               │
│       │  duplicates)│   │  elements)  │   │   priority) │               │
│       └──────┬──────┘   └──────┬──────┘   └──────┬──────┘               │
│              │                 │                 │                       │
│    ┌─────────┼────────┐       │           ┌─────┴─────┐                 │
│    │         │        │       │           │           │                 │
│    ▼         ▼        ▼       ▼           ▼           ▼                 │
│ ArrayList LinkedList Vector HashSet   PriorityQueue ArrayDeque         │
│                              LinkedHashSet           (also Deque)       │
│                              TreeSet (SortedSet)                        │
│                                                                          │
│                         ┌─────────────┐                                  │
│                         │    Map      │  (NOT a Collection!)            │
│                         └──────┬──────┘                                  │
│              ┌─────────────────┼─────────────────┐                       │
│              │                 │                 │                       │
│           HashMap       LinkedHashMap        TreeMap                    │
│           ConcurrentHashMap                  (SortedMap)                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### ArrayList vs LinkedList Internal Structure

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ArrayList (Dynamic Array)                             │
│                                                                          │
│   Backed by resizable array                                             │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │ Index: │  0  │  1  │  2  │  3  │  4  │  5  │  6  │  7  │ ... │     │
│   │        ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────│     │
│   │ Data:  │  A  │  B  │  C  │  D  │  E  │null │null │null │ ... │     │
│   │        └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘     │
│   │                                                                    │
│   │   size = 5, capacity = 10 (array length)                          │
│   │                                                                    │
│   │   ✓ Fast random access: O(1) - direct index access                │
│   │   ✗ Slow insert/delete in middle: O(n) - must shift elements      │
│   │   ✓ Memory efficient: contiguous memory                           │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   When array is full → grow by ~50%                                     │
│   ┌─────┬─────┬─────┬─────┬─────┐      ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│   │  A  │  B  │  C  │  D  │  E  │  ──▶ │  A  │  B  │  C  │  D  │  E  │null │null │null │
│   └─────┴─────┴─────┴─────┴─────┘      └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
│   (copy all elements to new larger array)                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                    LinkedList (Doubly-Linked List)                       │
│                                                                          │
│   Each element is a Node with prev/next pointers                        │
│                                                                          │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │                                                               │      │
│   │   head                                              tail      │      │
│   │    │                                                  │       │      │
│   │    ▼                                                  ▼       │      │
│   │  ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐         │      │
│   │  │  A  │ ←→ │  B  │ ←→ │  C  │ ←→ │  D  │ ←→ │  E  │         │      │
│   │  │prev │    │prev │    │prev │    │prev │    │prev │         │      │
│   │  │next │    │next │    │next │    │next │    │next │         │      │
│   │  └─────┘    └─────┘    └─────┘    └─────┘    └─────┘         │      │
│   │   null←     →←        →←         →←          →null          │      │
│   │                                                               │      │
│   └──────────────────────────────────────────────────────────────┘      │
│                                                                          │
│   ✗ Slow random access: O(n) - must traverse from head/tail             │
│   ✓ Fast insert/delete at ends: O(1) - just update pointers             │
│   ✓ Fast insert/delete in middle (if you have the node): O(1)          │
│   ✗ More memory: extra prev/next pointers per element                   │
│                                                                          │
│   Insert at position:                                                   │
│   ┌─────┐    ┌─────┐    ┌─────┐                                         │
│   │  A  │    │  X  │    │  B  │    Just update 4 pointers!             │
│   │     │──┐ │     │ ┌──│     │                                         │
│   │     │  └▶│     │◀┘  │     │                                         │
│   └─────┘    └─────┘    └─────┘                                         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### HashMap Internal Structure

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HashMap Internal Structure                            │
│                                                                          │
│   1. Bucket Array (default capacity 16)                                 │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │ Index │  0  │  1  │  2  │  3  │  4  │ ... │ 15 │              │      │
│   │       │     │     │     │     │     │     │    │              │      │
│   │       │  ↓  │  ↓  │  ↓  │  ↓  │  ↓  │     │  ↓ │              │      │
│   └──────────────────────────────────────────────────────────────┘      │
│              │           │     │                    │                   │
│              │           │     │                    │                   │
│              ▼           ▼     ▼                    ▼                   │
│           ┌─────┐     ┌─────┐ ┌─────┐           ┌─────┐                 │
│           │K1:V1│     │K2:V2│ │K4:V4│           │K7:V7│                 │
│           └──┬──┘     └─────┘ └──┬──┘           └─────┘                 │
│              │                   │                                       │
│              ▼                   ▼                                       │
│           ┌─────┐             ┌─────┐    Collision: same bucket         │
│           │K3:V3│             │K5:V5│    (different hashCodes,           │
│           └─────┘             └──┬──┘     same bucket index)            │
│                                  │                                       │
│                                  ▼                                       │
│                               ┌─────┐                                    │
│                               │K6:V6│   Linked List for collisions      │
│                               └─────┘                                    │
│                                                                          │
│   2. How put(key, value) works:                                         │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │ 1. Compute hash: hash = key.hashCode()                       │      │
│   │ 2. Find bucket index: index = hash & (capacity - 1)          │      │
│   │ 3. If bucket empty → insert new node                         │      │
│   │ 4. If bucket has entries → traverse list, check equals()     │      │
│   │    - If key exists: update value                             │      │
│   │    - If key doesn't exist: add to list (or convert to tree)  │      │
│   └──────────────────────────────────────────────────────────────┘      │
│                                                                          │
│   3. Treeification (Java 8+) - when list grows too long:                │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │                                                               │      │
│   │   List (>= 8 nodes)           Red-Black Tree                  │      │
│   │   ┌───┐                            ┌───┐                      │      │
│   │   │ A │                            │ D │                      │      │
│   │   └─┬─┘                           /     \                     │      │
│   │     ↓                         ┌───┐     ┌───┐                 │      │
│   │   ┌───┐                       │ B │     │ F │                 │      │
│   │   │ B │        ────▶         /   \     /   \                 │      │
│   │   └─┬─┘                   ┌───┐ ┌───┐ ┌───┐ ┌───┐            │      │
│   │     ↓                     │ A │ │ C │ │ E │ │ G │            │      │
│   │   ┌───┐                   └───┘ └───┘ └───┘ └───┘            │      │
│   │   │...│                                                       │      │
│   │                                                               │      │
│   │   O(n) lookup              O(log n) lookup                    │      │
│   └──────────────────────────────────────────────────────────────┘      │
│                                                                          │
│   4. Load Factor & Resizing:                                            │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │ Load Factor = entries / capacity (default 0.75)              │      │
│   │                                                               │      │
│   │ When load factor exceeded:                                   │      │
│   │   - Double capacity (16 → 32 → 64 → ...)                     │      │
│   │   - Rehash all entries (expensive!)                          │      │
│   │                                                               │      │
│   │ Capacity 16, Load Factor 0.75 → resize at 12 entries         │      │
│   └──────────────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────────────┘
```

### HashSet vs TreeSet vs LinkedHashSet

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Set Implementations Compared                          │
│                                                                          │
│   HashSet (backed by HashMap)                                           │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │ • No order guarantee                                           │     │
│   │ • Fastest: O(1) add, remove, contains                         │     │
│   │ • Allows one null element                                      │     │
│   │                                                                │     │
│   │ Iteration order: [D, A, C, B, E] (unpredictable)              │     │
│   │                                                                │     │
│   │ Internal: HashMap<E, PRESENT> where PRESENT is dummy object   │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   TreeSet (backed by TreeMap - Red-Black Tree)                          │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │ • Sorted order (natural or Comparator)                         │     │
│   │ • Slower: O(log n) add, remove, contains                       │     │
│   │ • Does NOT allow null (throws NPE)                             │     │
│   │                                                                │     │
│   │ Iteration order: [A, B, C, D, E] (sorted)                     │     │
│   │                                                                │     │
│   │ Internal structure:                                            │     │
│   │              ┌───┐                                              │     │
│   │              │ C │                                              │     │
│   │             /     \                                             │     │
│   │         ┌───┐     ┌───┐                                         │     │
│   │         │ A │     │ D │                                         │     │
│   │           \         \                                           │     │
│   │          ┌───┐     ┌───┐                                        │     │
│   │          │ B │     │ E │                                        │     │
│   │          └───┘     └───┘                                        │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   LinkedHashSet (backed by LinkedHashMap)                               │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │ • Insertion order preserved                                    │     │
│   │ • O(1) operations (like HashSet)                              │     │
│   │ • Slightly more memory (doubly-linked list)                    │     │
│   │                                                                │     │
│   │ Insert: A → B → C → D → E                                      │     │
│   │ Iteration order: [A, B, C, D, E] (insertion order)            │     │
│   │                                                                │     │
│   │ Internal: Hash table + doubly-linked list                      │     │
│   │ ┌───┬───┬───┬───┬───┐                                          │     │
│   │ │ A │ B │ C │ D │ E │  (linked list maintains order)          │     │
│   │ └─┬─┴─┬─┴─┬─┴─┬─┴─┬─┘                                          │     │
│   │   │   │   │   │   │                                            │     │
│   │   ▼   ▼   ▼   ▼   ▼  (hash buckets for O(1) lookup)           │     │
│   └───────────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────┘
```

### Generics Type Erasure

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Type Erasure                                     │
│                                                                          │
│   Generic type information exists ONLY at compile time!                 │
│                                                                          │
│   Source Code                          Compiled Bytecode                 │
│   ┌────────────────────────┐          ┌────────────────────────┐        │
│   │ List<String> strings;  │          │ List strings;          │        │
│   │ List<Integer> numbers; │   ───▶   │ List numbers;          │        │
│   │ Box<Employee> box;     │          │ Box box;               │        │
│   └────────────────────────┘          └────────────────────────┘        │
│                                                                          │
│   Class Definition:                                                     │
│   ┌────────────────────────────┐     ┌────────────────────────────┐     │
│   │ class Box<T> {             │     │ class Box {                │     │
│   │     private T item;        │ ──▶ │     private Object item;   │     │
│   │     public T get() {       │     │     public Object get() {  │     │
│   │         return item;       │     │         return item;       │     │
│   │     }                      │     │     }                      │     │
│   │ }                          │     │ }                          │     │
│   └────────────────────────────┘     └────────────────────────────┘     │
│                                                                          │
│   Bounded Type:                                                         │
│   ┌────────────────────────────┐     ┌────────────────────────────┐     │
│   │ class Box<T extends Number>│     │ class Box {                │     │
│   │     private T item;        │ ──▶ │     private Number item;   │     │
│   │ }                          │     │ }                          │     │
│   └────────────────────────────┘     └────────────────────────────┘     │
│                                                                          │
│   Consequences of Type Erasure:                                         │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │ 1. Cannot create generic arrays: new T[] ✗                    │     │
│   │ 2. Cannot use instanceof with generics: x instanceof List<String> ✗│     │
│   │ 3. Cannot create instances of type parameters: new T() ✗      │     │
│   │ 4. Cannot use primitives as type arguments: List<int> ✗       │     │
│   │ 5. At runtime, List<String> and List<Integer> are same class │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   Runtime type check:                                                   │
│   List<String> strings = new ArrayList<>();                             │
│   List<Integer> numbers = new ArrayList<>();                            │
│   strings.getClass() == numbers.getClass();  // true! Both are ArrayList│
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### PECS - Producer Extends, Consumer Super

```
┌─────────────────────────────────────────────────────────────────────────┐
│              PECS: Producer Extends, Consumer Super                      │
│                                                                          │
│   Type Hierarchy:    Object                                             │
│                         │                                                │
│                      Animal                                             │
│                      /    \                                             │
│                   Dog      Cat                                          │
│                    │                                                     │
│                 Labrador                                                │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   PRODUCER EXTENDS (Upper Bound) - Reading Data                         │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   List<? extends Animal> producer = ...;                                │
│                                                                          │
│   CAN hold:                                                             │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │ List<Animal>   - Animals                                     │       │
│   │ List<Dog>      - Dogs (Dog IS-A Animal)                      │       │
│   │ List<Cat>      - Cats (Cat IS-A Animal)                      │       │
│   │ List<Labrador> - Labradors (Labrador IS-A Animal)            │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ✓ CAN READ as Animal:                                                 │
│     Animal a = producer.get(0);  // Safe - all elements are Animals    │
│                                                                          │
│   ✗ CANNOT WRITE (except null):                                         │
│     producer.add(new Dog());     // ✗ Compile error!                   │
│     producer.add(new Animal());  // ✗ Compile error!                   │
│                                                                          │
│   Why? If list is List<Cat>, adding Dog would corrupt it!              │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   CONSUMER SUPER (Lower Bound) - Writing Data                           │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   List<? super Dog> consumer = ...;                                     │
│                                                                          │
│   CAN hold:                                                             │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │ List<Dog>    - exact type                                    │       │
│   │ List<Animal> - supertype of Dog                              │       │
│   │ List<Object> - supertype of Dog                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ✓ CAN WRITE Dog (and subtypes):                                       │
│     consumer.add(new Dog());      // Safe - Dog IS-A (Dog/Animal/Object)│
│     consumer.add(new Labrador()); // Safe - Labrador IS-A Dog          │
│                                                                          │
│   ✗ CANNOT READ (except as Object):                                     │
│     Dog d = consumer.get(0);      // ✗ Compile error!                  │
│     Object o = consumer.get(0);   // ✓ Only as Object                  │
│                                                                          │
│   Why? If list is List<Object>, element might not be a Dog!            │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   PECS Usage Example                                                    │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   // Copy from source (producer) to dest (consumer)                     │
│   void copy(List<? extends T> source,    // READ from source           │
│             List<? super T> dest) {      // WRITE to dest              │
│       for (T item : source) {                                           │
│           dest.add(item);                                               │
│       }                                                                  │
│   }                                                                      │
│                                                                          │
│   // Collections.copy signature                                         │
│   public static <T> void copy(List<? super T> dest,                     │
│                               List<? extends T> src)                    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Fail-Fast vs Fail-Safe Iterators

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 Fail-Fast vs Fail-Safe Iterators                         │
│                                                                          │
│   FAIL-FAST (ArrayList, HashMap, HashSet, etc.)                         │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │  List<String> list = new ArrayList<>(Arrays.asList("A","B")); │     │
│   │  Iterator<String> it = list.iterator();                        │     │
│   │  list.add("C");         // Structural modification!           │     │
│   │  it.next();             // Throws ConcurrentModificationException│   │
│   │                                                                │     │
│   │  How it works:                                                 │     │
│   │  ┌─────────────────────────────────────────────────────────┐  │     │
│   │  │ Collection has: modCount = 0                            │  │     │
│   │  │ Iterator stores: expectedModCount = 0                    │  │     │
│   │  │                                                          │  │     │
│   │  │ On list.add() → modCount = 1                             │  │     │
│   │  │ On it.next()  → check modCount != expectedModCount → FAIL│  │     │
│   │  └─────────────────────────────────────────────────────────┘  │     │
│   │                                                                │     │
│   │  Safe modification during iteration:                          │     │
│   │  • Use iterator.remove() - updates expectedModCount           │     │
│   │  • Use ListIterator.add() / set()                             │     │
│   │                                                                │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   FAIL-SAFE (ConcurrentHashMap, CopyOnWriteArrayList)                   │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │  CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();│ │
│   │  list.add("A");                                                │     │
│   │  list.add("B");                                                │     │
│   │                                                                │     │
│   │  for (String s : list) {                                       │     │
│   │      list.add("C");  // OK! No exception                      │     │
│   │      System.out.println(s);  // Prints A, B (original snapshot)│     │
│   │  }                                                             │     │
│   │  // After loop: list = [A, B, C, C]                           │     │
│   │                                                                │     │
│   │  How CopyOnWriteArrayList works:                               │     │
│   │  ┌─────────────────────────────────────────────────────────┐  │     │
│   │  │ On write (add/remove/set):                               │  │     │
│   │  │   1. Lock                                                 │  │     │
│   │  │   2. Copy entire array                                    │  │     │
│   │  │   3. Modify copy                                          │  │     │
│   │  │   4. Replace array reference                              │  │     │
│   │  │   5. Unlock                                               │  │     │
│   │  │                                                           │  │     │
│   │  │ Iterator: Works on snapshot (original array reference)    │  │     │
│   │  └─────────────────────────────────────────────────────────┘  │     │
│   │                                                                │     │
│   │  Trade-off: Write is O(n), but reads are fast and concurrent │     │
│   │  Best for: Read-heavy, write-rarely scenarios                 │     │
│   │                                                                │     │
│   └───────────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Code Examples

### 1) List Operations

```java
import java.util.*;

// ArrayList - best for random access
List<String> arrayList = new ArrayList<>();
arrayList.add("Apple");           // O(1) amortized
arrayList.add(0, "Orange");       // O(n) - shift elements
arrayList.get(0);                 // O(1) - direct index access
arrayList.set(0, "Grape");        // O(1) - direct replacement
arrayList.remove(0);              // O(n) - shift elements
arrayList.remove("Apple");        // O(n) - search + shift
arrayList.contains("Grape");      // O(n) - linear search
arrayList.indexOf("Grape");       // O(n) - linear search
arrayList.size();                 // O(1)

// Initialize with values
List<String> fruits = new ArrayList<>(Arrays.asList("A", "B", "C"));
List<String> fruits2 = new ArrayList<>(List.of("A", "B", "C"));
List<String> immutable = List.of("A", "B", "C");  // Immutable

// LinkedList - best for frequent insertions/deletions
List<String> linkedList = new LinkedList<>();
linkedList.add("First");          // O(1) - add to end
linkedList.addFirst("Zero");      // O(1) - LinkedList specific
linkedList.addLast("Last");       // O(1)
linkedList.removeFirst();         // O(1)
linkedList.removeLast();          // O(1)
linkedList.get(5);                // O(n) - must traverse!
linkedList.getFirst();            // O(1)
linkedList.getLast();             // O(1)

// As Queue/Deque
LinkedList<String> queue = new LinkedList<>();
queue.offer("A");                 // Add to tail (queue)
queue.poll();                     // Remove from head (queue)
queue.push("B");                  // Add to head (stack)
queue.pop();                      // Remove from head (stack)

// List operations
Collections.sort(fruits);                    // Sort in place
Collections.reverse(fruits);                 // Reverse in place
Collections.shuffle(fruits);                 // Random shuffle
Collections.binarySearch(fruits, "B");       // Must be sorted first!
Collections.frequency(fruits, "A");          // Count occurrences
Collections.max(fruits);                     // Find max
Collections.min(fruits);                     // Find min

// Sublist (view, not copy!)
List<String> subList = fruits.subList(0, 2);
subList.set(0, "X");              // Modifies original list!

// Convert to array
String[] array = fruits.toArray(new String[0]);
String[] array2 = fruits.toArray(String[]::new);  // Java 11+

// Safe iteration with modification
Iterator<String> iter = fruits.iterator();
while (iter.hasNext()) {
    String fruit = iter.next();
    if (fruit.equals("B")) {
        iter.remove();            // Safe removal during iteration
    }
}

// ListIterator - bidirectional, can add/modify
ListIterator<String> listIter = fruits.listIterator();
while (listIter.hasNext()) {
    String fruit = listIter.next();
    listIter.set(fruit.toUpperCase());  // Modify current element
}
while (listIter.hasPrevious()) {
    System.out.println(listIter.previous());  // Go backwards
}
```

### 2) Set Operations

```java
import java.util.*;

// HashSet - fastest, unordered
Set<String> hashSet = new HashSet<>();
hashSet.add("Apple");             // O(1)
hashSet.add("Apple");             // Returns false - duplicate
hashSet.contains("Apple");        // O(1)
hashSet.remove("Apple");          // O(1)
hashSet.size();                   // O(1)

// TreeSet - sorted
Set<String> treeSet = new TreeSet<>();
treeSet.add("Banana");            // O(log n)
treeSet.add("Apple");
treeSet.add("Cherry");
// Iteration: Apple, Banana, Cherry (sorted!)

// TreeSet navigation methods
TreeSet<Integer> numbers = new TreeSet<>(Set.of(10, 20, 30, 40, 50));
numbers.first();                  // 10 - smallest
numbers.last();                   // 50 - largest
numbers.lower(30);                // 20 - largest < 30
numbers.higher(30);               // 40 - smallest > 30
numbers.floor(25);                // 20 - largest <= 25
numbers.ceiling(25);              // 30 - smallest >= 25
numbers.headSet(30);              // [10, 20] - elements < 30
numbers.tailSet(30);              // [30, 40, 50] - elements >= 30
numbers.subSet(20, 40);           // [20, 30] - elements in range
numbers.descendingSet();          // View in reverse order

// LinkedHashSet - insertion order preserved
Set<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("Third");
linkedHashSet.add("First");
linkedHashSet.add("Second");
// Iteration: Third, First, Second (insertion order)

// Set operations
Set<Integer> set1 = new HashSet<>(Set.of(1, 2, 3, 4));
Set<Integer> set2 = new HashSet<>(Set.of(3, 4, 5, 6));

// Union
Set<Integer> union = new HashSet<>(set1);
union.addAll(set2);               // [1, 2, 3, 4, 5, 6]

// Intersection
Set<Integer> intersection = new HashSet<>(set1);
intersection.retainAll(set2);     // [3, 4]

// Difference
Set<Integer> difference = new HashSet<>(set1);
difference.removeAll(set2);       // [1, 2]

// Symmetric difference
Set<Integer> symDiff = new HashSet<>(set1);
symDiff.addAll(set2);
Set<Integer> temp = new HashSet<>(set1);
temp.retainAll(set2);
symDiff.removeAll(temp);          // [1, 2, 5, 6]

// Custom ordering with Comparator
Set<String> customOrder = new TreeSet<>(
    Comparator.comparingInt(String::length)
              .thenComparing(Comparator.naturalOrder())
);
customOrder.add("Apple");
customOrder.add("Banana");
customOrder.add("Kiwi");
// Iteration: Kiwi, Apple, Banana (by length, then alphabetically)

// Immutable sets
Set<String> immutable = Set.of("A", "B", "C");  // Unmodifiable
// immutable.add("D");  // Throws UnsupportedOperationException

// Convert to unmodifiable
Set<String> unmodifiable = Collections.unmodifiableSet(hashSet);
```

### 3) Map Operations

```java
import java.util.*;

// HashMap - fastest, unordered
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("Apple", 100);        // O(1)
hashMap.put("Banana", 200);
hashMap.put("Apple", 150);        // Replaces existing value
hashMap.get("Apple");             // 150, O(1)
hashMap.getOrDefault("Orange", 0);// 0 (default if not found)
hashMap.containsKey("Apple");     // true, O(1)
hashMap.containsValue(200);       // true, O(n) - must scan values
hashMap.remove("Banana");         // O(1)
hashMap.size();                   // O(1)

// putIfAbsent - only puts if key is absent
hashMap.putIfAbsent("Orange", 300);  // Puts 300
hashMap.putIfAbsent("Apple", 400);   // Does nothing (key exists)

// compute methods
hashMap.compute("Apple", (k, v) -> v == null ? 1 : v + 1);
hashMap.computeIfAbsent("Grape", k -> 500);  // Creates if absent
hashMap.computeIfPresent("Apple", (k, v) -> v + 50);  // Updates if present

// merge - useful for combining values
hashMap.merge("Apple", 50, Integer::sum);  // Add 50 to existing or put 50

// TreeMap - sorted by keys
Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("Banana", 2);
treeMap.put("Apple", 1);
treeMap.put("Cherry", 3);
// Keys iteration: Apple, Banana, Cherry (sorted)

// TreeMap navigation
TreeMap<Integer, String> nav = new TreeMap<>();
nav.put(1, "One");
nav.put(3, "Three");
nav.put(5, "Five");
nav.firstKey();                   // 1
nav.lastKey();                    // 5
nav.lowerKey(3);                  // 1 (largest key < 3)
nav.higherKey(3);                 // 5 (smallest key > 3)
nav.floorKey(4);                  // 3 (largest key <= 4)
nav.ceilingKey(4);                // 5 (smallest key >= 4)

// LinkedHashMap - insertion order preserved
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
linkedHashMap.put("Third", 3);
linkedHashMap.put("First", 1);
linkedHashMap.put("Second", 2);
// Keys iteration: Third, First, Second (insertion order)

// LinkedHashMap with access order (LRU cache)
Map<String, Integer> lruCache = new LinkedHashMap<>(16, 0.75f, true);
// true = access order (most recently accessed last)

// Iterating maps
for (Map.Entry<String, Integer> entry : hashMap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

hashMap.forEach((key, value) -> System.out.println(key + ": " + value));

for (String key : hashMap.keySet()) {
    System.out.println(key);
}

for (Integer value : hashMap.values()) {
    System.out.println(value);
}

// Immutable maps
Map<String, Integer> immutable = Map.of(
    "Apple", 1,
    "Banana", 2,
    "Cherry", 3
);

Map<String, Integer> immutableMany = Map.ofEntries(
    Map.entry("A", 1),
    Map.entry("B", 2),
    Map.entry("C", 3)
);

// Copy constructors
Map<String, Integer> copy = new HashMap<>(hashMap);
Map<String, Integer> unmodifiable = Collections.unmodifiableMap(hashMap);

// Replace operations
hashMap.replace("Apple", 999);             // Replace if key exists
hashMap.replace("Apple", 150, 1000);       // Replace if key-value matches
hashMap.replaceAll((k, v) -> v * 2);       // Transform all values
```

### 4) Queue and Deque Operations

```java
import java.util.*;

// Queue - FIFO (First In First Out)
Queue<String> queue = new LinkedList<>();
queue.offer("First");             // Add to tail, returns false if full
queue.offer("Second");
queue.offer("Third");
queue.poll();                     // Remove from head, returns null if empty
queue.peek();                     // Look at head without removing

// Queue methods that throw exceptions (avoid for optional operations)
queue.add("Fourth");              // Throws if full
queue.remove();                   // Throws if empty
queue.element();                  // Throws if empty

// PriorityQueue - elements sorted by priority
Queue<Integer> priorityQueue = new PriorityQueue<>();  // Min-heap
priorityQueue.offer(30);
priorityQueue.offer(10);
priorityQueue.offer(20);
priorityQueue.poll();             // 10 (smallest first!)
priorityQueue.poll();             // 20
priorityQueue.poll();             // 30

// Max-heap (reverse order)
Queue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(30);
maxHeap.offer(10);
maxHeap.offer(20);
maxHeap.poll();                   // 30 (largest first!)

// Custom priority
Queue<Task> taskQueue = new PriorityQueue<>(
    Comparator.comparingInt(Task::getPriority)
              .thenComparing(Task::getCreatedTime)
);

// Deque - Double-Ended Queue
Deque<String> deque = new ArrayDeque<>();
deque.offerFirst("A");            // Add to front
deque.offerLast("B");             // Add to back
deque.peekFirst();                // Look at front
deque.peekLast();                 // Look at back
deque.pollFirst();                // Remove from front
deque.pollLast();                 // Remove from back

// Deque as Stack (LIFO)
Deque<String> stack = new ArrayDeque<>();
stack.push("First");              // Push to top (front)
stack.push("Second");
stack.pop();                      // Pop from top, returns "Second"
stack.peek();                     // Look at top

// ArrayDeque vs LinkedList for Queue/Deque:
// ArrayDeque: Faster, no null elements, array-backed
// LinkedList: Slower, allows null, more memory per element
// Prefer ArrayDeque unless you need null elements

// BlockingQueue (concurrent)
import java.util.concurrent.*;

BlockingQueue<String> blockingQueue = new LinkedBlockingQueue<>(10);
blockingQueue.put("item");        // Blocks if full
blockingQueue.take();             // Blocks if empty
blockingQueue.offer("item", 1, TimeUnit.SECONDS);  // Timeout
blockingQueue.poll(1, TimeUnit.SECONDS);           // Timeout
```

### 5) Generics Basics

```java
// Generic class
public class Box<T> {
    private T content;
    
    public void set(T content) {
        this.content = content;
    }
    
    public T get() {
        return content;
    }
}

// Usage
Box<String> stringBox = new Box<>();
stringBox.set("Hello");
String s = stringBox.get();  // No casting needed

Box<Integer> intBox = new Box<>();
intBox.set(42);
Integer i = intBox.get();

// Generic method
public class Util {
    public static <T> T getFirst(List<T> list) {
        return list.isEmpty() ? null : list.get(0);
    }
    
    public static <T, U> Pair<T, U> makePair(T first, U second) {
        return new Pair<>(first, second);
    }
}

// Usage - type inference
String first = Util.getFirst(List.of("A", "B", "C"));
Pair<String, Integer> pair = Util.makePair("Age", 30);

// Generic interface
public interface Repository<T, ID> {
    T findById(ID id);
    List<T> findAll();
    T save(T entity);
    void delete(ID id);
}

public class UserRepository implements Repository<User, Long> {
    @Override
    public User findById(Long id) { /* ... */ }
    
    @Override
    public List<User> findAll() { /* ... */ }
    
    @Override
    public User save(User entity) { /* ... */ }
    
    @Override
    public void delete(Long id) { /* ... */ }
}

// Multiple type parameters
public class Pair<K, V> {
    private K key;
    private V value;
    
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    public K getKey() { return key; }
    public V getValue() { return value; }
}

// Bounded type parameters
public class NumberBox<T extends Number> {
    private T number;
    
    public double doubleValue() {
        return number.doubleValue();  // Can call Number methods
    }
}

NumberBox<Integer> integerBox = new NumberBox<>();
NumberBox<Double> doubleBox = new NumberBox<>();
// NumberBox<String> stringBox = new NumberBox<>();  // Compile error!

// Multiple bounds
public class MultiBound<T extends Number & Comparable<T>> {
    // T must be a Number AND implement Comparable
}
```

### 6) Wildcards and PECS

```java
// Unbounded wildcard - read as Object, cannot add
public void printList(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
    // list.add("something");  // Compile error!
    // Can only add null
}

// Upper bounded wildcard (extends) - PRODUCER
public double sumNumbers(List<? extends Number> numbers) {
    double sum = 0;
    for (Number n : numbers) {  // Can read as Number
        sum += n.doubleValue();
    }
    // numbers.add(1);  // Compile error! Cannot add
    return sum;
}

// Works with any List of Number or its subtypes
sumNumbers(List.of(1, 2, 3));           // List<Integer>
sumNumbers(List.of(1.0, 2.0, 3.0));     // List<Double>
sumNumbers(List.of(1L, 2L, 3L));        // List<Long>

// Lower bounded wildcard (super) - CONSUMER
public void addNumbers(List<? super Integer> list) {
    list.add(1);      // Can add Integer (and subtypes)
    list.add(2);
    // Integer i = list.get(0);  // Compile error! Can only read as Object
    Object o = list.get(0);      // OK, but not useful
}

// Works with any List that can hold Integers
addNumbers(new ArrayList<Integer>());
addNumbers(new ArrayList<Number>());
addNumbers(new ArrayList<Object>());

// PECS in action - copy from source to destination
public static <T> void copy(List<? extends T> source,   // Producer (read)
                            List<? super T> dest) {      // Consumer (write)
    for (T item : source) {
        dest.add(item);
    }
}

// Usage
List<Integer> integers = List.of(1, 2, 3);
List<Number> numbers = new ArrayList<>();
copy(integers, numbers);  // Works! Integer extends Number

// Practical example: Collections.addAll
public static <T> boolean addAll(Collection<? super T> c, T... elements) {
    boolean result = false;
    for (T element : elements) {
        result |= c.add(element);
    }
    return result;
}

// Practical example: Comparator
public static <T> T max(Collection<? extends T> collection,
                        Comparator<? super T> comparator) {
    // Collection is producer (we read from it)
    // Comparator is consumer (we pass T values to it)
}

// Wildcard capture
public void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);  // Delegate to helper for type safety
}

private <T> void swapHelper(List<T> list, int i, int j) {
    T temp = list.get(i);
    list.set(i, list.get(j));
    list.set(j, temp);
}
```

### 7) equals() and hashCode() for Collections

```java
// Correct implementation for use in HashMap/HashSet
public class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}

// What happens without proper equals/hashCode
class BadKey {
    String value;
    BadKey(String value) { this.value = value; }
    // No equals/hashCode override!
}

Map<BadKey, String> map = new HashMap<>();
map.put(new BadKey("test"), "value");
map.get(new BadKey("test"));  // Returns NULL! Different object = different hash

// With proper equals/hashCode
class GoodKey {
    String value;
    GoodKey(String value) { this.value = value; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof GoodKey)) return false;
        return Objects.equals(value, ((GoodKey) o).value);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}

Map<GoodKey, String> goodMap = new HashMap<>();
goodMap.put(new GoodKey("test"), "value");
goodMap.get(new GoodKey("test"));  // Returns "value"!

// DANGER: Mutable keys
class MutableKey {
    String value;
    
    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
    
    @Override
    public boolean equals(Object o) {
        // ... 
    }
}

MutableKey key = new MutableKey();
key.value = "original";
Set<MutableKey> set = new HashSet<>();
set.add(key);

key.value = "changed";  // Mutation!
set.contains(key);       // FALSE! Key is "lost"

// Best practice: Use immutable objects as keys
// Or use records (immutable by design)
record PersonKey(String name, int age) {}  // Auto equals/hashCode
```

### 8) Collections Utility Methods

```java
import java.util.*;

List<Integer> numbers = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5, 9, 2, 6));

// Sorting
Collections.sort(numbers);                          // [1, 1, 2, 3, 4, 5, 6, 9]
Collections.sort(numbers, Comparator.reverseOrder()); // Descending

// Binary search (list must be sorted!)
Collections.sort(numbers);
int index = Collections.binarySearch(numbers, 4);   // Returns index or -(insertion point + 1)

// Min/Max
int min = Collections.min(numbers);                 // 1
int max = Collections.max(numbers);                 // 9
int maxByComparator = Collections.max(numbers, Comparator.reverseOrder());

// Reverse
Collections.reverse(numbers);                       // In place reversal

// Shuffle
Collections.shuffle(numbers);                       // Random order
Collections.shuffle(numbers, new Random(42));       // Reproducible shuffle

// Fill
Collections.fill(numbers, 0);                       // All elements = 0

// Copy (destination must have enough space)
List<Integer> dest = new ArrayList<>(Collections.nCopies(numbers.size(), 0));
Collections.copy(dest, numbers);

// Swap
Collections.swap(numbers, 0, numbers.size() - 1);

// Rotate
Collections.rotate(numbers, 2);                     // Rotate right by 2

// Frequency
int count = Collections.frequency(numbers, 1);      // Count of 1s

// Disjoint (no common elements)
boolean disjoint = Collections.disjoint(list1, list2);

// Replace all
Collections.replaceAll(numbers, 1, 100);            // Replace all 1s with 100

// Immutable wrappers
List<String> unmodifiableList = Collections.unmodifiableList(new ArrayList<>());
Set<String> unmodifiableSet = Collections.unmodifiableSet(new HashSet<>());
Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(new HashMap<>());

// Synchronized wrappers (for thread safety)
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());

// Singleton collections (immutable, single element)
Set<String> singletonSet = Collections.singleton("only");
List<String> singletonList = Collections.singletonList("only");
Map<String, Integer> singletonMap = Collections.singletonMap("key", 1);

// Empty collections (immutable)
List<String> emptyList = Collections.emptyList();
Set<String> emptySet = Collections.emptySet();
Map<String, Integer> emptyMap = Collections.emptyMap();

// nCopies - immutable list with n copies
List<String> repeated = Collections.nCopies(5, "hello");  // [hello, hello, hello, hello, hello]

// Checked collections (runtime type safety)
List<String> checkedList = Collections.checkedList(new ArrayList<>(), String.class);
// checkedList.add(123);  // ClassCastException at runtime!
```

### 9) Immutable vs Unmodifiable Collections

```java
// Java 9+ Factory Methods - IMMUTABLE
List<String> immutableList = List.of("A", "B", "C");
Set<String> immutableSet = Set.of("A", "B", "C");
Map<String, Integer> immutableMap = Map.of("A", 1, "B", 2);

// Characteristics:
// - Null elements not allowed (NPE)
// - Duplicate elements not allowed in Set (IllegalArgumentException)
// - Duplicate keys not allowed in Map (IllegalArgumentException)
// - Truly immutable - no way to modify
// - Optimized for memory and performance

// immutableList.add("D");        // UnsupportedOperationException
// immutableList.set(0, "X");     // UnsupportedOperationException

// Unmodifiable Wrappers - VIEWS (underlying can change!)
List<String> original = new ArrayList<>();
original.add("A");
original.add("B");

List<String> unmodifiableView = Collections.unmodifiableList(original);
// unmodifiableView.add("C");     // UnsupportedOperationException

original.add("C");                // Modifies original!
System.out.println(unmodifiableView);  // [A, B, C] - reflects change!

// Defensive Copy - create independent immutable copy
List<String> defensiveCopy = List.copyOf(original);
original.add("D");
System.out.println(defensiveCopy);  // [A, B, C] - unchanged!

// When to use what:
// 
// List.of(), Set.of(), Map.of():
//   - Small fixed collections
//   - Return types for APIs
//   - No nulls allowed
//
// Collections.unmodifiableXxx():
//   - Expose read-only view of mutable collection
//   - When you need to control access
//   - Underlying collection might change
//
// List.copyOf(), Set.copyOf(), Map.copyOf():
//   - Create immutable copy of existing collection
//   - Defensive copies in constructors/getters
//   - Handles nulls gracefully (NPE on null elements)

// In class design
public class SafeClass {
    private final List<String> items;
    
    public SafeClass(List<String> items) {
        this.items = List.copyOf(items);  // Defensive copy
    }
    
    public List<String> getItems() {
        return items;  // Safe - already immutable
    }
}
```

---

## Big-O Complexity Reference

### List Operations

| Operation | ArrayList | LinkedList |
|-----------|-----------|------------|
| get(index) | O(1) | O(n) |
| add(element) | O(1)* | O(1) |
| add(index, element) | O(n) | O(n)** |
| remove(index) | O(n) | O(n)** |
| remove(object) | O(n) | O(n) |
| contains(object) | O(n) | O(n) |
| size() | O(1) | O(1) |
| iterator.remove() | O(n) | O(1) |

\* Amortized - occasional O(n) for resize
\** O(1) if you have iterator at position

### Set Operations

| Operation | HashSet | LinkedHashSet | TreeSet |
|-----------|---------|---------------|---------|
| add | O(1) | O(1) | O(log n) |
| remove | O(1) | O(1) | O(log n) |
| contains | O(1) | O(1) | O(log n) |
| size | O(1) | O(1) | O(1) |
| iteration | O(n) | O(n) | O(n) |

### Map Operations

| Operation | HashMap | LinkedHashMap | TreeMap |
|-----------|---------|---------------|---------|
| put | O(1) | O(1) | O(log n) |
| get | O(1) | O(1) | O(log n) |
| remove | O(1) | O(1) | O(log n) |
| containsKey | O(1) | O(1) | O(log n) |
| containsValue | O(n) | O(n) | O(n) |

### Queue/Deque Operations

| Operation | LinkedList | ArrayDeque | PriorityQueue |
|-----------|------------|------------|---------------|
| offer | O(1) | O(1)* | O(log n) |
| poll | O(1) | O(1) | O(log n) |
| peek | O(1) | O(1) | O(1) |
| remove(Object) | O(n) | O(n) | O(n) |

---

## Interview Questions

### Basic Questions

1. **What is the difference between ArrayList and LinkedList?**
   - ArrayList: Array-backed, O(1) random access, O(n) insert/delete in middle
   - LinkedList: Node-based, O(n) random access, O(1) insert/delete at ends
   - Use ArrayList for most cases unless frequent insertions at random positions

2. **What is the difference between HashSet, TreeSet, and LinkedHashSet?**
   - HashSet: Fastest O(1), no order, uses hashCode/equals
   - TreeSet: Sorted order, O(log n), uses Comparable/Comparator
   - LinkedHashSet: Insertion order preserved, O(1), slightly more memory

3. **What is the difference between HashMap and TreeMap?**
   - HashMap: O(1) operations, no order guarantee
   - TreeMap: O(log n) operations, sorted by keys
   - HashMap allows one null key; TreeMap doesn't (throws NPE)

4. **What happens if you don't override hashCode() when you override equals()?**
   - Objects that are equal may have different hashCodes
   - They'll be placed in different buckets in HashMap/HashSet
   - You won't be able to find them correctly

5. **What is the difference between fail-fast and fail-safe iterators?**
   - Fail-fast: Throws ConcurrentModificationException on structural change
   - Fail-safe: Works on snapshot, no exception, may not see changes
   - ArrayList/HashMap are fail-fast; ConcurrentHashMap is fail-safe

### Intermediate Questions

6. **How does HashMap work internally?**
   - Uses array of buckets (default 16)
   - Key's hashCode determines bucket index
   - Collisions handled by linked list (→ tree when > 8 nodes)
   - Resizes when load factor (0.75) exceeded
   - Java 8+: Treeifies long chains for O(log n) worst case

7. **What is the difference between Comparable and Comparator?**
   - Comparable: `compareTo()`, natural ordering, in the class
   - Comparator: `compare()`, custom ordering, external
   - Comparable: one per class; Comparator: many possible

8. **Explain generics type erasure.**
   - Generic type info removed at compile time
   - `List<String>` becomes `List` in bytecode
   - Cannot use instanceof with generics
   - Cannot create generic arrays: `new T[]`
   - Runtime: `List<String>` and `List<Integer>` are same class

9. **What is PECS?**
   - Producer Extends, Consumer Super
   - `? extends T`: Read as T, cannot write (producer)
   - `? super T`: Write T, read as Object (consumer)
   - Example: `Collections.copy(dest<? super T>, src<? extends T>)`

10. **What is the difference between Collection and Collections?**
    - Collection: Interface (root of collection hierarchy)
    - Collections: Utility class with static methods (sort, shuffle, etc.)

### Advanced Questions

11. **When would you use LinkedHashMap?**
    - Need HashMap performance with iteration order
    - Implementing LRU cache (access-order mode)
    - Processing in insertion order

12. **What is the initial capacity and load factor of HashMap?**
    - Initial capacity: 16
    - Load factor: 0.75
    - Resize when: size > capacity * load factor
    - Always specify initial capacity if you know approximate size

13. **How do you make a collection thread-safe?**
    - `Collections.synchronizedXxx()` - basic thread safety
    - Concurrent collections: ConcurrentHashMap, CopyOnWriteArrayList
    - Concurrent collections are preferred for better performance

14. **What is the difference between List.of() and Arrays.asList()?**
    - `List.of()`: Immutable, no nulls allowed
    - `Arrays.asList()`: Fixed-size view of array, allows set() but not add()
    - `Arrays.asList()` changes affect original array

15. **Why can't you use primitives as generic type parameters?**
    - Type erasure replaces generics with Object
    - Primitives don't extend Object
    - Use wrapper classes instead (Integer, Double, etc.)

---

## Quick Reference

### Choosing the Right Collection

```
Need ordered sequence with duplicates?
├─ Yes → Need fast random access?
│        ├─ Yes → ArrayList
│        └─ No → Need fast insert/delete at both ends?
│                ├─ Yes → LinkedList or ArrayDeque
│                └─ No → ArrayList
└─ No → Need unique elements?
         ├─ Yes → Need sorted order?
         │        ├─ Yes → TreeSet
         │        └─ No → Need insertion order?
         │                 ├─ Yes → LinkedHashSet
         │                 └─ No → HashSet
         └─ No → Need key-value pairs?
                  ├─ Yes → Need sorted by keys?
                  │        ├─ Yes → TreeMap
                  │        └─ No → Need insertion order?
                  │                 ├─ Yes → LinkedHashMap
                  │                 └─ No → HashMap
                  └─ No → Need FIFO queue?
                           ├─ Yes → Need priority?
                           │        ├─ Yes → PriorityQueue
                           │        └─ No → ArrayDeque or LinkedList
                           └─ No → Need stack (LIFO)?
                                    └─ Yes → ArrayDeque (push/pop)
```

### Common Patterns

```java
// Initialize with values
List<String> list = new ArrayList<>(List.of("A", "B", "C"));
Set<String> set = new HashSet<>(Set.of("A", "B", "C"));
Map<String, Integer> map = new HashMap<>(Map.of("A", 1, "B", 2));

// Remove while iterating
list.removeIf(s -> s.startsWith("A"));

// Get or compute
map.computeIfAbsent("key", k -> expensiveOperation());

// Group by
Map<String, List<Person>> byCity = persons.stream()
    .collect(Collectors.groupingBy(Person::getCity));

// Frequency count
Map<String, Long> frequency = words.stream()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```
