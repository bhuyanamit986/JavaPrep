# Cheat Sheets & Last-Minute Revision

Quick reference cards for rapid review before your Java interview.

## Definitions

- **Cheat sheet**: a compact summary of high-frequency facts and APIs.
- **Last-minute revision**: focused review to refresh recall, not to learn from scratch.
- **Recall cue**: a short prompt that helps you remember a concept quickly.
- **High-signal topic**: a topic that appears frequently in interviews (collections, concurrency, JVM).

## Illustrations

- **Flashcard mindset**: read a prompt, try to answer from memory, then verify.
- **Time-boxing**: spend 10-15 minutes per section to refresh key facts.
- **Layered review**: start with essentials, then skim deeper sections if time allows.

## Code Examples

```java
// Quick recall: map + filter with streams
java.util.List<Integer> evens = java.util.List.of(1, 2, 3, 4).stream()
    .filter(n -> n % 2 == 0)
    .toList();
```

## Interview Questions

1. What is the difference between `==` and `equals()`?
2. What are the default values of primitive types?
3. What is the difference between `HashMap` and `HashSet`?
4. What does `volatile` guarantee?
5. What is the purpose of a unit test?

---

## 1) Core Java Essentials

### Primitives

| Type | Size | Range | Default |
|------|------|-------|---------|
| `byte` | 8 bits | -128 to 127 | 0 |
| `short` | 16 bits | -32,768 to 32,767 | 0 |
| `int` | 32 bits | -2³¹ to 2³¹-1 | 0 |
| `long` | 64 bits | -2⁶³ to 2⁶³-1 | 0L |
| `float` | 32 bits | ±3.4E38 | 0.0f |
| `double` | 64 bits | ±1.7E308 | 0.0d |
| `char` | 16 bits | 0 to 65,535 | '\u0000' |
| `boolean` | 1 bit | true/false | false |

### String Methods

```java
s.length()                    // Length
s.charAt(i)                   // Character at index
s.substring(start)            // From start to end
s.substring(start, end)       // From start to end-1
s.indexOf(str)                // First index of str (-1 if not found)
s.lastIndexOf(str)            // Last index of str
s.contains(str)               // Contains check
s.startsWith(prefix)          // Prefix check
s.endsWith(suffix)            // Suffix check
s.equals(other)               // Content equality
s.equalsIgnoreCase(other)     // Case-insensitive equals
s.compareTo(other)            // Lexicographic comparison
s.toUpperCase()               // To upper case
s.toLowerCase()               // To lower case
s.trim()                      // Remove leading/trailing whitespace
s.strip()                     // Remove whitespace (Unicode-aware)
s.replace(old, new)           // Replace all occurrences
s.split(regex)                // Split into array
s.join(delimiter, elements)   // Join elements with delimiter
s.format(format, args)        // Format string
String.valueOf(x)             // Convert to string
s.toCharArray()               // Convert to char array
s.isEmpty()                   // Check if length == 0
s.isBlank()                   // Check if empty or whitespace only
```

### equals() and hashCode() Contract

```java
// 1. If a.equals(b), then a.hashCode() == b.hashCode()
// 2. If a.hashCode() != b.hashCode(), then !a.equals(b)
// 3. If a.hashCode() == b.hashCode(), a.equals(b) may or may not be true

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
```

### Access Modifiers

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `public` | ✓ | ✓ | ✓ | ✓ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| (default) | ✓ | ✓ | ✗ | ✗ |
| `private` | ✓ | ✗ | ✗ | ✗ |

### Keywords Quick Reference

| Keyword | Purpose |
|---------|---------|
| `final` | Constant variable, prevent override/inheritance |
| `static` | Class-level member (shared across instances) |
| `volatile` | Ensure visibility across threads |
| `synchronized` | Thread-safe access (monitor lock) |
| `transient` | Exclude from serialization |
| `abstract` | Incomplete class/method |
| `native` | Method implemented in native code |
| `strictfp` | Strict floating-point calculations |

---

## 2) Collections Quick Reference

### Collection Hierarchy

```
Iterable
└── Collection
    ├── List (ordered, duplicates allowed)
    │   ├── ArrayList    - O(1) access, O(n) insert/delete
    │   ├── LinkedList   - O(n) access, O(1) insert/delete at ends
    │   └── Vector       - Synchronized ArrayList (legacy)
    │
    ├── Set (no duplicates)
    │   ├── HashSet      - O(1) operations, unordered
    │   ├── LinkedHashSet - O(1) operations, insertion order
    │   └── TreeSet      - O(log n) operations, sorted
    │
    └── Queue
        ├── PriorityQueue - Heap-based, O(log n) insert/remove
        ├── LinkedList    - Implements Deque
        └── ArrayDeque    - Double-ended queue, faster than LinkedList

Map (not a Collection)
├── HashMap      - O(1) operations, unordered
├── LinkedHashMap - O(1) operations, insertion/access order
├── TreeMap      - O(log n) operations, sorted keys
├── Hashtable    - Synchronized HashMap (legacy)
└── ConcurrentHashMap - Thread-safe, high concurrency
```

### List Operations

```java
list.add(element)             // Add at end
list.add(index, element)      // Add at index
list.get(index)               // Get at index
list.set(index, element)      // Replace at index
list.remove(index)            // Remove at index
list.remove(object)           // Remove first occurrence
list.contains(object)         // Check existence
list.indexOf(object)          // First index (-1 if not found)
list.size()                   // Size
list.isEmpty()                // Check if empty
list.clear()                  // Remove all
list.subList(from, to)        // View of portion
Collections.sort(list)        // Sort
Collections.reverse(list)     // Reverse
Collections.shuffle(list)     // Randomize
```

### Map Operations

```java
map.put(key, value)           // Add/update
map.get(key)                  // Get value (null if absent)
map.getOrDefault(key, def)    // Get with default
map.containsKey(key)          // Check key exists
map.containsValue(value)      // Check value exists
map.remove(key)               // Remove by key
map.size()                    // Size
map.isEmpty()                 // Check if empty
map.keySet()                  // Set of keys
map.values()                  // Collection of values
map.entrySet()                // Set of entries

// Advanced operations
map.putIfAbsent(key, value)   // Put only if absent
map.computeIfAbsent(key, k -> value)  // Compute if absent
map.computeIfPresent(key, (k, v) -> newValue)
map.merge(key, value, (old, new) -> merged)
map.replaceAll((k, v) -> newValue)
```

### Set Operations

```java
set.add(element)              // Add (returns false if exists)
set.remove(element)           // Remove
set.contains(element)         // Check existence
set.size()                    // Size
set.isEmpty()                 // Check if empty

// Set operations
set1.addAll(set2)             // Union
set1.retainAll(set2)          // Intersection
set1.removeAll(set2)          // Difference
```

### Queue/Deque Operations

```java
// Queue (FIFO)
queue.offer(e)                // Add to tail
queue.poll()                  // Remove from head
queue.peek()                  // View head

// Deque as Stack (LIFO)
stack.push(e)                 // Add to head
stack.pop()                   // Remove from head
stack.peek()                  // View head

// PriorityQueue
pq.offer(e)                   // Add O(log n)
pq.poll()                     // Remove min/max O(log n)
pq.peek()                     // View min/max O(1)
```

### Time Complexity Summary

| Operation | ArrayList | LinkedList | HashSet | TreeSet | HashMap | TreeMap |
|-----------|-----------|------------|---------|---------|---------|---------|
| Access | O(1) | O(n) | - | - | - | - |
| Search | O(n) | O(n) | O(1) | O(log n) | O(1) | O(log n) |
| Insert | O(n)* | O(1)** | O(1) | O(log n) | O(1) | O(log n) |
| Delete | O(n) | O(1)** | O(1) | O(log n) | O(1) | O(log n) |

*Amortized O(1) at end; **At known position

---

## 3) Streams Cheat Sheet

### Creating Streams

```java
Stream.of(1, 2, 3)                     // From values
Arrays.stream(array)                   // From array
list.stream()                          // From collection
Stream.iterate(0, n -> n + 1)          // Infinite iterate
Stream.generate(Math::random)          // Infinite generate
IntStream.range(0, 10)                 // 0 to 9
IntStream.rangeClosed(1, 10)           // 1 to 10
Files.lines(path)                      // From file
```

### Intermediate Operations (Lazy)

```java
.filter(predicate)                     // Keep matching elements
.map(function)                         // Transform elements
.flatMap(function)                     // Flatten nested streams
.distinct()                            // Remove duplicates
.sorted()                              // Natural order sort
.sorted(comparator)                    // Custom sort
.limit(n)                              // First n elements
.skip(n)                               // Skip first n elements
.peek(consumer)                        // Debug (side effect)
.mapToInt/Long/Double(function)        // Convert to primitive stream
```

### Terminal Operations (Eager)

```java
.forEach(consumer)                     // Iterate (no return)
.collect(collector)                    // Collect to collection
.toList()                              // Collect to unmodifiable list
.toArray()                             // Collect to array
.reduce(identity, accumulator)         // Reduce to single value
.count()                               // Count elements
.findFirst()                           // First element (Optional)
.findAny()                             // Any element (Optional)
.anyMatch(predicate)                   // Any match?
.allMatch(predicate)                   // All match?
.noneMatch(predicate)                  // None match?
.min(comparator)                       // Minimum (Optional)
.max(comparator)                       // Maximum (Optional)
```

### Common Collectors

```java
Collectors.toList()                    // To ArrayList
Collectors.toSet()                     // To HashSet
Collectors.toMap(keyMapper, valueMapper)
Collectors.joining(delimiter)          // Join strings
Collectors.counting()                  // Count
Collectors.summingInt(mapper)          // Sum
Collectors.averagingInt(mapper)        // Average
Collectors.groupingBy(classifier)      // Group into Map
Collectors.partitioningBy(predicate)   // Partition into true/false
Collectors.mapping(mapper, downstream) // Map then collect
```

### Common Patterns

```java
// Filter and collect
List<String> filtered = list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());

// Map and collect
List<Integer> lengths = list.stream()
    .map(String::length)
    .collect(Collectors.toList());

// Group by
Map<Integer, List<String>> byLength = list.stream()
    .collect(Collectors.groupingBy(String::length));

// Count by property
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDept, Collectors.counting()));

// Find max
Optional<Employee> highest = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));

// Sum
int total = orders.stream()
    .mapToInt(Order::getAmount)
    .sum();

// Join strings
String names = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", "));
```

---

## 4) Concurrency Cheat Sheet

### Thread Creation

```java
// Runnable (no return)
Thread t1 = new Thread(() -> doWork());
t1.start();

// Callable (with return)
ExecutorService executor = Executors.newFixedThreadPool(4);
Future<Integer> future = executor.submit(() -> compute());
Integer result = future.get();  // Blocks
executor.shutdown();
```

### Thread Pools

```java
Executors.newFixedThreadPool(n)        // Fixed size
Executors.newCachedThreadPool()        // Grows as needed
Executors.newSingleThreadExecutor()    // Single thread
Executors.newScheduledThreadPool(n)    // Scheduled tasks
```

### Synchronization

```java
// Synchronized method
public synchronized void method() { }

// Synchronized block
synchronized (lockObject) { }

// ReentrantLock
Lock lock = new ReentrantLock();
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();
}

// ReadWriteLock
ReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.readLock().lock();    // Multiple readers
rwLock.writeLock().lock();   // Exclusive writer
```

### Atomic Variables

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();             // ++counter
counter.getAndIncrement();             // counter++
counter.addAndGet(5);                  // counter += 5
counter.compareAndSet(expected, new);  // CAS
```

### Concurrent Collections

```java
ConcurrentHashMap<K, V>                // Thread-safe HashMap
CopyOnWriteArrayList<E>                // Thread-safe ArrayList (read-heavy)
ConcurrentLinkedQueue<E>               // Thread-safe queue
BlockingQueue<E>                       // Producer-consumer
    - ArrayBlockingQueue               // Bounded
    - LinkedBlockingQueue              // Optionally bounded
    - PriorityBlockingQueue            // Priority-based
```

### volatile vs synchronized

| Feature | volatile | synchronized |
|---------|----------|--------------|
| Visibility | Yes | Yes |
| Atomicity | Only single read/write | Yes |
| Blocking | No | Yes |
| Use case | Flags | Compound operations |

### CompletableFuture

```java
CompletableFuture.supplyAsync(() -> compute())
    .thenApply(result -> transform(result))
    .thenAccept(System.out::println)
    .exceptionally(ex -> fallback);

// Combine
CompletableFuture.allOf(cf1, cf2, cf3).join();
CompletableFuture.anyOf(cf1, cf2, cf3);
cf1.thenCombine(cf2, (r1, r2) -> combine(r1, r2));
```

---

## 5) JVM & Memory Cheat Sheet

### Memory Areas

| Area | Scope | Stores |
|------|-------|--------|
| Heap | Shared | Objects |
| Stack | Per thread | Primitives, references, frames |
| Metaspace | Shared | Class metadata |
| PC Register | Per thread | Current instruction |

### Heap Generations

```
Young Generation (Minor GC)
├── Eden          - New objects
├── Survivor S0   - Survivors
└── Survivor S1   - Survivors

Old Generation (Major GC)
└── Tenured       - Long-lived objects
```

### GC Algorithms

| Collector | Best For | Pause |
|-----------|----------|-------|
| Serial | Single core, small heap | Stop-the-world |
| Parallel | Throughput, batch | Stop-the-world |
| G1 | Large heap, predictable pause | Mostly concurrent |
| ZGC | Ultra-low latency | < 10ms |

### Common JVM Flags

```bash
-Xms512m                    # Initial heap
-Xmx2g                      # Maximum heap
-Xss256k                    # Stack size
-XX:+UseG1GC                # Use G1 collector
-XX:MaxGCPauseMillis=200    # Target pause time
-XX:+HeapDumpOnOutOfMemoryError
-Xlog:gc*                   # GC logging
```

### Reference Types

| Type | Collected | Use Case |
|------|-----------|----------|
| Strong | Never (unless unreachable) | Normal references |
| Soft | On memory pressure | Caches |
| Weak | On any GC | Canonicalizing maps |
| Phantom | After finalization | Cleanup |

---

## 6) OOP & SOLID Cheat Sheet

### OOP Pillars

| Pillar | Description |
|--------|-------------|
| **Encapsulation** | Hide internal state, expose behavior |
| **Abstraction** | Model essential features, hide complexity |
| **Inheritance** | IS-A relationship, code reuse |
| **Polymorphism** | Same interface, different behavior |

### SOLID Principles

| Principle | Summary |
|-----------|---------|
| **S**ingle Responsibility | One class, one reason to change |
| **O**pen/Closed | Open for extension, closed for modification |
| **L**iskov Substitution | Subtypes must be substitutable |
| **I**nterface Segregation | Prefer small, specific interfaces |
| **D**ependency Inversion | Depend on abstractions, not concretions |

### Interface vs Abstract Class

| Feature | Interface | Abstract Class |
|---------|-----------|----------------|
| Multiple inheritance | Yes | No |
| State | No (except constants) | Yes |
| Constructor | No | Yes |
| Default methods | Yes (Java 8+) | Yes |
| Use when | Define capability | Share code/state |

---

## 7) Exceptions Cheat Sheet

### Hierarchy

```
Throwable
├── Error (don't catch)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception
    ├── Checked (must handle)
    │   ├── IOException
    │   └── SQLException
    └── RuntimeException (unchecked)
        ├── NullPointerException
        ├── IllegalArgumentException
        └── IndexOutOfBoundsException
```

### Best Practices

```java
// Try-with-resources (always use for closeable)
try (InputStream is = new FileInputStream("file")) {
    // Auto-closed
}

// Don't catch generic Exception
try {
    // ...
} catch (SpecificException e) {
    // Handle specifically
}

// Preserve cause
throw new CustomException("message", cause);

// Don't swallow exceptions
try {
    // ...
} catch (Exception e) {
    log.error("Error", e);  // At minimum log
    throw e;                // Or rethrow
}
```

---

## 8) Design Patterns Cheat Sheet

### Creational

| Pattern | Intent | When |
|---------|--------|------|
| **Singleton** | One instance | Configuration, logging |
| **Factory** | Create without specifying class | Multiple implementations |
| **Builder** | Complex construction | Many parameters |
| **Prototype** | Clone existing objects | Expensive creation |

### Structural

| Pattern | Intent | When |
|---------|--------|------|
| **Adapter** | Convert interface | Legacy integration |
| **Decorator** | Add responsibilities | Dynamic features |
| **Facade** | Simplify complex subsystem | API simplification |
| **Proxy** | Control access | Lazy loading, security |

### Behavioral

| Pattern | Intent | When |
|---------|--------|------|
| **Strategy** | Interchangeable algorithms | Payment methods |
| **Observer** | Notify on state change | Event handling |
| **Template Method** | Algorithm skeleton | Framework hooks |
| **Command** | Encapsulate request | Undo/redo, queues |

---

## 9) Spring Boot Cheat Sheet

### Common Annotations

| Annotation | Purpose |
|------------|---------|
| `@SpringBootApplication` | Main class (combines 3 annotations) |
| `@RestController` | REST controller |
| `@Service` | Service layer |
| `@Repository` | Data access layer |
| `@Component` | Generic bean |
| `@Autowired` | Dependency injection |
| `@Value` | Inject property |
| `@Configuration` | Configuration class |
| `@Bean` | Define bean in config |

### Request Mappings

| Annotation | HTTP Method |
|------------|-------------|
| `@GetMapping` | GET |
| `@PostMapping` | POST |
| `@PutMapping` | PUT |
| `@DeleteMapping` | DELETE |
| `@PatchMapping` | PATCH |

### Request Parameters

```java
@PathVariable Long id                  // /users/{id}
@RequestParam String name              // ?name=value
@RequestBody User user                 // JSON body
@RequestHeader String auth             // Header
```

### Bean Scopes

| Scope | Description |
|-------|-------------|
| `singleton` | One per container (default) |
| `prototype` | New per injection |
| `request` | One per HTTP request |
| `session` | One per HTTP session |

### Transaction Propagation

| Type | Behavior |
|------|----------|
| `REQUIRED` | Join or create (default) |
| `REQUIRES_NEW` | Always create new |
| `NESTED` | Nested with savepoint |
| `SUPPORTS` | Use existing or none |

---

## 10) Testing Cheat Sheet

### JUnit 5 Annotations

| Annotation | Purpose |
|------------|---------|
| `@Test` | Test method |
| `@BeforeEach` | Setup before each test |
| `@AfterEach` | Cleanup after each test |
| `@BeforeAll` | Setup once (static) |
| `@AfterAll` | Cleanup once (static) |
| `@Disabled` | Skip test |
| `@ParameterizedTest` | Parameterized test |
| `@DisplayName` | Custom test name |

### Assertions

```java
assertEquals(expected, actual);
assertNotEquals(unexpected, actual);
assertTrue(condition);
assertFalse(condition);
assertNull(value);
assertNotNull(value);
assertThrows(Exception.class, () -> method());
assertAll(() -> {}, () -> {});
```

### Mockito

```java
// Create mock
@Mock UserRepository repo;

// Stub
when(repo.findById(1L)).thenReturn(Optional.of(user));
when(repo.save(any())).thenAnswer(inv -> inv.getArgument(0));
doThrow(Exception.class).when(repo).delete(any());

// Verify
verify(repo).save(any());
verify(repo, times(2)).findById(anyLong());
verify(repo, never()).delete(any());

// Argument capture
ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
verify(repo).save(captor.capture());
User saved = captor.getValue();
```

---

## 11) Quick Interview Checklist

### Before the Interview

- [ ] Review equals/hashCode contract
- [ ] Know when to use ArrayList vs LinkedList
- [ ] Understand HashMap internals
- [ ] Practice explaining Stream operations
- [ ] Review thread synchronization
- [ ] Know singleton implementation patterns
- [ ] Understand @Transactional behavior
- [ ] Review common Big-O complexities

### During the Interview

- [ ] Clarify requirements before coding
- [ ] Think aloud while problem-solving
- [ ] Consider edge cases
- [ ] Discuss time/space complexity
- [ ] Mention trade-offs in design decisions
- [ ] Ask about existing code style/patterns
- [ ] Be honest about what you don't know

### Red Flags to Avoid

- ❌ Field injection (`@Autowired` on fields)
- ❌ Catching generic `Exception`
- ❌ Swallowing exceptions silently
- ❌ Not using try-with-resources
- ❌ Mutable objects as HashMap keys
- ❌ Not specifying character encoding
- ❌ Using `==` for String comparison
- ❌ Not handling `null` properly

---

## 12) Common Interview Questions (One-Liners)

| Question | Answer |
|----------|--------|
| Why is String immutable? | Security, caching, thread-safety, string pool |
| HashMap vs Hashtable? | HashMap: not synchronized, allows null. Hashtable: legacy, synchronized |
| ArrayList vs LinkedList? | ArrayList: fast access O(1). LinkedList: fast insert/delete O(1) at known position |
| Checked vs Unchecked? | Checked: compile-time handling. Unchecked: runtime errors |
| Interface vs Abstract? | Interface: contract, multiple inheritance. Abstract: partial implementation |
| volatile vs synchronized? | volatile: visibility only. synchronized: visibility + atomicity |
| == vs equals()? | ==: reference/primitive equality. equals(): logical equality |
| final vs finally vs finalize? | final: constant. finally: cleanup block. finalize: GC callback (deprecated) |
| Stack vs Heap? | Stack: per-thread, primitives. Heap: shared, objects |
| What is GC? | Automatic memory reclamation for unreachable objects |

---

*Good luck with your interview! 🍀*

