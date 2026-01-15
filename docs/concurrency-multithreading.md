# Concurrency & Multithreading

This chapter covers one of the most challenging topics in Java - concurrent programming. Understanding threads, synchronization, and the Java Memory Model is essential for building scalable applications and acing senior-level interviews.

---

## Definitions

### Core Concepts

- **Thread**: A lightweight unit of execution within a process. Each thread has its own stack but shares the heap with other threads.

- **Process**: An independent program in execution with its own memory space. Multiple processes are isolated from each other.

- **Concurrency**: Multiple tasks making progress within overlapping time periods. Tasks may not execute simultaneously but appear to run concurrently.

- **Parallelism**: Multiple tasks executing literally at the same time on multiple CPU cores.

- **Race Condition**: A bug where program behavior depends on the relative timing of events. Multiple threads access shared data and at least one modifies it.

- **Critical Section**: A code segment that accesses shared resources and must be executed atomically.

- **Mutual Exclusion (Mutex)**: Ensuring only one thread can access a critical section at a time.

- **Deadlock**: A situation where two or more threads are blocked forever, each waiting for resources held by the other.

- **Livelock**: Threads actively trying to resolve a conflict but making no progress.

- **Starvation**: A thread is unable to gain regular access to shared resources because other threads monopolize them.

### Synchronization Mechanisms

- **synchronized**: Java keyword for intrinsic locking. Provides mutual exclusion and memory visibility.

- **Lock**: Explicit locking mechanism (java.util.concurrent.locks) with more flexibility than synchronized.

- **volatile**: Keyword ensuring visibility of changes across threads. Reads and writes go directly to main memory.

- **Atomic**: Operations that complete entirely without interruption. Atomic classes use CAS for lock-free thread safety.

### Java Memory Model (JMM)

- **Happens-before**: A guarantee that memory writes by one thread are visible to another. Establishes ordering between operations.

- **Memory Visibility**: The guarantee that one thread's changes to shared data are visible to other threads.

- **Memory Barrier/Fence**: Instructions that ensure memory operations are completed in order.

---

## Illustrations

### Thread Lifecycle

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Thread Lifecycle                                 │
│                                                                          │
│                          ┌─────────┐                                     │
│                          │   NEW   │                                     │
│                          └────┬────┘                                     │
│                               │ start()                                  │
│                               ▼                                          │
│                        ┌──────────────┐                                  │
│              ┌────────►│   RUNNABLE   │◄────────┐                        │
│              │         └──────┬───────┘         │                        │
│              │                │                 │                        │
│              │           Scheduler              │                        │
│              │           picks thread           │                        │
│              │                │                 │                        │
│              │                ▼                 │                        │
│              │         ┌──────────────┐         │                        │
│              │         │   RUNNING    │         │                        │
│              │         └──────┬───────┘         │                        │
│              │                │                 │                        │
│    ┌─────────┴────────────────┼─────────────────┴────────────┐           │
│    │                          │                              │           │
│    │ notify()/notifyAll()     │ wait()                       │           │
│    │ Lock acquired            │ synchronized                 │           │
│    │ I/O complete             │ I/O blocked                  │           │
│    │ sleep() timeout          │ sleep()                      │           │
│    │                          │                              │           │
│    ▼                          ▼                              ▼           │
│ ┌──────────┐            ┌──────────┐                  ┌──────────┐       │
│ │ WAITING  │            │ BLOCKED  │                  │ TIMED_   │       │
│ │          │            │          │                  │ WAITING  │       │
│ └──────────┘            └──────────┘                  └──────────┘       │
│                                                                          │
│                          ┌──────────────┐                                │
│                          │  TERMINATED  │                                │
│                          │  (Dead)      │                                │
│                          └──────────────┘                                │
│                                                                          │
│  State Descriptions:                                                     │
│  ┌────────────────────────────────────────────────────────────┐         │
│  │ NEW         - Thread created, not started                  │         │
│  │ RUNNABLE    - Ready to run (may be waiting for CPU)        │         │
│  │ RUNNING     - Currently executing (not a real state)       │         │
│  │ BLOCKED     - Waiting to enter synchronized block          │         │
│  │ WAITING     - Waiting indefinitely (wait(), join())        │         │
│  │ TIMED_WAITING - Waiting for specified time                 │         │
│  │ TERMINATED  - Thread completed or died with exception      │         │
│  └────────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Race Condition

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Race Condition Example                           │
│                                                                          │
│   Shared variable: counter = 0                                          │
│   Operation: counter++  (actually 3 operations: read, increment, write)  │
│                                                                          │
│   Thread 1                  Time                Thread 2                 │
│   ────────                  ────                ────────                 │
│                              │                                           │
│   Read counter (0)           │                                           │
│        │                     │                                           │
│        ▼                     │                                           │
│   Increment (0→1)            │                                           │
│        │                     │                  Read counter (0)         │
│        │                     │                       │                   │
│        │                     │                       ▼                   │
│   Write counter (1)          │                  Increment (0→1)          │
│                              │                       │                   │
│                              │                       ▼                   │
│                              │                  Write counter (1)        │
│                              │                                           │
│                              ▼                                           │
│                                                                          │
│   Expected: counter = 2                                                 │
│   Actual:   counter = 1     ← RACE CONDITION!                           │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   Solution: Synchronization                                             │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   Thread 1                  Lock                Thread 2                 │
│   ────────                 ──────               ────────                 │
│                              │                                           │
│   ACQUIRE LOCK ──────────────┤                                           │
│        │                     │                  TRY ACQUIRE              │
│   Read counter (0)           │                       │                   │
│   Increment (0→1)            │                       │ BLOCKED           │
│   Write counter (1)          │                       │                   │
│   RELEASE LOCK ──────────────┤                       │                   │
│                              │                       │                   │
│                              │               ACQUIRE LOCK                │
│                              │                  Read counter (1)         │
│                              │                  Increment (1→2)          │
│                              │                  Write counter (2)        │
│                              │               RELEASE LOCK                │
│                              │                                           │
│   Result: counter = 2  ✓                                                │
└─────────────────────────────────────────────────────────────────────────┘
```

### Deadlock

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          DEADLOCK                                        │
│                                                                          │
│   Thread 1                                    Thread 2                   │
│   ────────                                    ────────                   │
│                                                                          │
│   1. Acquire Lock A ────┐          ┌────── 1. Acquire Lock B            │
│                         │          │                                     │
│                         ▼          ▼                                     │
│                    ┌─────────┐ ┌─────────┐                               │
│                    │ Lock A  │ │ Lock B  │                               │
│                    │ (held)  │ │ (held)  │                               │
│                    └────┬────┘ └────┬────┘                               │
│                         │          │                                     │
│   2. Try Lock B ◄───────┼──────────┼───────► 2. Try Lock A              │
│      BLOCKED!           │          │          BLOCKED!                   │
│         │               │          │             │                       │
│         │               ▼          ▼             │                       │
│         │          ┌─────────────────────┐       │                       │
│         │          │     DEADLOCK!       │       │                       │
│         └─────────►│  Both threads wait  │◄──────┘                       │
│                    │     forever...      │                               │
│                    └─────────────────────┘                               │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   Four Conditions for Deadlock (ALL must be present):                   │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   1. Mutual Exclusion    - Resources can't be shared                    │
│   2. Hold and Wait       - Thread holds one, waits for another          │
│   3. No Preemption       - Resources can't be forcibly taken            │
│   4. Circular Wait       - A waits for B, B waits for A                 │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   Prevention: Lock Ordering                                             │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   Always acquire locks in the SAME ORDER:                               │
│                                                                          │
│   Thread 1                                    Thread 2                   │
│   1. Acquire Lock A                           1. Acquire Lock A          │
│   2. Acquire Lock B                           2. Acquire Lock B          │
│   3. Do work                                  3. Do work                 │
│   4. Release Lock B                           4. Release Lock B          │
│   5. Release Lock A                           5. Release Lock A          │
│                                                                          │
│   Now no circular wait possible!                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

### Java Memory Model (JMM)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Java Memory Model                                     │
│                                                                          │
│   Each thread has its own working memory (cache):                       │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                        CPU                                   │       │
│   │   ┌─────────────────────┐     ┌─────────────────────┐       │       │
│   │   │      Thread 1       │     │      Thread 2       │       │       │
│   │   │  ┌───────────────┐  │     │  ┌───────────────┐  │       │       │
│   │   │  │ Working       │  │     │  │ Working       │  │       │       │
│   │   │  │ Memory (Cache)│  │     │  │ Memory (Cache)│  │       │       │
│   │   │  │  x = 0        │  │     │  │  x = 0        │  │       │       │
│   │   │  │  y = 0        │  │     │  │  y = 0        │  │       │       │
│   │   │  └───────────────┘  │     │  └───────────────┘  │       │       │
│   │   └─────────┬───────────┘     └──────────┬──────────┘       │       │
│   │             │                            │                   │       │
│   │             │     read/write             │                   │       │
│   │             │     operations             │                   │       │
│   │             ▼                            ▼                   │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                              │                                           │
│                              ▼                                           │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                    MAIN MEMORY (HEAP)                        │       │
│   │                                                              │       │
│   │                        x = 0                                 │       │
│   │                        y = 0                                 │       │
│   │                                                              │       │
│   │   Shared variables live here                                │       │
│   │   But threads may cache copies!                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   Problem: Thread 1 sets x = 1, but Thread 2 may still see x = 0!       │
│   Solution: volatile, synchronized, or atomic operations                │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   volatile Keyword                                                      │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   Without volatile:                   With volatile:                    │
│   ┌─────────────────┐                 ┌─────────────────┐               │
│   │ Thread 1        │                 │ Thread 1        │               │
│   │ cache: x=1      │                 │ writes x=1      │               │
│   │   └──? maybe ──►│                 │   └──always────►│               │
│   │        flush    │                 │        flush    │               │
│   └─────────────────┘                 └─────────────────┘               │
│          │                                    │                          │
│          ?                                    ▼                          │
│          ▼                            ┌─────────────────┐               │
│   ┌─────────────────┐                 │ Main Memory     │               │
│   │ Main Memory     │                 │ x = 1           │               │
│   │ x = ? (stale?)  │                 └────────┬────────┘               │
│   └─────────────────┘                          │                         │
│          │                                     │ always                  │
│          ?                                     │ read from               │
│          ▼                                     │ main memory             │
│   ┌─────────────────┐                          ▼                         │
│   │ Thread 2        │                 ┌─────────────────┐               │
│   │ cache: x=0      │ ← stale!        │ Thread 2        │               │
│   │ (never updated) │                 │ reads x=1       │ ✓             │
│   └─────────────────┘                 └─────────────────┘               │
└─────────────────────────────────────────────────────────────────────────┘
```

### Happens-Before Relationship

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Happens-Before Relationships                          │
│                                                                          │
│   If A happens-before B, then A's effects are visible to B              │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   Key Happens-Before Rules:                                             │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   1. Program Order Rule                                                 │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │ Within a single thread, each statement happens-before     │         │
│   │ the next statement in program order                       │         │
│   │                                                           │         │
│   │ x = 1;     ──happens-before──►  y = x + 1;               │         │
│   └───────────────────────────────────────────────────────────┘         │
│                                                                          │
│   2. Monitor Lock Rule                                                  │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │ Unlock of monitor happens-before lock of same monitor    │         │
│   │                                                           │         │
│   │ Thread 1:                    Thread 2:                    │         │
│   │ synchronized(lock) {         synchronized(lock) {         │         │
│   │   x = 1;                       int r = x;                 │         │
│   │ } ──happens-before──────────► }                          │         │
│   │   (unlock)                     (lock)                     │         │
│   │                                // r is guaranteed to be 1 │         │
│   └───────────────────────────────────────────────────────────┘         │
│                                                                          │
│   3. Volatile Variable Rule                                             │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │ Write to volatile happens-before subsequent read of same │         │
│   │                                                           │         │
│   │ volatile boolean ready;                                   │         │
│   │                                                           │         │
│   │ Thread 1:                    Thread 2:                    │         │
│   │ x = 42;                      while (!ready) { }           │         │
│   │ ready = true; ──h-b───────► if (ready)                   │         │
│   │                               // x is guaranteed to be 42 │         │
│   └───────────────────────────────────────────────────────────┘         │
│                                                                          │
│   4. Thread Start Rule                                                  │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │ thread.start() happens-before any action in that thread  │         │
│   │                                                           │         │
│   │ x = 1;                                                    │         │
│   │ thread.start(); ──h-b──► // Thread sees x = 1            │         │
│   └───────────────────────────────────────────────────────────┘         │
│                                                                          │
│   5. Thread Join Rule                                                   │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │ All actions in thread happen-before join() returns       │         │
│   │                                                           │         │
│   │ Thread t: x = 1;                                          │         │
│   │                                                           │         │
│   │ t.join(); ──h-b──► // We see x = 1 here                  │         │
│   └───────────────────────────────────────────────────────────┘         │
│                                                                          │
│   6. Transitivity                                                       │
│   ┌───────────────────────────────────────────────────────────┐         │
│   │ If A h-b B and B h-b C, then A h-b C                      │         │
│   └───────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Thread Pool Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Thread Pool (ExecutorService)                         │
│                                                                          │
│   Without Thread Pool:           With Thread Pool:                      │
│   ┌───────────────────┐          ┌───────────────────────────────────┐  │
│   │ Task 1 → new Thread()        │                                   │  │
│   │ Task 2 → new Thread()        │   ┌─────────────────────────────┐ │  │
│   │ Task 3 → new Thread()        │   │      Task Queue             │ │  │
│   │ ...                          │   │ [Task1][Task2][Task3][...]  │ │  │
│   │ Task N → new Thread()        │   └────────────┬────────────────┘ │  │
│   └───────────────────┘          │                │                  │  │
│                                  │                ▼                  │  │
│   Problems:                      │   ┌─────────────────────────────┐ │  │
│   • Thread creation is slow      │   │      Worker Threads         │ │  │
│   • Uncontrolled thread count    │   │  ┌────┐ ┌────┐ ┌────┐      │ │  │
│   • Resource exhaustion          │   │  │ T1 │ │ T2 │ │ T3 │ ...  │ │  │
│                                  │   │  └────┘ └────┘ └────┘      │ │  │
│                                  │   │   (fixed number, reused)    │ │  │
│                                  │   └─────────────────────────────┘ │  │
│                                  └───────────────────────────────────┘  │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   Types of Thread Pools:                                                │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │ newFixedThreadPool(n)                                        │       │
│   │ ┌────┐ ┌────┐ ┌────┐ ┌────┐                                  │       │
│   │ │ T1 │ │ T2 │ │ T3 │ │ T4 │   Fixed n threads              │       │
│   │ └────┘ └────┘ └────┘ └────┘   Unbounded queue               │       │
│   │ Best for: CPU-bound tasks                                   │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │ newCachedThreadPool()                                        │       │
│   │ ┌────┐ ┌────┐ ┌────┐ ... ┌────┐                              │       │
│   │ │ T1 │ │ T2 │ │ T3 │     │ Tn │   0 to MAX threads          │       │
│   │ └────┘ └────┘ └────┘     └────┘   (grows/shrinks as needed) │       │
│   │ Best for: Many short-lived tasks                            │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │ newSingleThreadExecutor()                                    │       │
│   │ ┌────┐                                                       │       │
│   │ │ T1 │   Single thread, tasks execute sequentially          │       │
│   │ └────┘   Guarantees task ordering                           │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │ newScheduledThreadPool(n)                                    │       │
│   │ ┌────┐ ┌────┐ ┌────┐                                         │       │
│   │ │ T1 │ │ T2 │ │ T3 │   For delayed/periodic tasks           │       │
│   │ └────┘ └────┘ └────┘   schedule(), scheduleAtFixedRate()    │       │
│   └─────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
```

### CompletableFuture Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CompletableFuture Pipeline                            │
│                                                                          │
│   Async computation chain:                                              │
│                                                                          │
│   ┌──────────────┐   thenApply    ┌──────────────┐   thenApply         │
│   │ supplyAsync  │──────────────►│  Transform   │──────────────►...    │
│   │ (start)      │               │  result      │                      │
│   └──────────────┘               └──────────────┘                      │
│          │                                                              │
│          │  thenCompose (flatMap)                                       │
│          ▼                                                              │
│   ┌──────────────────────────────────────────────────────────┐         │
│   │                                                           │         │
│   │   CompletableFuture.supplyAsync(() -> fetchUser(id))     │         │
│   │       .thenApply(user -> user.getEmail())                │         │
│   │       .thenCompose(email -> sendEmailAsync(email))       │         │
│   │       .thenAccept(result -> log("Done: " + result))      │         │
│   │       .exceptionally(ex -> handleError(ex));             │         │
│   │                                                           │         │
│   └──────────────────────────────────────────────────────────┘         │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   Combining Futures:                                                    │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   thenCombine (parallel, combine results):                              │
│   ┌─────────────┐                                                       │
│   │  Future A   │─────┐                                                 │
│   └─────────────┘     │    ┌─────────────────┐                          │
│                       ├───►│ combine(A, B)   │───► Result              │
│   ┌─────────────┐     │    └─────────────────┘                          │
│   │  Future B   │─────┘    (both complete first)                       │
│   └─────────────┘                                                       │
│                                                                          │
│   allOf (wait for all):                                                 │
│   ┌─────────────┐                                                       │
│   │  Future A   │───┐                                                   │
│   ├─────────────┤   │                                                   │
│   │  Future B   │───┼────► CompletableFuture.allOf(A, B, C)            │
│   ├─────────────┤   │              │                                    │
│   │  Future C   │───┘              ▼                                    │
│   └─────────────┘           Completes when ALL complete                │
│                                                                          │
│   anyOf (first to complete):                                            │
│   ┌─────────────┐                                                       │
│   │  Future A   │───┐                                                   │
│   ├─────────────┤   │                                                   │
│   │  Future B   │───┼────► CompletableFuture.anyOf(A, B, C)            │
│   ├─────────────┤   │              │                                    │
│   │  Future C   │───┘              ▼                                    │
│   └─────────────┘           Completes when FIRST completes             │
│                                                                          │
│   ═══════════════════════════════════════════════════════════════       │
│   Async vs Non-Async Methods:                                           │
│   ═══════════════════════════════════════════════════════════════       │
│                                                                          │
│   thenApply(fn)      - runs fn in same thread                          │
│   thenApplyAsync(fn) - runs fn in ForkJoinPool.commonPool()            │
│   thenApplyAsync(fn, executor) - runs fn in specified executor         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Concurrent Collections

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Concurrent Collections                                │
│                                                                          │
│   ConcurrentHashMap (Java 8+):                                          │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │  Segment-based (pre-Java 8)  →  Node-based (Java 8+)          │     │
│   │                                                                │     │
│   │  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐            │     │
│   │  │ [0] │ [1] │ [2] │ [3] │ [4] │ [5] │ [6] │ [7] │  Buckets   │     │
│   │  └──┬──┴──┬──┴──┬──┴─────┴──┬──┴─────┴──┬──┴─────┘            │     │
│   │     │     │     │           │           │                      │     │
│   │     ▼     ▼     ▼           ▼           ▼                      │     │
│   │   ┌───┐ ┌───┐ ┌───┐      ┌───┐       ┌───┐                     │     │
│   │   │K:V│ │K:V│ │K:V│      │K:V│       │K:V│                     │     │
│   │   └───┘ └─┬─┘ └───┘      └─┬─┘       └─┬─┘                     │     │
│   │           │                │           │                        │     │
│   │         ┌─▼─┐            ┌─▼─┐       ┌─▼─┐                      │     │
│   │         │K:V│            │K:V│       │...│                      │     │
│   │         └───┘            └───┘       └───┘                      │     │
│   │                                                                │     │
│   │  • Lock only on specific bucket (fine-grained)                │     │
│   │  • Reads don't block (lock-free)                              │     │
│   │  • Writes lock only the affected bucket                       │     │
│   │  • No null keys or values                                     │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   CopyOnWriteArrayList:                                                 │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │  Read (no lock):           Write (copy entire array):         │     │
│   │                                                                │     │
│   │  ┌───┬───┬───┬───┬───┐    1. Lock                             │     │
│   │  │ A │ B │ C │ D │ E │    2. Copy array                       │     │
│   │  └───┴───┴───┴───┴───┘    3. Modify copy                      │     │
│   │       │                    4. Replace reference                │     │
│   │       │ read               5. Unlock                          │     │
│   │       ▼                                                        │     │
│   │    element                 ┌───┬───┬───┬───┬───┬───┐          │     │
│   │                            │ A │ B │ C │ D │ E │ X │  (new)   │     │
│   │                            └───┴───┴───┴───┴───┴───┘          │     │
│   │                                                                │     │
│   │  • Great for read-heavy, write-rarely scenarios               │     │
│   │  • Iterator sees snapshot (no ConcurrentModificationException)│     │
│   │  • Write is O(n) - copies entire array                        │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   BlockingQueue:                                                        │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │  Producer ──► [━━━━━━━━━━━━━━━━━━━━] ──► Consumer             │     │
│   │                      Queue                                     │     │
│   │                                                                │     │
│   │  put(): blocks if full                                        │     │
│   │  take(): blocks if empty                                      │     │
│   │                                                                │     │
│   │  Implementations:                                              │     │
│   │  • ArrayBlockingQueue  - bounded, array-backed                │     │
│   │  • LinkedBlockingQueue - optionally bounded, linked nodes     │     │
│   │  • PriorityBlockingQueue - unbounded, priority ordering       │     │
│   │  • SynchronousQueue    - no capacity (handoff)                │     │
│   └───────────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────┘
```

### Synchronizers

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Synchronizers                                    │
│                                                                          │
│   CountDownLatch (One-time barrier):                                    │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │   count = 3                                                   │     │
│   │                                                                │     │
│   │   Thread A: countDown() ──► count = 2                         │     │
│   │   Thread B: countDown() ──► count = 1                         │     │
│   │   Thread C: countDown() ──► count = 0 ──► Release waiting     │     │
│   │                                                                │     │
│   │   Main Thread: await() ────────────────► Proceeds!            │     │
│   │                  (blocked until count = 0)                     │     │
│   │                                                                │     │
│   │   Use case: Wait for N tasks to complete                      │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   CyclicBarrier (Reusable rendezvous):                                  │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │   parties = 3                                                  │     │
│   │                                                                │     │
│   │   Thread A: await() ──┐                                        │     │
│   │   Thread B: await() ──┼──► All 3 arrive → barrier trips       │     │
│   │   Thread C: await() ──┘              │                         │     │
│   │                                      ▼                         │     │
│   │                         Run optional barrierAction            │     │
│   │                                      │                         │     │
│   │                                      ▼                         │     │
│   │                         All threads continue                   │     │
│   │                         (barrier resets for next round)        │     │
│   │                                                                │     │
│   │   Use case: Parallel iterations (compute, sync, repeat)       │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   Semaphore (Permit-based):                                             │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │   permits = 3                                                  │     │
│   │                                                                │     │
│   │   Thread A: acquire() ──► permits = 2 (has 1 permit)          │     │
│   │   Thread B: acquire() ──► permits = 1 (has 1 permit)          │     │
│   │   Thread C: acquire() ──► permits = 0 (has 1 permit)          │     │
│   │   Thread D: acquire() ──► BLOCKED (no permits available)      │     │
│   │                                                                │     │
│   │   Thread A: release() ──► permits = 1                         │     │
│   │   Thread D: ──────────────────────────► Unblocked!            │     │
│   │                                                                │     │
│   │   Use case: Rate limiting, resource pooling                   │     │
│   └───────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   Phaser (Flexible barrier):                                            │
│   ┌───────────────────────────────────────────────────────────────┐     │
│   │                                                                │     │
│   │   Like CyclicBarrier but:                                     │     │
│   │   • Dynamic party count (register/deregister)                 │     │
│   │   • Multiple phases with phase numbers                        │     │
│   │   • Supports hierarchical phasers                             │     │
│   │                                                                │     │
│   │   Phase 0 ──► Phase 1 ──► Phase 2 ──► ...                    │     │
│   │                                                                │     │
│   │   Use case: Complex multi-phase computations                  │     │
│   └───────────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Code Examples

### 1) Creating and Running Threads

```java
// Method 1: Extend Thread class
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread: " + Thread.currentThread().getName());
    }
}

// Method 2: Implement Runnable (preferred)
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable: " + Thread.currentThread().getName());
    }
}

// Method 3: Lambda (most concise)
Runnable lambdaRunnable = () -> {
    System.out.println("Lambda: " + Thread.currentThread().getName());
};

// Starting threads
public static void main(String[] args) {
    // Extend Thread
    MyThread t1 = new MyThread();
    t1.setName("Thread-1");
    t1.start();  // Creates new thread, calls run()
    
    // Implement Runnable
    Thread t2 = new Thread(new MyRunnable(), "Thread-2");
    t2.start();
    
    // Lambda
    Thread t3 = new Thread(lambdaRunnable, "Thread-3");
    t3.start();
    
    // Inline lambda
    new Thread(() -> System.out.println("Inline lambda")).start();
    
    // Don't call run() directly - executes in current thread!
    t1.run();  // Wrong! Doesn't create new thread
    t1.start(); // Correct! Creates new thread
}

// Thread with return value: Callable
Callable<Integer> callable = () -> {
    Thread.sleep(1000);
    return 42;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(callable);
Integer result = future.get();  // Blocks until complete
executor.shutdown();
```

### 2) Thread Synchronization

```java
// Problem: Race condition
class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // Not atomic! Read-increment-write
    }
    
    public int getCount() {
        return count;
    }
}

// Solution 1: synchronized method
class SynchronizedCounter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;  // Only one thread can execute at a time
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// Solution 2: synchronized block
class BlockSynchronizedCounter {
    private int count = 0;
    private final Object lock = new Object();  // Dedicated lock object
    
    public void increment() {
        synchronized (lock) {
            count++;
        }
    }
    
    public int getCount() {
        synchronized (lock) {
            return count;
        }
    }
}

// Solution 3: ReentrantLock (more flexible)
class LockCounter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();
    
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();  // Always unlock in finally!
        }
    }
    
    // Try lock with timeout
    public boolean tryIncrement() throws InterruptedException {
        if (lock.tryLock(1, TimeUnit.SECONDS)) {
            try {
                count++;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false;
    }
}

// Solution 4: AtomicInteger (best for simple counters)
class AtomicCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // Atomic operation
    }
    
    public int getCount() {
        return count.get();
    }
}

// Read-Write Lock (multiple readers, single writer)
class Cache {
    private final Map<String, Object> cache = new HashMap<>();
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    
    public Object get(String key) {
        rwLock.readLock().lock();
        try {
            return cache.get(key);  // Multiple readers allowed
        } finally {
            rwLock.readLock().unlock();
        }
    }
    
    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            cache.put(key, value);  // Exclusive access
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}
```

### 3) wait(), notify(), and notifyAll()

```java
// Producer-Consumer with wait/notify
class BlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final Object lock = new Object();
    
    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }
    
    public void put(T item) throws InterruptedException {
        synchronized (lock) {
            while (queue.size() == capacity) {  // Always use while, not if!
                lock.wait();  // Release lock, wait for notify
            }
            queue.offer(item);
            lock.notifyAll();  // Wake up waiting consumers
        }
    }
    
    public T take() throws InterruptedException {
        synchronized (lock) {
            while (queue.isEmpty()) {  // Always use while, not if!
                lock.wait();  // Release lock, wait for notify
            }
            T item = queue.poll();
            lock.notifyAll();  // Wake up waiting producers
            return item;
        }
    }
}

// Usage
BlockingQueue<Integer> queue = new BlockingQueue<>(10);

// Producer thread
new Thread(() -> {
    for (int i = 0; i < 100; i++) {
        try {
            queue.put(i);
            System.out.println("Produced: " + i);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}).start();

// Consumer thread
new Thread(() -> {
    for (int i = 0; i < 100; i++) {
        try {
            int item = queue.take();
            System.out.println("Consumed: " + item);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}).start();

// Why while instead of if?
// Spurious wakeups: thread can wake without notify()
// Multiple threads: another thread might have taken the item
// Always re-check condition after waking!
```

### 4) volatile Keyword

```java
// Problem: Visibility issue
class FlagExample {
    private boolean running = true;  // May be cached by thread
    
    public void run() {
        while (running) {  // Thread might never see the change!
            // do work
        }
    }
    
    public void stop() {
        running = false;  // Change may not be visible to other thread
    }
}

// Solution: volatile ensures visibility
class VolatileFlagExample {
    private volatile boolean running = true;  // Always read from main memory
    
    public void run() {
        while (running) {  // Will see changes made by other threads
            // do work
        }
    }
    
    public void stop() {
        running = false;  // Change is immediately visible to all threads
    }
}

// volatile is NOT enough for compound operations!
class VolatileCounter {
    private volatile int count = 0;
    
    public void increment() {
        count++;  // STILL NOT ATOMIC! Read-increment-write
        // Even though reads/writes are volatile, the compound
        // operation is not atomic
    }
}

// Use AtomicInteger instead
class SafeCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // Atomic
    }
}

// volatile is good for:
// - Simple flags (start/stop)
// - Publishing immutable objects
// - Double-checked locking (with synchronization)

// Double-checked locking (DCL) pattern
class Singleton {
    private static volatile Singleton instance;  // volatile required!
    
    public static Singleton getInstance() {
        if (instance == null) {  // First check (no locking)
            synchronized (Singleton.class) {
                if (instance == null) {  // Second check (with locking)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### 5) ExecutorService and Thread Pools

```java
import java.util.concurrent.*;

// Fixed thread pool
ExecutorService fixedPool = Executors.newFixedThreadPool(4);
for (int i = 0; i < 10; i++) {
    final int taskId = i;
    fixedPool.execute(() -> {
        System.out.println("Task " + taskId + " on " + Thread.currentThread().getName());
    });
}

// Submit with return value
Future<String> future = fixedPool.submit(() -> {
    Thread.sleep(1000);
    return "Result";
});

// Block and get result
String result = future.get();  // Blocks until complete
String resultWithTimeout = future.get(5, TimeUnit.SECONDS);  // Timeout

// Check status
future.isDone();      // true if complete
future.isCancelled(); // true if cancelled
future.cancel(true);  // Try to cancel (mayInterruptIfRunning)

// Cached thread pool (for many short-lived tasks)
ExecutorService cachedPool = Executors.newCachedThreadPool();

// Single thread executor (sequential execution)
ExecutorService singlePool = Executors.newSingleThreadExecutor();

// Scheduled executor
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

// Schedule with delay
scheduler.schedule(() -> System.out.println("Delayed"), 5, TimeUnit.SECONDS);

// Schedule at fixed rate (every N seconds)
scheduler.scheduleAtFixedRate(
    () -> System.out.println("Periodic"),
    0,     // initialDelay
    1,     // period
    TimeUnit.SECONDS
);

// Schedule with fixed delay (N seconds after previous completion)
scheduler.scheduleWithFixedDelay(
    () -> System.out.println("Fixed delay"),
    0,     // initialDelay
    1,     // delay after completion
    TimeUnit.SECONDS
);

// Proper shutdown
fixedPool.shutdown();  // Stop accepting new tasks
try {
    if (!fixedPool.awaitTermination(60, TimeUnit.SECONDS)) {
        fixedPool.shutdownNow();  // Force shutdown
    }
} catch (InterruptedException e) {
    fixedPool.shutdownNow();
    Thread.currentThread().interrupt();
}

// Custom thread pool (recommended for production)
ThreadPoolExecutor customPool = new ThreadPoolExecutor(
    4,                      // corePoolSize
    8,                      // maximumPoolSize
    60, TimeUnit.SECONDS,   // keepAliveTime
    new ArrayBlockingQueue<>(100),  // workQueue
    new ThreadPoolExecutor.CallerRunsPolicy()  // rejectionHandler
);

// invokeAll - execute all and wait for completion
List<Callable<String>> tasks = List.of(
    () -> "Task 1",
    () -> "Task 2",
    () -> "Task 3"
);
List<Future<String>> futures = fixedPool.invokeAll(tasks);

// invokeAny - return first successful result
String firstResult = fixedPool.invokeAny(tasks);
```

### 6) CompletableFuture

```java
import java.util.concurrent.CompletableFuture;

// Create async computation
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // Simulating async operation
    sleep(1000);
    return "Hello";
});

// Transform result
CompletableFuture<Integer> lengthFuture = future.thenApply(s -> s.length());

// Chain multiple operations
CompletableFuture<String> result = CompletableFuture
    .supplyAsync(() -> fetchUser(userId))
    .thenApply(user -> user.getEmail())
    .thenCompose(email -> sendEmailAsync(email))  // Returns CompletableFuture
    .thenApply(response -> "Sent to " + response);

// Handle exceptions
CompletableFuture<String> safe = future
    .exceptionally(ex -> "Default value on error: " + ex.getMessage());

// Handle both success and error
CompletableFuture<String> handled = future
    .handle((result1, ex) -> {
        if (ex != null) {
            return "Error: " + ex.getMessage();
        }
        return "Success: " + result1;
    });

// Run when complete (doesn't transform result)
future.whenComplete((result2, ex) -> {
    if (ex != null) {
        System.out.println("Error: " + ex.getMessage());
    } else {
        System.out.println("Result: " + result2);
    }
});

// Consume result (no return value)
future.thenAccept(s -> System.out.println("Got: " + s));

// Run after completion (ignore result)
future.thenRun(() -> System.out.println("Completed!"));

// Combine two futures
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<String> combined = future1.thenCombine(
    future2,
    (s1, s2) -> s1 + " " + s2
);

// Wait for all futures
CompletableFuture<Void> allOf = CompletableFuture.allOf(future1, future2);
allOf.thenRun(() -> {
    // All futures complete
    String r1 = future1.join();
    String r2 = future2.join();
});

// First to complete wins
CompletableFuture<Object> anyOf = CompletableFuture.anyOf(future1, future2);

// Timeout (Java 9+)
CompletableFuture<String> withTimeout = future
    .orTimeout(1, TimeUnit.SECONDS)
    .exceptionally(ex -> "Timeout!");

// Complete exceptionally
CompletableFuture<String> manual = new CompletableFuture<>();
manual.completeExceptionally(new RuntimeException("Failed!"));

// Custom executor
ExecutorService executor = Executors.newFixedThreadPool(4);
CompletableFuture<String> customExecutor = CompletableFuture.supplyAsync(
    () -> "Custom",
    executor  // Use custom executor
);

// Parallel operations and collect results
List<Integer> ids = List.of(1, 2, 3, 4, 5);
List<CompletableFuture<User>> futures = ids.stream()
    .map(id -> CompletableFuture.supplyAsync(() -> fetchUser(id)))
    .collect(Collectors.toList());

CompletableFuture<List<User>> allUsers = CompletableFuture
    .allOf(futures.toArray(new CompletableFuture[0]))
    .thenApply(v -> futures.stream()
        .map(CompletableFuture::join)
        .collect(Collectors.toList()));
```

### 7) Atomic Classes

```java
import java.util.concurrent.atomic.*;

// AtomicInteger - thread-safe int
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();    // ++i, returns new value
counter.getAndIncrement();    // i++, returns old value
counter.addAndGet(5);         // Add and return new value
counter.compareAndSet(5, 10); // Set to 10 if current is 5
counter.updateAndGet(x -> x * 2);  // Custom update function

// AtomicLong - thread-safe long
AtomicLong bigCounter = new AtomicLong(0L);

// AtomicBoolean - thread-safe boolean
AtomicBoolean flag = new AtomicBoolean(false);
flag.compareAndSet(false, true);  // Set true only if currently false

// AtomicReference - thread-safe object reference
AtomicReference<String> ref = new AtomicReference<>("initial");
ref.compareAndSet("initial", "updated");
ref.getAndUpdate(s -> s.toUpperCase());
ref.updateAndGet(s -> s + "!");

// AtomicIntegerArray - array of atomic integers
AtomicIntegerArray array = new AtomicIntegerArray(10);
array.incrementAndGet(0);  // Increment element at index 0
array.compareAndSet(0, 1, 10);  // At index 0, set to 10 if current is 1

// LongAdder - better for high contention counters
LongAdder adder = new LongAdder();
adder.increment();
adder.add(10);
long sum = adder.sum();  // Get current sum

// DoubleAdder - for double values
DoubleAdder doubleAdder = new DoubleAdder();
doubleAdder.add(3.14);

// AtomicStampedReference - prevents ABA problem
AtomicStampedReference<String> stamped = new AtomicStampedReference<>("A", 0);
int[] stampHolder = new int[1];
String current = stamped.get(stampHolder);  // Get value and stamp
stamped.compareAndSet("A", "B", 0, 1);  // Include stamp in comparison

// Lock-free stack using AtomicReference
class LockFreeStack<T> {
    private final AtomicReference<Node<T>> head = new AtomicReference<>();
    
    public void push(T item) {
        Node<T> newHead = new Node<>(item);
        Node<T> oldHead;
        do {
            oldHead = head.get();
            newHead.next = oldHead;
        } while (!head.compareAndSet(oldHead, newHead));  // CAS loop
    }
    
    public T pop() {
        Node<T> oldHead;
        Node<T> newHead;
        do {
            oldHead = head.get();
            if (oldHead == null) return null;
            newHead = oldHead.next;
        } while (!head.compareAndSet(oldHead, newHead));
        return oldHead.item;
    }
    
    private static class Node<T> {
        final T item;
        Node<T> next;
        Node(T item) { this.item = item; }
    }
}
```

### 8) Concurrent Collections

```java
import java.util.concurrent.*;

// ConcurrentHashMap - thread-safe HashMap
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);
map.get("a");
map.putIfAbsent("b", 2);  // Atomic
map.computeIfAbsent("c", k -> expensiveCompute(k));  // Atomic
map.merge("a", 1, Integer::sum);  // Atomic increment

// Bulk operations (Java 8+)
map.forEach(2, (k, v) -> System.out.println(k + "=" + v));  // Parallel
long count = map.mappingCount();  // Use instead of size() for concurrent
map.search(2, (k, v) -> v > 100 ? k : null);  // Parallel search
int sum = map.reduce(2, (k, v) -> v, Integer::sum);  // Parallel reduce

// CopyOnWriteArrayList - read-heavy, write-light
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("item");  // Creates new array copy!
// Safe iteration without ConcurrentModificationException
for (String item : cowList) {
    cowList.add("new");  // OK! Iterator sees snapshot
}

// CopyOnWriteArraySet - Set backed by CopyOnWriteArrayList
CopyOnWriteArraySet<String> cowSet = new CopyOnWriteArraySet<>();

// BlockingQueue implementations
// ArrayBlockingQueue - bounded
BlockingQueue<String> arrayQueue = new ArrayBlockingQueue<>(100);
arrayQueue.put("item");   // Blocks if full
arrayQueue.take();        // Blocks if empty
arrayQueue.offer("item", 1, TimeUnit.SECONDS);  // Timeout
arrayQueue.poll(1, TimeUnit.SECONDS);           // Timeout

// LinkedBlockingQueue - optionally bounded
BlockingQueue<String> linkedQueue = new LinkedBlockingQueue<>(100);

// PriorityBlockingQueue - unbounded priority queue
BlockingQueue<Integer> priorityQueue = new PriorityBlockingQueue<>();

// SynchronousQueue - no capacity, direct handoff
BlockingQueue<String> syncQueue = new SynchronousQueue<>();

// ConcurrentLinkedQueue - non-blocking, unbounded
ConcurrentLinkedQueue<String> clq = new ConcurrentLinkedQueue<>();
clq.offer("item");  // Never blocks
clq.poll();         // Returns null if empty

// ConcurrentSkipListMap - sorted, concurrent
ConcurrentSkipListMap<String, Integer> skipMap = new ConcurrentSkipListMap<>();
skipMap.put("b", 2);
skipMap.put("a", 1);
// Iteration in sorted order

// ConcurrentSkipListSet - sorted, concurrent set
ConcurrentSkipListSet<String> skipSet = new ConcurrentSkipListSet<>();
```

### 9) Synchronizers

```java
import java.util.concurrent.*;

// CountDownLatch - wait for N events
CountDownLatch latch = new CountDownLatch(3);

// Worker threads
for (int i = 0; i < 3; i++) {
    final int id = i;
    new Thread(() -> {
        System.out.println("Worker " + id + " done");
        latch.countDown();  // Decrement count
    }).start();
}

latch.await();  // Wait until count reaches 0
System.out.println("All workers done!");

// CyclicBarrier - reusable synchronization point
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("Barrier action: all threads arrived!");
});

for (int i = 0; i < 3; i++) {
    final int id = i;
    new Thread(() -> {
        try {
            System.out.println("Thread " + id + " waiting at barrier");
            barrier.await();  // Wait for all threads
            System.out.println("Thread " + id + " passed barrier");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }).start();
}

// Semaphore - limit concurrent access
Semaphore semaphore = new Semaphore(3);  // 3 permits

for (int i = 0; i < 10; i++) {
    final int id = i;
    new Thread(() -> {
        try {
            semaphore.acquire();  // Get permit (blocks if none available)
            System.out.println("Thread " + id + " acquired permit");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            semaphore.release();  // Return permit
        }
    }).start();
}

// Phaser - flexible barrier
Phaser phaser = new Phaser(3);  // 3 parties

for (int i = 0; i < 3; i++) {
    final int id = i;
    new Thread(() -> {
        System.out.println("Thread " + id + " phase 0");
        phaser.arriveAndAwaitAdvance();  // Wait at barrier
        
        System.out.println("Thread " + id + " phase 1");
        phaser.arriveAndAwaitAdvance();
        
        System.out.println("Thread " + id + " done");
        phaser.arriveAndDeregister();  // Leave the phaser
    }).start();
}

// Exchanger - exchange data between two threads
Exchanger<String> exchanger = new Exchanger<>();

new Thread(() -> {
    try {
        String data = "From Thread A";
        String received = exchanger.exchange(data);  // Send and receive
        System.out.println("Thread A received: " + received);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}).start();

new Thread(() -> {
    try {
        String data = "From Thread B";
        String received = exchanger.exchange(data);
        System.out.println("Thread B received: " + received);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}).start();
```

### 10) Deadlock Prevention

```java
// Problem: Deadlock due to different lock ordering
class DeadlockExample {
    private final Object lockA = new Object();
    private final Object lockB = new Object();
    
    public void method1() {
        synchronized (lockA) {  // Thread 1 gets lockA
            sleep(100);
            synchronized (lockB) {  // Thread 1 waits for lockB
                System.out.println("method1");
            }
        }
    }
    
    public void method2() {
        synchronized (lockB) {  // Thread 2 gets lockB
            sleep(100);
            synchronized (lockA) {  // Thread 2 waits for lockA
                System.out.println("method2");
            }
        }
    }
}

// Solution 1: Consistent lock ordering
class FixedLockOrder {
    private final Object lockA = new Object();
    private final Object lockB = new Object();
    
    public void method1() {
        synchronized (lockA) {  // Always get A first
            synchronized (lockB) {
                System.out.println("method1");
            }
        }
    }
    
    public void method2() {
        synchronized (lockA) {  // Always get A first (same order!)
            synchronized (lockB) {
                System.out.println("method2");
            }
        }
    }
}

// Solution 2: Use tryLock with timeout
class TryLockExample {
    private final Lock lockA = new ReentrantLock();
    private final Lock lockB = new ReentrantLock();
    
    public boolean transferMoney(Account from, Account to, double amount) 
            throws InterruptedException {
        while (true) {
            if (lockA.tryLock(100, TimeUnit.MILLISECONDS)) {
                try {
                    if (lockB.tryLock(100, TimeUnit.MILLISECONDS)) {
                        try {
                            // Transfer logic
                            return true;
                        } finally {
                            lockB.unlock();
                        }
                    }
                } finally {
                    lockA.unlock();
                }
            }
            // Back off and retry
            Thread.sleep(new Random().nextInt(100));
        }
    }
}

// Solution 3: Lock ordering based on hash
class LockByHash {
    public void transfer(Account from, Account to, double amount) {
        int fromHash = System.identityHashCode(from);
        int toHash = System.identityHashCode(to);
        
        Object firstLock = fromHash < toHash ? from : to;
        Object secondLock = fromHash < toHash ? to : from;
        
        synchronized (firstLock) {
            synchronized (secondLock) {
                // Transfer logic
            }
        }
    }
}
```

---

## Common Pitfalls

### 1. Not Handling InterruptedException Properly

```java
// ❌ Bad: Swallowing the interrupt
public void run() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        // Don't just ignore!
    }
}

// ✅ Good: Restore interrupt status or propagate
public void run() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();  // Restore status
        // Or rethrow if method allows
    }
}
```

### 2. Using notify() Instead of notifyAll()

```java
// ❌ Bad: notify() may wake wrong thread
synchronized (lock) {
    while (!condition) {
        lock.wait();
    }
    lock.notify();  // Only wakes one thread - may not be the right one!
}

// ✅ Good: notifyAll() wakes all threads
synchronized (lock) {
    while (!condition) {
        lock.wait();
    }
    lock.notifyAll();  // All threads wake and recheck condition
}
```

### 3. Forgetting to Unlock

```java
// ❌ Bad: Lock may never be released if exception occurs
lock.lock();
doSomething();  // If this throws, lock is never released!
lock.unlock();

// ✅ Good: Always unlock in finally
lock.lock();
try {
    doSomething();
} finally {
    lock.unlock();
}
```

### 4. Using synchronized on Wrong Object

```java
// ❌ Bad: Each thread synchronizes on different string
public void method(String key) {
    synchronized (key) {  // Different strings = different locks!
        // Not properly synchronized
    }
}

// ✅ Good: Use consistent lock object
private final Object lock = new Object();
public void method(String key) {
    synchronized (lock) {
        // Properly synchronized
    }
}
```

---

## Interview Questions

### Basic Questions

1. **What is the difference between Thread and Runnable?**
   - Thread is a class, Runnable is an interface
   - Implementing Runnable is preferred (allows extending another class)
   - Runnable is more flexible with executors

2. **What is the difference between start() and run()?**
   - `start()`: Creates new thread, calls run() in that thread
   - `run()`: Just a method call in current thread (no new thread)

3. **What are the thread states in Java?**
   - NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED

4. **What is a race condition?**
   - Bug where behavior depends on timing of thread execution
   - Multiple threads access shared data, at least one modifies
   - Solution: Synchronization

5. **What is deadlock? How do you prevent it?**
   - Two threads blocked forever, waiting for each other's locks
   - Prevention: Lock ordering, tryLock with timeout, avoid nested locks

### Intermediate Questions

6. **What is the difference between synchronized and Lock?**
   - synchronized: Implicit, automatic release, no timeout
   - Lock: Explicit, must release manually, supports tryLock, interruptible

7. **What is volatile?**
   - Ensures visibility of changes across threads
   - Reads/writes go to main memory, not cache
   - Not enough for compound operations (use atomic classes)

8. **What is the difference between wait() and sleep()?**
   - `wait()`: Releases lock, must be in synchronized, wait for notify
   - `sleep()`: Doesn't release lock, can call anywhere, wait for time

9. **What is a thread pool? Why use it?**
   - Pool of reusable threads
   - Avoids thread creation overhead
   - Controls max concurrent threads
   - Use ExecutorService

10. **What is CAS (Compare-And-Swap)?**
    - Atomic operation: compare value, if matches, swap with new value
    - Used by atomic classes for lock-free operations
    - Hardware-supported, very fast

### Advanced Questions

11. **Explain the Java Memory Model.**
    - Defines how threads interact with memory
    - Each thread may cache variables
    - Happens-before establishes visibility guarantees
    - volatile/synchronized provide memory barriers

12. **What is happens-before?**
    - Guarantee that memory writes are visible to another statement
    - Program order, unlock-lock, volatile write-read, thread start-run, join-after

13. **When would you use CompletableFuture vs ExecutorService?**
    - ExecutorService: Simple task execution
    - CompletableFuture: Complex async pipelines, combining results, exception handling

14. **What is the difference between ConcurrentHashMap and Collections.synchronizedMap()?**
    - ConcurrentHashMap: Fine-grained locking, better performance, no nulls
    - synchronizedMap: Whole-map locking, allows null, simpler

15. **How does ConcurrentHashMap achieve thread safety?**
    - Java 7: Segment-based locking
    - Java 8+: Node-level CAS + synchronized on bucket
    - Reads are lock-free
    - Treeification for long chains

---

## Quick Reference

### Thread State Transitions

| From | To | Trigger |
|------|----|---------| 
| NEW | RUNNABLE | start() |
| RUNNABLE | BLOCKED | Enter synchronized |
| RUNNABLE | WAITING | wait(), join(), park() |
| RUNNABLE | TIMED_WAITING | sleep(), wait(timeout) |
| RUNNABLE | TERMINATED | run() completes |
| BLOCKED | RUNNABLE | Lock acquired |
| WAITING | RUNNABLE | notify(), join returns |
| TIMED_WAITING | RUNNABLE | Timeout or notify |

### Synchronization Comparison

| Feature | synchronized | ReentrantLock | volatile | Atomic |
|---------|--------------|---------------|----------|--------|
| Mutual Exclusion | ✓ | ✓ | ✗ | ✗ |
| Visibility | ✓ | ✓ | ✓ | ✓ |
| Atomicity | Block | Block | Single read/write | ✓ |
| Try Lock | ✗ | ✓ | N/A | N/A |
| Interruptible | ✗ | ✓ | N/A | N/A |
| Fair Lock | ✗ | Optional | N/A | N/A |

### Concurrent Collections Comparison

| Collection | Thread-Safe | Null | Ordering | Notes |
|------------|-------------|------|----------|-------|
| HashMap | ✗ | ✓ | None | Not thread-safe |
| Hashtable | ✓ | ✗ | None | Legacy, slow |
| ConcurrentHashMap | ✓ | ✗ | None | Best concurrent map |
| CopyOnWriteArrayList | ✓ | ✓ | Insertion | Read-heavy |
| ConcurrentSkipListMap | ✓ | ✗ | Sorted | Concurrent TreeMap |
