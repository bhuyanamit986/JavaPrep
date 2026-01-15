# Concurrency & Multithreading (Deep Dive)

Concurrency is one of the **most heavily tested topics** in Java interviews. This guide covers everything from thread fundamentals to advanced concurrent utilities.

## Definitions

- **Thread**: the smallest unit of execution within a process.
- **Concurrency vs parallelism**: concurrency is managing multiple tasks; parallelism is running tasks at the same time.
- **Race condition**: incorrect behavior due to unsynchronized access to shared state.
- **Synchronization**: coordination that makes access to shared state safe (e.g., `synchronized`, locks).
- **Deadlock**: two or more threads waiting forever on each other’s locks.
- **Volatile**: a visibility guarantee for a variable across threads (not atomicity for compound actions).
- **Happens-before**: a formal rule defining visibility and ordering in the Java Memory Model.
- **Executor**: a higher-level API for managing thread pools and task execution.

## Illustrations

- **Race condition**: two clerks update the same bank balance at the same time and lose one update.
- **Deadlock**: two people each hold one key and refuse to release until they get the other.
- **Volatile**: a shared "status board" that always shows the latest posted value.

## Code Examples

```java
class Counter {
    private int value = 0;
    synchronized void inc() { value++; }
    synchronized int get() { return value; }
}
```

## Interview Questions

1. What is the difference between concurrency and parallelism?
2. What does `volatile` guarantee, and what does it not guarantee?
3. How can a deadlock happen, and how do you prevent it?
4. When would you use `synchronized` vs `Lock`?
5. What is the difference between `Runnable` and `Callable`?
6. Why use `ExecutorService` instead of creating threads manually?

---

## 1) Threads Fundamentals

### What is a Thread?

A thread is the smallest unit of execution within a process. Java applications run in the JVM process, which can have multiple threads executing concurrently.

### Creating Threads

**Method 1: Extending Thread class**

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

// Usage
MyThread t = new MyThread();
t.start(); // NOT t.run() - that runs in current thread!
```

**Method 2: Implementing Runnable (Preferred)**

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

// Usage
Thread t = new Thread(new MyRunnable());
t.start();

// Or with lambda
Thread t2 = new Thread(() -> System.out.println("Lambda thread"));
t2.start();
```

**Method 3: Implementing Callable (returns result)**

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};

ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task);
Integer result = future.get(); // Blocks until complete
executor.shutdown();
```

### Runnable vs Callable

| Feature | Runnable | Callable |
|---------|----------|----------|
| Return value | `void` | Generic `V` |
| Exception | Cannot throw checked | Can throw checked |
| Method | `run()` | `call()` |
| Usage | `Thread`, `Executor` | `Executor` only |

**Interview tip**: Prefer `Runnable` for fire-and-forget tasks; use `Callable` when you need results or checked exception handling.

---

## 2) Thread Lifecycle

A thread can be in one of these states (from `Thread.State` enum):

```
NEW → RUNNABLE → (BLOCKED | WAITING | TIMED_WAITING) → TERMINATED
```

### State Details

| State | Description |
|-------|-------------|
| **NEW** | Thread created but `start()` not called |
| **RUNNABLE** | Executing or ready to execute |
| **BLOCKED** | Waiting to acquire a monitor lock |
| **WAITING** | Waiting indefinitely (`wait()`, `join()`, `park()`) |
| **TIMED_WAITING** | Waiting with timeout (`sleep()`, `wait(timeout)`) |
| **TERMINATED** | Completed execution |

### Important Thread Methods

```java
// Starting
thread.start();          // Starts execution in new thread
// NEVER call run() directly - runs in current thread!

// Sleeping
Thread.sleep(1000);      // Pause current thread (milliseconds)
                         // Does NOT release locks!

// Joining
thread.join();           // Wait for thread to complete
thread.join(5000);       // Wait with timeout

// Interrupting
thread.interrupt();      // Sets interrupt flag
Thread.interrupted();    // Checks AND clears flag (static)
thread.isInterrupted();  // Checks flag (instance)

// Yielding
Thread.yield();          // Hint to scheduler (rarely used)

// Priority (1-10, default 5)
thread.setPriority(Thread.MAX_PRIORITY);
```

### Daemon Threads

```java
Thread daemon = new Thread(task);
daemon.setDaemon(true);  // Must set before start()
daemon.start();
```

- Daemon threads don't prevent JVM shutdown
- Used for background tasks (GC, finalizer thread)
- Child threads inherit daemon status from parent

**Interview pitfall**: Don't rely on daemon threads for critical cleanup - they may be killed abruptly.

---

## 3) Synchronization and Monitors

### The Problem: Race Conditions

```java
// UNSAFE - Race condition!
class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // NOT atomic: read → increment → write
    }
    
    public int getCount() {
        return count;
    }
}
```

### Solution 1: synchronized keyword

**Synchronized Method**

```java
class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;  // Only one thread at a time
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

**Synchronized Block (more granular)**

```java
class Counter {
    private int count = 0;
    private final Object lock = new Object();
    
    public void increment() {
        synchronized (lock) {  // Lock on specific object
            count++;
        }
    }
}
```

### Monitor Concept

Every Java object has an associated **monitor** (intrinsic lock):
- Only one thread can hold a monitor at a time
- `synchronized` acquires the monitor on entry, releases on exit
- Instance methods lock on `this`
- Static methods lock on `Class` object

**Interview question**: "What happens if a thread calls a synchronized method while holding the lock?"
**Answer**: Java's intrinsic locks are **reentrant** - same thread can acquire it multiple times.

```java
class Reentrant {
    public synchronized void outer() {
        inner();  // Same thread, same lock - OK
    }
    
    public synchronized void inner() {
        System.out.println("Inner called");
    }
}
```

---

## 4) wait(), notify(), notifyAll()

These methods enable **thread coordination** and must be called from within a synchronized block.

### Producer-Consumer Pattern

```java
class SharedQueue {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int capacity;
    
    public SharedQueue(int capacity) {
        this.capacity = capacity;
    }
    
    public synchronized void produce(int item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait();  // Release lock and wait
        }
        queue.add(item);
        notifyAll();  // Wake up consumers
    }
    
    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();  // Release lock and wait
        }
        int item = queue.poll();
        notifyAll();  // Wake up producers
        return item;
    }
}
```

### Critical Rules

1. **Always call wait() in a loop** - handles spurious wakeups
2. **Use notifyAll() over notify()** - notify() wakes only one thread (may not be the right one)
3. **Must hold the monitor** - otherwise `IllegalMonitorStateException`

### wait() vs sleep()

| Feature | wait() | sleep() |
|---------|--------|---------|
| Lock release | Yes | No |
| Called on | Object | Thread (static) |
| Wake condition | notify/notifyAll | Timeout |
| Location | synchronized block | Anywhere |

---

## 5) Deadlocks, Livelocks, and Starvation

### Deadlock

Two or more threads blocked forever, each waiting for the other's resource.

```java
// DEADLOCK EXAMPLE
Object lockA = new Object();
Object lockB = new Object();

// Thread 1
synchronized (lockA) {
    Thread.sleep(100);
    synchronized (lockB) { /* ... */ }  // Waits for lockB
}

// Thread 2
synchronized (lockB) {
    Thread.sleep(100);
    synchronized (lockA) { /* ... */ }  // Waits for lockA
}
```

**Deadlock conditions (all must be true):**
1. **Mutual exclusion**: Resources cannot be shared
2. **Hold and wait**: Thread holds one resource while waiting for another
3. **No preemption**: Resources cannot be forcibly taken
4. **Circular wait**: Circular chain of threads waiting

**Prevention strategies:**
- **Lock ordering**: Always acquire locks in the same order
- **Lock timeout**: Use `tryLock()` with timeout
- **Deadlock detection**: Use thread dumps, JMX monitoring
- **Avoid nested locks**: Minimize lock scope

### Livelock

Threads keep responding to each other but make no progress.

```java
// Example: Two threads trying to be "polite"
while (otherThreadWantsResource) {
    releaseResource();
    Thread.yield();
    acquireResource();
}
```

### Starvation

A thread cannot access resources because other threads hog them.

**Causes:**
- Threads with low priority never scheduled
- Threads stuck waiting for locks held by greedy threads
- Using `notify()` instead of `notifyAll()` (same thread keeps waking)

---

## 6) Java Memory Model (JMM) and volatile

### The Problem: Visibility

Without synchronization, changes made by one thread may not be visible to others:

```java
// BROKEN - stop may never be seen!
class BrokenRunner {
    private boolean stop = false;
    
    public void stop() { stop = true; }
    
    public void run() {
        while (!stop) {  // May loop forever
            doWork();
        }
    }
}
```

### volatile Keyword

Ensures visibility and prevents reordering for a single variable:

```java
class CorrectRunner {
    private volatile boolean stop = false;
    
    public void stop() { stop = true; }  // Write visible to all threads
    
    public void run() {
        while (!stop) {  // Read sees latest value
            doWork();
        }
    }
}
```

### volatile Guarantees

1. **Visibility**: Writes are immediately visible to all threads
2. **Ordering**: Prevents reordering of volatile accesses
3. **Atomicity for single read/write**: `long`/`double` become atomic

### volatile Does NOT Provide

- Atomicity for compound operations (`count++`)
- Mutual exclusion

**Interview tip**: `volatile` is for flags and single-value publication; use `synchronized` or atomics for compound operations.

### Happens-Before Relationship

The JMM defines **happens-before** ordering:

| Action A | Happens-Before Action B |
|----------|-------------------------|
| Thread start | First action in started thread |
| Synchronized unlock | Next lock of same monitor |
| Volatile write | Subsequent volatile read of same variable |
| Thread termination | `join()` returning |
| Interrupt | Interrupted thread detecting it |

**Key insight**: If A happens-before B, then A's effects are visible to B.

---

## 7) Executors and Thread Pools

### Why Thread Pools?

Creating threads is expensive. Thread pools:
- Reuse threads
- Control maximum concurrency
- Manage task queuing
- Provide graceful shutdown

### ExecutorService Basics

```java
// Creating thread pools
ExecutorService fixed = Executors.newFixedThreadPool(4);
ExecutorService cached = Executors.newCachedThreadPool();
ExecutorService single = Executors.newSingleThreadExecutor();
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);

// Submitting tasks
executor.execute(runnable);        // Fire and forget
Future<?> f1 = executor.submit(runnable);  // Can check completion
Future<T> f2 = executor.submit(callable);  // Returns result

// Shutdown
executor.shutdown();               // Graceful - finish submitted tasks
executor.shutdownNow();            // Forceful - interrupt running tasks
executor.awaitTermination(60, TimeUnit.SECONDS);
```

### ThreadPoolExecutor (Detailed Configuration)

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    4,                              // corePoolSize
    8,                              // maximumPoolSize
    60L, TimeUnit.SECONDS,          // keepAliveTime for idle threads
    new ArrayBlockingQueue<>(100),  // work queue
    new ThreadPoolExecutor.CallerRunsPolicy()  // rejection policy
);
```

### Rejection Policies

When queue is full and max threads reached:

| Policy | Behavior |
|--------|----------|
| `AbortPolicy` (default) | Throws `RejectedExecutionException` |
| `CallerRunsPolicy` | Runs task in caller's thread |
| `DiscardPolicy` | Silently discards task |
| `DiscardOldestPolicy` | Discards oldest queued task |

### Scheduled Tasks

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

// Run once after delay
scheduler.schedule(task, 5, TimeUnit.SECONDS);

// Run repeatedly at fixed rate (period between starts)
scheduler.scheduleAtFixedRate(task, 0, 1, TimeUnit.SECONDS);

// Run repeatedly with fixed delay (delay between end and next start)
scheduler.scheduleWithFixedDelay(task, 0, 1, TimeUnit.SECONDS);
```

**Interview pitfall**: `scheduleAtFixedRate` can have task overlap if execution exceeds period; `scheduleWithFixedDelay` guarantees gap between runs.

---

## 8) Future and CompletableFuture

### Future Interface

```java
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "Result";
});

// Blocking get
String result = future.get();                    // Blocks forever
String result = future.get(5, TimeUnit.SECONDS); // Timeout

// Non-blocking checks
boolean isDone = future.isDone();
boolean isCancelled = future.isCancelled();
boolean cancelled = future.cancel(true);  // true = interrupt if running
```

**Limitations of Future:**
- No way to chain operations
- Cannot combine multiple futures easily
- `get()` blocks

### CompletableFuture (Modern Async)

```java
// Creating
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<Void> cf2 = CompletableFuture.runAsync(() -> doWork());

// Chaining transformations
CompletableFuture<Integer> result = CompletableFuture
    .supplyAsync(() -> fetchData())
    .thenApply(data -> process(data))
    .thenApply(processed -> transform(processed));

// Side effects
cf.thenAccept(System.out::println);  // Consume result
cf.thenRun(() -> cleanup());          // Run action

// Exception handling
cf.exceptionally(ex -> fallbackValue)
  .thenAccept(System.out::println);

cf.handle((result, ex) -> {
    if (ex != null) return fallback;
    return transform(result);
});

// Combining futures
CompletableFuture<String> combined = cf1.thenCombine(cf2, (r1, r2) -> r1 + r2);

// Either first
CompletableFuture<String> first = cf1.applyToEither(cf2, Function.identity());

// All of
CompletableFuture<Void> all = CompletableFuture.allOf(cf1, cf2, cf3);

// Any of
CompletableFuture<Object> any = CompletableFuture.anyOf(cf1, cf2, cf3);
```

### Async Variants

```java
// Default: uses ForkJoinPool.commonPool()
cf.thenApply(x -> transform(x));

// Async: may use different thread
cf.thenApplyAsync(x -> transform(x));

// Async with custom executor
cf.thenApplyAsync(x -> transform(x), myExecutor);
```

---

## 9) Locks (java.util.concurrent.locks)

### ReentrantLock

More flexible than `synchronized`:

```java
private final ReentrantLock lock = new ReentrantLock();

public void doWork() {
    lock.lock();
    try {
        // Critical section
    } finally {
        lock.unlock();  // ALWAYS in finally!
    }
}
```

### tryLock (Non-blocking)

```java
if (lock.tryLock()) {
    try {
        // Got the lock
    } finally {
        lock.unlock();
    }
} else {
    // Lock not available - do something else
}

// With timeout
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    // ...
}
```

### ReentrantLock Features

```java
ReentrantLock lock = new ReentrantLock(true);  // Fair lock

lock.isHeldByCurrentThread();  // Check if current thread holds lock
lock.getHoldCount();           // Reentrant count
lock.isLocked();               // Check if any thread holds lock
lock.getQueueLength();         // Threads waiting for lock
```

### ReadWriteLock

Allows concurrent reads, exclusive writes:

```java
private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
private final Lock readLock = rwLock.readLock();
private final Lock writeLock = rwLock.writeLock();

public String read() {
    readLock.lock();
    try {
        return data;  // Multiple readers allowed
    } finally {
        readLock.unlock();
    }
}

public void write(String value) {
    writeLock.lock();
    try {
        data = value;  // Exclusive access
    } finally {
        writeLock.unlock();
    }
}
```

### StampedLock (Java 8+)

Optimistic locking for better read performance:

```java
private final StampedLock sl = new StampedLock();

public String optimisticRead() {
    long stamp = sl.tryOptimisticRead();
    String result = data;  // Read without lock
    if (!sl.validate(stamp)) {
        // Data may have changed, acquire read lock
        stamp = sl.readLock();
        try {
            result = data;
        } finally {
            sl.unlockRead(stamp);
        }
    }
    return result;
}
```

### Lock vs synchronized

| Feature | synchronized | Lock |
|---------|--------------|------|
| Syntax | Implicit | Explicit |
| Fairness | No control | Configurable |
| tryLock | No | Yes |
| Timeout | No | Yes |
| Multiple conditions | No | Yes |
| Interruptible | No | `lockInterruptibly()` |

---

## 10) Atomic Variables

For lock-free thread-safe operations on single variables:

### Common Atomic Classes

```java
AtomicInteger count = new AtomicInteger(0);
AtomicLong longVal = new AtomicLong(0);
AtomicBoolean flag = new AtomicBoolean(false);
AtomicReference<User> userRef = new AtomicReference<>(null);
```

### AtomicInteger Operations

```java
AtomicInteger count = new AtomicInteger(0);

count.incrementAndGet();    // ++count
count.getAndIncrement();    // count++
count.decrementAndGet();    // --count
count.addAndGet(5);         // count += 5
count.getAndAdd(5);         // return count, then count += 5

// Compare-and-swap (CAS)
count.compareAndSet(expected, newValue);  // Atomic if current == expected

// Update functions (Java 8+)
count.updateAndGet(x -> x * 2);
count.accumulateAndGet(5, (current, delta) -> current + delta);
```

### Compare-and-Swap (CAS) Pattern

```java
// Lock-free increment using CAS
public void safeIncrement(AtomicInteger count) {
    int current;
    int next;
    do {
        current = count.get();
        next = current + 1;
    } while (!count.compareAndSet(current, next));
}
```

### LongAdder (High Contention)

Better than `AtomicLong` for high-contention counters:

```java
LongAdder adder = new LongAdder();
adder.increment();
adder.add(10);
long sum = adder.sum();  // Get current value
```

**Why faster?** Maintains separate counters per thread, combines on read.

---

## 11) Concurrent Collections

### ConcurrentHashMap

Thread-safe map with high concurrency:

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Basic operations are atomic
map.put("key", 1);
map.putIfAbsent("key", 2);  // Only if absent
map.remove("key", 1);        // Only if value matches
map.replace("key", 1, 2);    // Only if current value is 1

// Atomic compute operations
map.compute("key", (k, v) -> v == null ? 1 : v + 1);
map.computeIfAbsent("key", k -> expensiveCompute(k));
map.computeIfPresent("key", (k, v) -> v + 1);
map.merge("key", 1, Integer::sum);  // Add or merge
```

**Interview point**: `ConcurrentHashMap` does NOT lock the entire map; uses fine-grained locking (Java 7: segments, Java 8+: per-bucket locks + CAS).

### CopyOnWriteArrayList

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
```

- **Reads**: Lock-free, always consistent
- **Writes**: Copy entire array (expensive)
- **Best for**: Read-heavy, write-rare scenarios (listeners, handlers)

### BlockingQueue Implementations

For producer-consumer patterns:

```java
// Bounded queues
BlockingQueue<Task> queue = new ArrayBlockingQueue<>(100);
BlockingQueue<Task> linked = new LinkedBlockingQueue<>(100);

// Unbounded (dangerous!)
BlockingQueue<Task> unbounded = new LinkedBlockingQueue<>();

// Priority-based
BlockingQueue<Task> priority = new PriorityBlockingQueue<>();

// Delayed elements
BlockingQueue<Delayed> delayed = new DelayQueue<>();

// Synchronous (no capacity - direct handoff)
BlockingQueue<Task> sync = new SynchronousQueue<>();
```

### BlockingQueue Operations

| Operation | Blocks | Throws | Returns special value | Timeout |
|-----------|--------|--------|----------------------|---------|
| Insert | `put(e)` | `add(e)` | `offer(e)` → false | `offer(e, time, unit)` |
| Remove | `take()` | `remove()` | `poll()` → null | `poll(time, unit)` |
| Examine | - | `element()` | `peek()` → null | - |

### ConcurrentSkipListMap/Set

Concurrent sorted collections (similar to TreeMap/TreeSet):

```java
ConcurrentSkipListMap<String, Integer> sortedMap = new ConcurrentSkipListMap<>();
ConcurrentSkipListSet<String> sortedSet = new ConcurrentSkipListSet<>();
```

---

## 12) Fork/Join Framework

For divide-and-conquer parallelism:

### RecursiveTask (Returns Result)

```java
class SumTask extends RecursiveTask<Long> {
    private final long[] array;
    private final int start, end;
    private static final int THRESHOLD = 10_000;
    
    public SumTask(long[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }
    
    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // Small enough: compute directly
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        } else {
            // Split into subtasks
            int mid = (start + end) / 2;
            SumTask left = new SumTask(array, start, mid);
            SumTask right = new SumTask(array, mid, end);
            
            left.fork();          // Submit left to pool
            Long rightResult = right.compute();  // Compute right in this thread
            Long leftResult = left.join();       // Wait for left
            
            return leftResult + rightResult;
        }
    }
}

// Usage
ForkJoinPool pool = ForkJoinPool.commonPool();
long sum = pool.invoke(new SumTask(array, 0, array.length));
```

### RecursiveAction (No Return Value)

```java
class SortAction extends RecursiveAction {
    private final int[] array;
    private final int start, end;
    
    @Override
    protected void compute() {
        if (end - start < THRESHOLD) {
            Arrays.sort(array, start, end);
        } else {
            int mid = (start + end) / 2;
            invokeAll(
                new SortAction(array, start, mid),
                new SortAction(array, mid, end)
            );
            merge(array, start, mid, end);
        }
    }
}
```

### Work Stealing

Fork/Join uses **work stealing**: idle threads steal tasks from busy threads' queues. This balances load automatically.

---

## 13) Synchronizers

### CountDownLatch

One-time barrier that allows threads to wait for a count to reach zero:

```java
CountDownLatch latch = new CountDownLatch(3);

// Worker threads
for (int i = 0; i < 3; i++) {
    executor.submit(() -> {
        doWork();
        latch.countDown();  // Decrement count
    });
}

// Main thread waits
latch.await();  // Blocks until count is 0
System.out.println("All workers completed");
```

### CyclicBarrier

Reusable barrier for multiple threads to wait at a common point:

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("All threads reached barrier");
});

// Each thread
doPhase1();
barrier.await();  // Wait for all threads
doPhase2();
barrier.await();  // Can be reused!
doPhase3();
```

### CountDownLatch vs CyclicBarrier

| Feature | CountDownLatch | CyclicBarrier |
|---------|----------------|---------------|
| Reusable | No | Yes |
| Threads | N threads count down, M wait | N threads wait together |
| Action | None | Optional barrier action |
| Reset | Not possible | `reset()` |

### Semaphore

Controls access to a limited number of resources:

```java
Semaphore semaphore = new Semaphore(3);  // 3 permits

public void accessResource() throws InterruptedException {
    semaphore.acquire();  // Get permit (blocks if none available)
    try {
        useResource();
    } finally {
        semaphore.release();  // Return permit
    }
}

// Non-blocking
if (semaphore.tryAcquire()) {
    try { useResource(); }
    finally { semaphore.release(); }
}
```

### Phaser

Flexible barrier (like CyclicBarrier but dynamic participation):

```java
Phaser phaser = new Phaser(1);  // Register main thread

for (int i = 0; i < 3; i++) {
    phaser.register();  // Dynamic registration
    executor.submit(() -> {
        doWork();
        phaser.arriveAndAwaitAdvance();
        doMoreWork();
        phaser.arriveAndDeregister();  // Leave
    });
}

phaser.arriveAndDeregister();  // Main thread leaves
```

### Exchanger

Pair of threads exchange objects:

```java
Exchanger<String> exchanger = new Exchanger<>();

// Thread 1
String fromThread2 = exchanger.exchange("data from thread 1");

// Thread 2
String fromThread1 = exchanger.exchange("data from thread 2");
```

---

## 14) Thread-Local Storage

Each thread gets its own copy of a variable:

```java
private static final ThreadLocal<SimpleDateFormat> dateFormat =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

public String formatDate(Date date) {
    return dateFormat.get().format(date);  // Thread-safe!
}
```

### Use Cases

- **Non-thread-safe objects** (SimpleDateFormat, Random)
- **Per-request context** (user info, transaction ID)
- **Avoiding parameter passing** through deep call stacks

### Cleanup

```java
try {
    threadLocal.set(value);
    // Use value
} finally {
    threadLocal.remove();  // IMPORTANT: Prevent memory leaks in pools!
}
```

**Interview warning**: Thread pools reuse threads. Failing to remove ThreadLocal values can leak data between requests.

### InheritableThreadLocal

Child threads inherit parent's value:

```java
private static final InheritableThreadLocal<String> context = 
    new InheritableThreadLocal<>();

context.set("parent-value");
new Thread(() -> {
    System.out.println(context.get());  // "parent-value"
}).start();
```

---

## 15) Common Concurrency Patterns

### Double-Checked Locking (Singleton)

```java
public class Singleton {
    private static volatile Singleton instance;  // volatile is CRITICAL!
    
    public static Singleton getInstance() {
        if (instance == null) {                  // First check (no lock)
            synchronized (Singleton.class) {
                if (instance == null) {          // Second check (with lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Better alternative**: Use enum or initialization-on-demand holder:

```java
public class Singleton {
    private static class Holder {
        static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

### Thread-Safe Lazy Initialization

```java
private final AtomicReference<ExpensiveObject> ref = new AtomicReference<>();

public ExpensiveObject get() {
    ExpensiveObject obj = ref.get();
    if (obj == null) {
        obj = new ExpensiveObject();
        if (!ref.compareAndSet(null, obj)) {
            obj = ref.get();  // Another thread won
        }
    }
    return obj;
}
```

### Producer-Consumer with BlockingQueue

```java
BlockingQueue<Task> queue = new LinkedBlockingQueue<>(100);

// Producer
executor.submit(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        Task task = createTask();
        queue.put(task);  // Blocks if full
    }
});

// Consumer
executor.submit(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        Task task = queue.take();  // Blocks if empty
        process(task);
    }
});
```

---

## 16) Common Pitfalls and Best Practices

### Pitfall 1: Synchronizing on Wrong Object

```java
// WRONG - New lock object every call!
public void bad() {
    synchronized (new Object()) {
        // Not protected!
    }
}

// WRONG - Autoboxing creates new objects
private Integer count = 0;
synchronized (count) {  // May sync on different objects!
    count++;
}
```

### Pitfall 2: Holding Locks During I/O

```java
// BAD - Thread blocked during network call
synchronized (lock) {
    String data = httpClient.fetch(url);  // Slow!
    process(data);
}

// BETTER - Minimize lock scope
String data = httpClient.fetch(url);
synchronized (lock) {
    process(data);
}
```

### Pitfall 3: Not Handling InterruptedException

```java
// BAD - Swallows interrupt
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // Silent - thread's interrupted status is cleared!
}

// GOOD - Restore interrupt status
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();  // Restore flag
    throw new RuntimeException("Interrupted", e);
}
```

### Best Practices Summary

1. **Prefer immutability** - Immutable objects are inherently thread-safe
2. **Minimize shared mutable state** - Less sharing = fewer synchronization issues
3. **Use high-level concurrency utilities** - `ExecutorService`, `BlockingQueue`, concurrent collections
4. **Keep synchronized blocks small** - Lock only what needs protection
5. **Document thread-safety** - Use `@ThreadSafe`, `@NotThreadSafe`, `@GuardedBy` annotations
6. **Test concurrency** - Use tools like JCStress, thread analyzers
7. **Prefer `volatile` for flags, atomics for counters** - Avoid over-synchronization
8. **Always unlock in finally** - Prevents lock leaks on exceptions

---

## 17) Interview Questions

### Fundamentals
1. What is the difference between `Runnable` and `Callable`?
2. Explain the thread lifecycle and all possible states.
3. What is the difference between `start()` and `run()`?
4. What are daemon threads? When would you use them?

### Synchronization
5. Explain the concept of a monitor in Java.
6. What is the difference between `synchronized` method and `synchronized` block?
7. Can a thread acquire multiple locks? What problems can arise?
8. Explain `wait()`, `notify()`, and `notifyAll()`. Why must they be in synchronized blocks?

### Memory Model
9. What is the Java Memory Model? Why do we need it?
10. Explain `volatile`. What guarantees does it provide?
11. What is happens-before relationship?
12. Why is double-checked locking broken without `volatile`?

### Executors
13. Explain the thread pool and its benefits.
14. What are different types of `ExecutorService`?
15. What is `CompletableFuture`? How is it better than `Future`?
16. How would you handle exceptions in a thread pool?

### Locks and Atomics
17. What is the difference between `ReentrantLock` and `synchronized`?
18. Explain `ReadWriteLock` and when to use it.
19. What is CAS (Compare-and-Swap)? How do atomic variables use it?
20. When would you use `LongAdder` instead of `AtomicLong`?

### Concurrent Collections
21. How does `ConcurrentHashMap` achieve thread-safety without locking the entire map?
22. What is the difference between `ConcurrentHashMap` and `Collections.synchronizedMap()`?
23. Explain `CopyOnWriteArrayList`. When would you use it?
24. What is a `BlockingQueue`? Explain different implementations.

### Advanced
25. Explain the Fork/Join framework. What is work stealing?
26. What is the difference between `CountDownLatch` and `CyclicBarrier`?
27. How would you detect and prevent deadlocks?
28. What are `ThreadLocal` variables? What are the pitfalls?
29. How do parallel streams use threading under the hood?
30. Design a thread-safe singleton.

---

## 18) Code Examples for Practice

### Example 1: Thread-Safe Counter

```java
public class ThreadSafeCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public int increment() {
        return count.incrementAndGet();
    }
    
    public int decrement() {
        return count.decrementAndGet();
    }
    
    public int get() {
        return count.get();
    }
}
```

### Example 2: Bounded Buffer

```java
public class BoundedBuffer<T> {
    private final BlockingQueue<T> queue;
    
    public BoundedBuffer(int capacity) {
        this.queue = new ArrayBlockingQueue<>(capacity);
    }
    
    public void put(T item) throws InterruptedException {
        queue.put(item);  // Blocks if full
    }
    
    public T take() throws InterruptedException {
        return queue.take();  // Blocks if empty
    }
    
    public int size() {
        return queue.size();
    }
}
```

### Example 3: Rate Limiter

```java
public class RateLimiter {
    private final Semaphore semaphore;
    private final int maxPermits;
    private final ScheduledExecutorService scheduler;
    
    public RateLimiter(int permitsPerSecond) {
        this.maxPermits = permitsPerSecond;
        this.semaphore = new Semaphore(permitsPerSecond);
        this.scheduler = Executors.newScheduledThreadPool(1);
        
        // Refill permits every second
        scheduler.scheduleAtFixedRate(() -> {
            semaphore.release(maxPermits - semaphore.availablePermits());
        }, 1, 1, TimeUnit.SECONDS);
    }
    
    public boolean tryAcquire() {
        return semaphore.tryAcquire();
    }
    
    public void acquire() throws InterruptedException {
        semaphore.acquire();
    }
}
```

### Example 4: Async Task Combiner

```java
public class AsyncTaskCombiner {
    public CompletableFuture<List<String>> fetchAllData(List<String> urls) {
        List<CompletableFuture<String>> futures = urls.stream()
            .map(url -> CompletableFuture.supplyAsync(() -> fetch(url))
                .exceptionally(ex -> "Error: " + ex.getMessage()))
            .collect(Collectors.toList());
        
        return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList()));
    }
    
    private String fetch(String url) {
        // HTTP fetch implementation
        return "data from " + url;
    }
}
```

