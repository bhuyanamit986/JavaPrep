# JVM, Memory Model & Garbage Collection

Understanding the JVM internals is crucial for Java interviews—especially for senior positions. This guide covers memory architecture, class loading, and garbage collection.

## Definitions

- **JVM**: the runtime that executes Java bytecode and manages memory, threads, and execution.
- **Class loader**: component that loads classes into the JVM (loading, linking, initialization).
- **Heap vs stack**: heap stores objects shared across threads; stack stores per-thread call frames.
- **Garbage collection (GC)**: automatic memory reclamation for unreachable objects.
- **Generational GC**: strategy that treats young and old objects differently to optimize collection.
- **JIT compiler**: runtime compiler that optimizes "hot" bytecode into native code.
- **Metaspace**: native memory area where class metadata is stored (replaces PermGen).

## Illustrations

- **Stack frames**: like a stack of plates; each method call adds a plate, returns remove it.
- **Young vs old generation**: a nursery for short-lived objects and a retirement home for long-lived ones.
- **GC**: a cleaning crew that clears empty rooms so new guests can move in.

## Code Examples

```java
Runtime rt = Runtime.getRuntime();
long used = rt.totalMemory() - rt.freeMemory();
System.out.println("Used memory bytes: " + used);
```

## Interview Questions

1. What is stored on the heap vs the stack?
2. What is the role of the class loader?
3. Why do generational collectors work well in practice?
4. What is a stop-the-world pause?
5. How does JIT compilation improve performance?
6. What causes memory leaks in Java?

---

## 1) JVM Architecture Overview

The Java Virtual Machine (JVM) is an abstract computing machine that enables Java bytecode execution.

### JVM Components

```
┌──────────────────────────────────────────────────────────────────┐
│                          JVM                                      │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    Class Loader Subsystem                    │ │
│  │   ┌───────────┐  ┌───────────┐  ┌───────────────────────┐   │ │
│  │   │  Loading  │→ │  Linking  │→ │    Initialization     │   │ │
│  │   └───────────┘  └───────────┘  └───────────────────────┘   │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              ↓                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    Runtime Data Areas                        │ │
│  │   ┌────────┐ ┌─────────┐ ┌────────────────────────────────┐ │ │
│  │   │ Method │ │  Heap   │ │   Per-Thread:                  │ │ │
│  │   │  Area  │ │         │ │   • PC Register                │ │ │
│  │   │(shared)│ │(shared) │ │   • JVM Stack                  │ │ │
│  │   └────────┘ └─────────┘ │   • Native Method Stack        │ │ │
│  │                          └────────────────────────────────┘ │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              ↓                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    Execution Engine                          │ │
│  │   ┌─────────────┐  ┌───────────┐  ┌─────────────────────┐   │ │
│  │   │ Interpreter │  │    JIT    │  │  Garbage Collector  │   │ │
│  │   │             │  │  Compiler │  │                     │   │ │
│  │   └─────────────┘  └───────────┘  └─────────────────────┘   │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              ↓                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              Native Method Interface (JNI)                   │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2) JVM Memory Areas

### Heap Memory (Shared)

The heap is where **all objects** are allocated. It's shared across all threads and is the main area managed by Garbage Collection.

**Heap Structure (Generational)**:

```
┌─────────────────────────────────────────────────────────────┐
│                         Heap                                 │
│  ┌─────────────────────────────────────┐ ┌───────────────┐  │
│  │          Young Generation           │ │     Old       │  │
│  │  ┌───────┐ ┌───────┐ ┌───────┐     │ │  Generation   │  │
│  │  │ Eden  │ │  S0   │ │  S1   │     │ │   (Tenured)   │  │
│  │  │       │ │(from) │ │ (to)  │     │ │               │  │
│  │  └───────┘ └───────┘ └───────┘     │ │               │  │
│  │         Survivor Spaces             │ │               │  │
│  └─────────────────────────────────────┘ └───────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

| Area | Description | GC Type |
|------|-------------|---------|
| **Eden** | New objects allocated here | Minor GC |
| **Survivor (S0, S1)** | Objects surviving minor GC | Minor GC |
| **Old Generation** | Long-lived objects | Major/Full GC |

**Default Heap Sizing**:
- `-Xms`: Initial heap size
- `-Xmx`: Maximum heap size
- `-Xmn`: Young generation size

### Metaspace (Java 8+)

Replaced **PermGen** (Pre-Java 8). Stores:
- Class metadata
- Method metadata
- Constant pools
- Annotations

```
# Metaspace grows automatically but can be limited
-XX:MetaspaceSize=128m        # Initial size
-XX:MaxMetaspaceSize=256m     # Maximum size
```

**Key difference from PermGen**:
- Metaspace uses **native memory** (not JVM heap)
- Can grow dynamically (less `OutOfMemoryError: PermGen space`)

### Stack Memory (Per Thread)

Each thread has its own **JVM Stack** containing:
- **Stack Frames**: One per method call
  - Local variables
  - Operand stack
  - Frame data (return address, exception info)

```
Thread Stack
┌─────────────────────┐
│ Frame: method3()    │ ← Current frame (top)
│   - local vars      │
│   - operand stack   │
├─────────────────────┤
│ Frame: method2()    │
├─────────────────────┤
│ Frame: method1()    │
├─────────────────────┤
│ Frame: main()       │ ← Bottom of stack
└─────────────────────┘
```

**Configuration**:
```
-Xss512k    # Stack size per thread (default ~1MB)
```

**StackOverflowError**: Deep recursion exhausts stack space.

### PC Register (Per Thread)

Holds address of current instruction being executed.

### Native Method Stack (Per Thread)

For native (C/C++) method calls via JNI.

---

## 3) Stack vs Heap

| Feature | Stack | Heap |
|---------|-------|------|
| **Scope** | Per thread | Shared across threads |
| **Stores** | Primitives, references, frames | Objects |
| **Access** | LIFO (fast) | Dynamic (slower) |
| **Size** | Fixed (small) | Large, growable |
| **Thread-safe** | Yes (isolated) | No (needs synchronization) |
| **Allocation** | Automatic (method entry/exit) | `new` keyword |
| **Deallocation** | Automatic (method exit) | Garbage Collection |
| **Errors** | `StackOverflowError` | `OutOfMemoryError` |

### Memory Example

```java
public void example() {
    int x = 10;              // Primitive → Stack
    String s = "hello";      // Reference → Stack, String object → Heap (String pool)
    Person p = new Person(); // Reference → Stack, Person object → Heap
}
```

---

## 4) Object Layout in Memory

### Object Header

Every Java object has a header (typically 12-16 bytes on 64-bit):

| Component | Size | Contents |
|-----------|------|----------|
| **Mark Word** | 8 bytes | HashCode, GC age, lock state, biased locking info |
| **Class Pointer** | 4 bytes* | Pointer to class metadata |
| **Array Length** | 4 bytes | Only for arrays |

*With compressed oops (default in most cases)

### Object Fields

Fields are typically aligned and ordered to minimize padding:

```java
class Example {
    boolean flag;  // 1 byte + 7 padding
    long value;    // 8 bytes
    int count;     // 4 bytes
    // Object size: 12 (header) + 8 + 8 + 4 + 4 (padding) = ~36 bytes
}
```

### Compressed OOPs (Ordinary Object Pointers)

For heaps < 32GB, JVM compresses 64-bit references to 32-bit.

```
-XX:+UseCompressedOops    # Enabled by default when heap < 32GB
-XX:-UseCompressedOops    # Disable
```

---

## 5) Class Loading

### Class Loader Hierarchy

```
Bootstrap ClassLoader (C/C++)
         ↓
   Extension ClassLoader (Platform)
         ↓
   Application ClassLoader (System)
         ↓
   Custom ClassLoaders
```

| ClassLoader | Loads From |
|-------------|------------|
| **Bootstrap** | `rt.jar`, core classes (`java.lang.*`) |
| **Extension/Platform** | `lib/ext`, `jre/lib/ext` |
| **Application/System** | Classpath (`-cp`, `CLASSPATH`) |

### Class Loading Process

1. **Loading**: Find `.class` file, load bytes
2. **Linking**:
   - **Verification**: Verify bytecode correctness
   - **Preparation**: Allocate static fields, set defaults
   - **Resolution**: Resolve symbolic references (optional lazy)
3. **Initialization**: Execute `<clinit>` (static blocks, static field initializers)

### Delegation Model (Parent-First)

```java
Class<?> loadClass(String name) {
    // 1. Check if already loaded
    Class<?> c = findLoadedClass(name);
    if (c != null) return c;
    
    // 2. Delegate to parent
    try {
        c = parent.loadClass(name);
        return c;
    } catch (ClassNotFoundException e) {
        // 3. Try to load ourselves
        return findClass(name);
    }
}
```

**Why parent-first?**
- Security: Core classes can't be spoofed
- Avoids duplicate loading
- Consistent class identity

### Class Unloading

Classes can be unloaded when:
- ClassLoader is unreachable
- No instances of the class exist
- Class object itself is unreachable

**Common in**: Application servers with hot deployment.

### Common Issues

**ClassNotFoundException**: Class not found on classpath.

**NoClassDefFoundError**: Class was present at compile time but missing at runtime, or initialization failed.

**LinkageError/ClassCastException**: Same class loaded by different classloaders (considered different classes!).

---

## 6) Garbage Collection Concepts

### What is GC?

Automatic memory management that reclaims memory from objects no longer reachable.

### Object Reachability

Objects are **reachable** if accessible from **GC roots**:

- Local variables in active threads
- Static fields
- JNI references
- Active threads themselves

### Reference Types

| Type | Collected When | Use Case |
|------|----------------|----------|
| **Strong** | Never (unless unreachable) | Normal references |
| **Soft** | Memory pressure | Caches |
| **Weak** | Any GC cycle | Canonicalizing mappings |
| **Phantom** | After finalization | Cleanup actions |

```java
// Strong reference
Object strong = new Object();

// Soft reference (cleared before OOM)
SoftReference<Object> soft = new SoftReference<>(new Object());

// Weak reference (cleared at next GC)
WeakReference<Object> weak = new WeakReference<>(new Object());

// Phantom reference (for post-mortem cleanup)
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantom = new PhantomReference<>(new Object(), queue);
```

### Generational Hypothesis

**Observation**: Most objects die young.

**Strategy**: Divide heap into generations:
- **Young Gen**: Frequent, fast GC (Minor GC)
- **Old Gen**: Less frequent, slower GC (Major GC)

Objects that survive multiple minor GCs get **promoted** to old generation.

### GC Terminology

| Term | Meaning |
|------|---------|
| **Minor GC** | Collects young generation |
| **Major GC** | Collects old generation |
| **Full GC** | Collects entire heap (young + old + metaspace) |
| **Stop-the-World (STW)** | Application pauses during GC |
| **Concurrent** | GC runs alongside application |
| **Throughput** | % time spent on application (vs GC) |
| **Latency** | GC pause duration |

---

## 7) GC Algorithms

### Mark and Sweep

1. **Mark**: Traverse from GC roots, mark reachable objects
2. **Sweep**: Remove unmarked objects, reclaim memory

**Problem**: Memory fragmentation.

### Mark-Sweep-Compact

1. **Mark**: Same as above
2. **Sweep**: Remove unmarked objects
3. **Compact**: Move live objects together, eliminating gaps

**Benefit**: No fragmentation.
**Cost**: Additional time for compaction.

### Copying Collection

1. Divide space into two halves (from-space, to-space)
2. Copy live objects to to-space
3. Clear from-space
4. Swap roles

**Used in**: Young generation (Survivor spaces S0/S1).

**Benefit**: Fast allocation, no fragmentation.
**Cost**: Uses only half the space.

---

## 8) Garbage Collectors

### Serial GC

```
-XX:+UseSerialGC
```

- Single-threaded
- Stop-the-world
- Best for: Small heaps, single-core machines, client applications

### Parallel GC (Throughput Collector)

```
-XX:+UseParallelGC
```

- Multi-threaded young and old gen collection
- Stop-the-world
- Optimizes for **throughput**
- Best for: Batch processing, backend jobs

### CMS (Concurrent Mark Sweep) [Deprecated in Java 9, Removed in Java 14]

```
-XX:+UseConcMarkSweepGC
```

- Mostly concurrent (minimal STW)
- No compaction (can fragment)
- Best for: Low-latency applications

**Phases**:
1. Initial Mark (STW) - Mark GC roots
2. Concurrent Mark - Traverse object graph
3. Remark (STW) - Handle changes during concurrent mark
4. Concurrent Sweep - Reclaim memory

### G1 GC (Garbage First)

```
-XX:+UseG1GC    # Default since Java 9
```

- Divides heap into **regions** (not contiguous generations)
- Prioritizes regions with most garbage ("Garbage First")
- Predictable pause times
- Concurrent and parallel

**Heap Layout**:
```
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ E │ E │ S │ O │ O │ H │ H │ E │
└───┴───┴───┴───┴───┴───┴───┴───┘
E = Eden, S = Survivor, O = Old, H = Humongous (large objects)
```

**G1 Phases**:
1. Young GC (STW) - Evacuate Eden/Survivors
2. Concurrent Marking - Mark live objects
3. Mixed GC (STW) - Collect young + some old regions

**Tuning**:
```
-XX:MaxGCPauseMillis=200     # Target pause time
-XX:G1HeapRegionSize=4m      # Region size (1-32MB)
```

### ZGC (Z Garbage Collector) [Java 11+]

```
-XX:+UseZGC
```

- Ultra-low latency (< 10ms pauses)
- Scalable to TB-sized heaps
- Concurrent compaction
- Colored pointers + load barriers

Best for: Applications requiring consistent low latency.

### Shenandoah GC [Java 12+]

```
-XX:+UseShenandoahGC
```

- Similar goals to ZGC
- Concurrent compaction
- Lower pause times

---

## 9) GC Tuning Basics

### Common JVM Flags

```bash
# Heap sizing
-Xms2g              # Initial heap
-Xmx4g              # Maximum heap
-Xmn1g              # Young generation size

# Generation ratios
-XX:NewRatio=2      # Old:Young = 2:1
-XX:SurvivorRatio=8 # Eden:Survivor = 8:1

# GC selection
-XX:+UseG1GC
-XX:+UseZGC
-XX:+UseParallelGC

# GC logging (modern syntax)
-Xlog:gc*:file=gc.log:time,uptime,level,tags

# GC logging (Java 8)
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:gc.log
```

### Tuning Goals

| Goal | Strategy |
|------|----------|
| **High throughput** | Larger heap, Parallel GC |
| **Low latency** | G1/ZGC, smaller regions, shorter pauses |
| **Small footprint** | Serial GC, minimal heap |

### Monitoring Tools

| Tool | Purpose |
|------|---------|
| `jstat` | GC statistics |
| `jmap` | Heap dumps |
| `jcmd` | Diagnostic commands |
| `jvisualvm` | Visual monitoring |
| `JFR` | Java Flight Recorder |
| GC Logs | Detailed GC behavior |

```bash
# GC statistics
jstat -gc <pid> 1000 10  # Every 1s, 10 times

# Heap dump
jmap -dump:format=b,file=heap.hprof <pid>

# Force GC
jcmd <pid> GC.run
```

---

## 10) Memory Leaks

### What is a Memory Leak in Java?

Objects that are no longer needed but are still reachable (can't be collected).

### Common Causes

**1. Static Collections**

```java
// LEAK: Objects added but never removed
private static final List<Object> cache = new ArrayList<>();

public void process(Object obj) {
    cache.add(obj);  // Never cleared!
}
```

**2. Unclosed Resources**

```java
// LEAK: Connection never returned to pool
public void query() {
    Connection conn = dataSource.getConnection();
    // No finally block to close
    ResultSet rs = conn.executeQuery("...");
}
```

**3. Inner Classes Holding Outer Reference**

```java
class Outer {
    private byte[] largeData = new byte[10_000_000];
    
    public Runnable createTask() {
        return new Runnable() {  // Holds reference to Outer.this
            public void run() {
                // Even if largeData isn't used, Outer can't be GC'd
            }
        };
    }
}

// FIX: Use static inner class or lambda
```

**4. Listeners/Callbacks Not Removed**

```java
// LEAK: Listeners accumulate
button.addActionListener(listener);
// Never: button.removeActionListener(listener);
```

**5. ThreadLocal Not Cleaned**

```java
// LEAK in thread pools
private static ThreadLocal<HeavyObject> cache = new ThreadLocal<>();

public void handle() {
    cache.set(new HeavyObject());
    // Never: cache.remove();
}
```

**6. Custom Key Class Without Proper equals/hashCode**

```java
// LEAK: Can't find entries to remove
Map<BadKey, Value> map = new HashMap<>();

class BadKey {
    int id;
    // No equals() or hashCode() override!
}
```

### Detection Strategies

1. **Monitor heap growth** over time
2. **Analyze heap dumps** with MAT (Memory Analyzer Tool)
3. **Look for**:
   - Growing collections
   - Unexpected object retention
   - Dominator tree anomalies

### Prevention Best Practices

```java
// 1. Use try-with-resources
try (Connection conn = dataSource.getConnection()) {
    // Auto-closed
}

// 2. Use weak references for caches
Map<Key, SoftReference<Value>> cache = new ConcurrentHashMap<>();

// 3. Clear ThreadLocals
try {
    threadLocal.set(value);
    // use
} finally {
    threadLocal.remove();
}

// 4. Prefer lambdas over anonymous inner classes
Runnable task = () -> doWork();  // Doesn't capture outer this

// 5. Use bounded caches
Cache<Key, Value> cache = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build();
```

---

## 11) JIT Compilation

### Interpreter vs JIT

| Aspect | Interpreter | JIT Compiler |
|--------|-------------|--------------|
| **Startup** | Fast | Slow (compilation overhead) |
| **Runtime** | Slower | Faster (native code) |
| **Memory** | Lower | Higher (code cache) |

### JIT Tiers (Tiered Compilation)

1. **Level 0**: Interpreter
2. **Level 1-3**: C1 compiler (fast compilation, basic optimizations)
3. **Level 4**: C2 compiler (slow compilation, aggressive optimizations)

### Common JIT Optimizations

| Optimization | Description |
|--------------|-------------|
| **Inlining** | Replace method call with method body |
| **Escape Analysis** | Avoid heap allocation if object doesn't escape |
| **Loop Unrolling** | Reduce loop overhead |
| **Dead Code Elimination** | Remove unreachable code |
| **Lock Elision** | Remove unnecessary synchronization |
| **Devirtualization** | Convert virtual calls to direct calls |

### JIT Flags

```bash
# View compiled methods
-XX:+PrintCompilation

# Code cache size
-XX:ReservedCodeCacheSize=256m

# Disable JIT (debugging)
-Xint

# Force all methods to compile
-Xcomp

# Compilation threshold
-XX:CompileThreshold=10000
```

---

## 12) String Pool and Interning

### String Pool

String literals are stored in a special pool in heap memory (moved from PermGen in Java 7).

```java
String s1 = "hello";    // Pool
String s2 = "hello";    // Same reference from pool
String s3 = new String("hello");  // New object on heap

s1 == s2;   // true (same pooled reference)
s1 == s3;   // false (different objects)
s1.equals(s3);  // true (same content)

String s4 = s3.intern();  // Get pooled version
s1 == s4;   // true
```

### intern() Method

Forces a string into the pool:

```java
String computed = new StringBuilder("hel").append("lo").toString();
String interned = computed.intern();

"hello" == interned;  // true
```

**Use case**: Reducing memory for many duplicate strings.

**Caution**: Pool lookups have overhead; don't intern everything.

---

## 13) Native Memory

Memory outside the JVM heap:

- **Metaspace**: Class metadata
- **Code Cache**: JIT-compiled code
- **Direct ByteBuffers**: NIO direct buffers
- **Thread Stacks**: Native thread memory
- **JNI**: Native library memory

### Tracking Native Memory

```bash
# Enable native memory tracking
-XX:NativeMemoryTracking=summary  # or 'detail'

# Print summary
jcmd <pid> VM.native_memory summary
```

### Direct ByteBuffers

```java
// Heap buffer (backed by byte array)
ByteBuffer heap = ByteBuffer.allocate(1024);

// Direct buffer (native memory)
ByteBuffer direct = ByteBuffer.allocateDirect(1024);
```

**Direct buffers**:
- Faster I/O (no copy between JVM and OS)
- More expensive allocation/deallocation
- Not managed by GC (only the reference is)
- Can cause `OutOfMemoryError` even with free heap

---

## 14) Profiling and Diagnostics

### Key Tools

| Tool | Purpose | Command |
|------|---------|---------|
| `jps` | List Java processes | `jps -l` |
| `jstat` | GC statistics | `jstat -gcutil <pid> 1000` |
| `jmap` | Memory maps, heap dump | `jmap -heap <pid>` |
| `jstack` | Thread dump | `jstack <pid>` |
| `jcmd` | Diagnostic commands | `jcmd <pid> help` |
| `jconsole` | GUI monitoring | `jconsole` |
| `jvisualvm` | Visual profiler | `jvisualvm` |

### Java Flight Recorder (JFR)

Low-overhead profiling built into the JVM:

```bash
# Start recording
jcmd <pid> JFR.start name=recording1 duration=60s filename=rec.jfr

# Continuous recording
-XX:StartFlightRecording=duration=60s,filename=rec.jfr
```

### Useful jcmd Commands

```bash
jcmd <pid> VM.flags              # All JVM flags
jcmd <pid> VM.system_properties  # System properties
jcmd <pid> GC.heap_info          # Heap information
jcmd <pid> Thread.print          # Thread dump
jcmd <pid> GC.run                # Force GC
jcmd <pid> VM.native_memory summary  # Native memory
```

---

## 15) Interview Questions

### JVM Memory
1. Explain the different memory areas in JVM.
2. What is the difference between stack and heap?
3. What is Metaspace? How is it different from PermGen?
4. What happens when you create an object with `new`?

### Class Loading
5. Explain the class loader hierarchy.
6. What is the parent delegation model? Why is it used?
7. When is a class loaded? When is it initialized?
8. What is the difference between `ClassNotFoundException` and `NoClassDefFoundError`?

### Garbage Collection
9. What is garbage collection? Why do we need it?
10. Explain the generational hypothesis.
11. What are GC roots?
12. What is the difference between Minor GC and Major GC?
13. Explain different reference types (Strong, Soft, Weak, Phantom).

### GC Algorithms and Collectors
14. Explain Mark and Sweep algorithm.
15. What is Stop-the-World? How does it affect applications?
16. Compare Serial, Parallel, CMS, and G1 collectors.
17. When would you use ZGC or Shenandoah?
18. How does G1 GC work?

### Tuning and Troubleshooting
19. How would you tune GC for low latency vs high throughput?
20. What JVM flags would you use to diagnose memory issues?
21. How do you detect memory leaks in Java?
22. What tools do you use for JVM monitoring?
23. Explain how to analyze a heap dump.

### Advanced
24. What is JIT compilation? What optimizations does it perform?
25. What is escape analysis?
26. Explain the String pool and interning.
27. What is native memory? When is it used?
28. How do direct ByteBuffers differ from heap buffers?

---

## 16) Quick Reference: JVM Flags

```bash
# Memory
-Xms4g                    # Initial heap
-Xmx8g                    # Maximum heap
-Xmn2g                    # Young generation
-Xss1m                    # Thread stack size
-XX:MetaspaceSize=256m    # Initial metaspace
-XX:MaxMetaspaceSize=512m # Max metaspace

# GC Selection
-XX:+UseG1GC              # G1 (default Java 9+)
-XX:+UseZGC               # ZGC (Java 11+)
-XX:+UseParallelGC        # Parallel
-XX:+UseShenandoahGC      # Shenandoah

# GC Tuning
-XX:MaxGCPauseMillis=200  # Target pause (G1)
-XX:NewRatio=2            # Old:Young ratio
-XX:SurvivorRatio=8       # Eden:Survivor ratio
-XX:+UseStringDeduplication  # G1 string dedup

# GC Logging (Java 9+)
-Xlog:gc*:file=gc.log:time,uptime,level,tags

# Diagnostics
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/dump.hprof
-XX:OnOutOfMemoryError="kill -9 %p"

# Performance
-XX:+UseCompressedOops    # Compressed pointers
-XX:ReservedCodeCacheSize=256m
-XX:+TieredCompilation    # Default on

# Debugging
-XX:+PrintFlagsFinal      # Show all flags
-XX:NativeMemoryTracking=summary
```

---

## 17) Memory Troubleshooting Flowchart

```
OutOfMemoryError
       │
       ├── "Java heap space"
       │   └── Increase -Xmx, check for leaks
       │
       ├── "GC overhead limit exceeded"
       │   └── GC taking >98% time, reclaiming <2%
       │   └── Check for memory leak, increase heap
       │
       ├── "Metaspace"
       │   └── Too many classes, classloader leak
       │   └── Increase -XX:MaxMetaspaceSize
       │
       ├── "Direct buffer memory"
       │   └── NIO ByteBuffer.allocateDirect()
       │   └── Increase -XX:MaxDirectMemorySize
       │
       ├── "Unable to create new native thread"
       │   └── Too many threads, OS limit
       │   └── Reduce -Xss, increase ulimit
       │
       └── "Requested array size exceeds VM limit"
           └── Trying to allocate huge array
```

