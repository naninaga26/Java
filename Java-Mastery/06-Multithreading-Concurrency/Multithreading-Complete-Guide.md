# Multithreading and Concurrency - Complete Guide

## Table of Contents
1. [Thread Basics](#thread-basics)
2. [Thread Lifecycle](#thread-lifecycle)
3. [Synchronization](#synchronization)
4. [Locks and Conditions](#locks-and-conditions)
5. [Concurrent Collections](#concurrent-collections)
6. [Executor Framework](#executor-framework)
7. [CompletableFuture](#completablefuture)
8. [Common Concurrency Problems](#common-concurrency-problems)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Thread Basics

### Creating Threads

```java
// Method 1: Extend Thread class
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

MyThread thread = new MyThread();
thread.start();  // ✅ Starts new thread
// thread.run();  // ❌ Runs in current thread (no multithreading!)

// Method 2: Implement Runnable (Preferred - allows class to extend other classes)
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable running: " + Thread.currentThread().getName());
    }
}

Thread thread2 = new Thread(new MyRunnable());
thread2.start();

// Method 3: Lambda (Java 8+)
Thread thread3 = new Thread(() -> {
    System.out.println("Lambda thread: " + Thread.currentThread().getName());
});
thread3.start();

// Method 4: ExecutorService (Preferred for production)
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(() -> {
    System.out.println("Executor thread: " + Thread.currentThread().getName());
});
executor.shutdown();
```

**Common Pitfall #1: Calling run() instead of start()**

```java
// ❌ WRONG: Calls run() in current thread (no new thread created!)
Thread thread = new Thread(() -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
});
thread.run();  // Runs in main thread!

// ✅ CORRECT: Creates and starts new thread
thread.start();  // Runs in new thread
```

### Thread Methods

```java
Thread thread = Thread.currentThread();

// Basic info
String name = thread.getName();
long id = thread.getId();
Thread.State state = thread.getState();
int priority = thread.getPriority();  // 1 (MIN) to 10 (MAX), default 5 (NORM)

// Setting properties
thread.setName("Worker-1");
thread.setPriority(Thread.MAX_PRIORITY);
thread.setDaemon(true);  // Daemon thread (JVM exits when only daemon threads remain)

// Thread control
thread.interrupt();  // Interrupt the thread
boolean interrupted = thread.isInterrupted();
Thread.sleep(1000);  // Sleep for 1 second (throws InterruptedException)
thread.join();  // Wait for thread to complete
thread.join(5000);  // Wait max 5 seconds
```

---

## Thread Lifecycle

```
┌──────────┐
│   NEW    │  Thread created but not started
└────┬─────┘
     │ start()
     ↓
┌──────────┐
│ RUNNABLE │  Thread ready to run (may be running or waiting for CPU)
└──┬───┬───┘
   │   │
   │   │ sleep(), wait(), I/O, lock
   │   ↓
   │ ┌─────────┐
   │ │ WAITING │  Waiting indefinitely for another thread
   │ │ TIMED   │  Waiting for specified time
   │ │ WAITING │
   │ └────┬────┘
   │      │ notify(), notifyAll(), timeout, I/O complete
   │      ↓
   │   [back to RUNNABLE]
   │
   │ Thread completes
   ↓
┌──────────┐
│TERMINATED│  Thread execution completed
└──────────┘
```

### Thread State Example

```java
public class ThreadStateDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        System.out.println("State after creation: " + thread.getState());  // NEW

        thread.start();
        System.out.println("State after start(): " + thread.getState());   // RUNNABLE

        Thread.sleep(100);  // Let thread start sleeping
        System.out.println("State while sleeping: " + thread.getState());  // TIMED_WAITING

        thread.join();  // Wait for completion
        System.out.println("State after completion: " + thread.getState());  // TERMINATED
    }
}
```

---

## Synchronization

### The Problem: Race Condition

```java
// ❌ UNSAFE: Race condition
class Counter {
    private int count = 0;

    public void increment() {
        count++;  // NOT atomic! (read, increment, write)
    }

    public int getCount() {
        return count;
    }
}

// Running with multiple threads:
Counter counter = new Counter();

Thread t1 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        counter.increment();
    }
});

Thread t2 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        counter.increment();
    }
});

t1.start();
t2.start();
t1.join();
t2.join();

System.out.println("Count: " + counter.getCount());  // Expected: 2000, Actual: < 2000 (race condition!)
```

**Why count++ is not thread-safe:**

```
Thread 1                    Thread 2                Memory (count)
─────────────────────────────────────────────────────────────────
Read count (0)                                      0
                            Read count (0)          0
Increment (0 + 1 = 1)                               0
Write count (1)                                     1
                            Increment (0 + 1 = 1)   1
                            Write count (1)         1 (Lost update!)

Expected: 2
Actual: 1
```

### Solution 1: Synchronized Method

```java
// ✅ SAFE: Synchronized method
class Counter {
    private int count = 0;

    public synchronized void increment() {  // Only one thread at a time
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}

// Now result is always 2000
```

### Solution 2: Synchronized Block

```java
class Counter {
    private int count = 0;
    private final Object lock = new Object();  // Lock object

    public void increment() {
        synchronized(lock) {  // Fine-grained locking
            count++;
        }
    }

    public int getCount() {
        synchronized(lock) {
            return count;
        }
    }
}

// Or use 'this' as lock
synchronized(this) {
    count++;
}
```

### Solution 3: Atomic Variables

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    private AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();  // Atomic operation
    }

    public int getCount() {
        return count.get();
    }
}
```

**Common Pitfall #2: Synchronizing on Wrong Object**

```java
// ❌ WRONG: Synchronizing on local variable (new lock each time!)
public void increment() {
    Object lock = new Object();  // New object each time!
    synchronized(lock) {  // Each thread has different lock!
        count++;
    }
}

// ✅ CORRECT: Synchronize on instance field
private final Object lock = new Object();

public void increment() {
    synchronized(lock) {  // All threads use same lock
        count++;
    }
}
```

### Deadlock

**The Problem:**

```java
// ❌ DEADLOCK SCENARIO
class BankAccount {
    private double balance;
    private final Object lock = new Object();

    public void transfer(BankAccount target, double amount) {
        synchronized(this) {  // Lock account1
            synchronized(target) {  // Lock account2
                this.balance -= amount;
                target.balance += amount;
            }
        }
    }
}

// Thread 1: account1.transfer(account2, 100)
// Thread 2: account2.transfer(account1, 50)
//
// Thread 1: Locks account1, waits for account2
// Thread 2: Locks account2, waits for account1
// → DEADLOCK!
```

**Solution: Lock Ordering**

```java
// ✅ DEADLOCK-FREE: Always lock in same order
class BankAccount {
    private final long id;
    private double balance;

    public void transfer(BankAccount target, double amount) {
        BankAccount first = (this.id < target.id) ? this : target;
        BankAccount second = (this.id < target.id) ? target : this;

        synchronized(first) {
            synchronized(second) {
                this.balance -= amount;
                target.balance += amount;
            }
        }
    }
}
```

---

## Locks and Conditions

### ReentrantLock

More flexible than synchronized.

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Counter {
    private int count = 0;
    private final Lock lock = new ReentrantLock();

    public void increment() {
        lock.lock();  // Acquire lock
        try {
            count++;
        } finally {
            lock.unlock();  // Always unlock in finally!
        }
    }

    // With try-lock (avoid indefinite waiting)
    public boolean tryIncrement() {
        if (lock.tryLock()) {  // Try to acquire, return immediately
            try {
                count++;
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false;  // Couldn't acquire lock
    }

    // With timeout
    public boolean incrementWithTimeout() throws InterruptedException {
        if (lock.tryLock(1, TimeUnit.SECONDS)) {  // Wait max 1 second
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
```

**Common Pitfall #3: Forgetting to Unlock**

```java
// ❌ WRONG: Lock never released if exception occurs
lock.lock();
count++;  // If exception here, lock never released!
lock.unlock();

// ✅ CORRECT: Always unlock in finally
lock.lock();
try {
    count++;
} finally {
    lock.unlock();  // Always executes
}
```

### ReadWriteLock

Allows multiple readers OR one writer.

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class SharedResource {
    private int value = 0;
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();

    public int read() {
        rwLock.readLock().lock();  // Multiple threads can read simultaneously
        try {
            return value;
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void write(int newValue) {
        rwLock.writeLock().lock();  // Only one thread can write
        try {
            value = newValue;
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}
```

### Condition Variables

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.LinkedList;
import java.util.Queue;

class BoundedQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public BoundedQueue(int capacity) {
        this.capacity = capacity;
    }

    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await();  // Wait until queue not full
            }
            queue.offer(item);
            notEmpty.signal();  // Notify waiting consumers
        } finally {
            lock.unlock();
        }
    }

    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();  // Wait until queue not empty
            }
            T item = queue.poll();
            notFull.signal();  // Notify waiting producers
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

---

## Concurrent Collections

### ConcurrentHashMap

Thread-safe HashMap without locking entire map.

```java
import java.util.concurrent.ConcurrentHashMap;

Map<String, Integer> map = new ConcurrentHashMap<>();

// Thread-safe operations
map.put("key", 1);
map.get("key");
map.remove("key");

// Atomic operations (Java 8+)
map.putIfAbsent("key", 1);  // Only if key doesn't exist
map.computeIfAbsent("key", k -> expensiveComputation(k));
map.computeIfPresent("key", (k, v) -> v + 1);
map.merge("key", 1, Integer::sum);  // Increment by 1

// No ConcurrentModificationException during iteration
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    map.put("new", 42);  // Safe! (though may or may not be visible in iteration)
}
```

**ConcurrentHashMap vs Hashtable vs Collections.synchronizedMap:**

| Feature | ConcurrentHashMap | Hashtable | synchronizedMap |
|---------|-------------------|-----------|-----------------|
| Locking | Segmented locking | Entire map | Entire map |
| Performance | High (concurrent reads) | Low | Low |
| Null keys | No | No | Depends on map |
| Null values | No | No | Depends on map |
| Fail-fast iteration | No | Yes | Yes |
| Use | **Preferred** | Legacy | Rarely |

### CopyOnWriteArrayList

Thread-safe List optimized for read-heavy scenarios.

```java
import java.util.concurrent.CopyOnWriteArrayList;

List<String> list = new CopyOnWriteArrayList<>();

// Writes create a copy of array (expensive!)
list.add("A");  // Copies entire array
list.add("B");  // Copies entire array again

// Reads are very fast (no locking)
String item = list.get(0);

// Iteration never throws ConcurrentModificationException
for (String s : list) {
    list.add("C");  // Safe! Iterator uses snapshot
}

// Use when:
// - Many reads, few writes
// - Iteration far more common than modification
// - Small to medium sized lists

// Don't use when:
// - Frequent writes (expensive copying)
// - Large lists
```

### BlockingQueue

Thread-safe queue with blocking operations.

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ArrayBlockingQueue;

// Producer-Consumer pattern
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);

// Producer thread
Thread producer = new Thread(() -> {
    try {
        for (int i = 0; i < 100; i++) {
            queue.put(i);  // Blocks if queue full
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

// Consumer thread
Thread consumer = new Thread(() -> {
    try {
        while (true) {
            Integer item = queue.take();  // Blocks if queue empty
            process(item);
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

producer.start();
consumer.start();
```

---

## Executor Framework

### ThreadPool Types

```java
import java.util.concurrent.*;

// 1. Fixed Thread Pool - Fixed number of threads
ExecutorService fixed = Executors.newFixedThreadPool(5);
// Use: Bounded number of concurrent tasks

// 2. Cached Thread Pool - Creates threads as needed, reuses idle threads
ExecutorService cached = Executors.newCachedThreadPool();
// Use: Many short-lived tasks

// 3. Single Thread Executor - Sequential execution
ExecutorService single = Executors.newSingleThreadExecutor();
// Use: Tasks must be executed sequentially

// 4. Scheduled Thread Pool - Delayed/periodic execution
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(3);
// Use: Scheduled tasks

// Custom ThreadPoolExecutor (preferred for production)
ExecutorService custom = new ThreadPoolExecutor(
    5,  // Core pool size
    10,  // Maximum pool size
    60, TimeUnit.SECONDS,  // Keep-alive time for excess threads
    new LinkedBlockingQueue<>(100),  // Work queue
    new ThreadPoolExecutor.CallerRunsPolicy()  // Rejection policy
);
```

### Submitting Tasks

```java
ExecutorService executor = Executors.newFixedThreadPool(5);

// 1. execute() - Fire and forget (Runnable only)
executor.execute(() -> {
    System.out.println("Task executed");
});

// 2. submit() - Returns Future (Runnable or Callable)
Future<?> future1 = executor.submit(() -> {
    System.out.println("Task submitted");
});

// 3. submit() with Callable - Returns result
Future<Integer> future2 = executor.submit(() -> {
    return 42;
});

try {
    Integer result = future2.get();  // Blocks until result available
    System.out.println("Result: " + result);
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}

// 4. submit() with timeout
try {
    Integer result = future2.get(5, TimeUnit.SECONDS);  // Wait max 5 seconds
} catch (TimeoutException e) {
    future2.cancel(true);  // Cancel if timeout
}

// Always shutdown executor
executor.shutdown();  // Graceful shutdown (finish existing tasks)
// executor.shutdownNow();  // Forceful shutdown (interrupt running tasks)

// Wait for termination
executor.awaitTermination(1, TimeUnit.MINUTES);
```

### invokeAll and invokeAny

```java
ExecutorService executor = Executors.newFixedThreadPool(5);

List<Callable<Integer>> tasks = Arrays.asList(
    () -> { Thread.sleep(1000); return 1; },
    () -> { Thread.sleep(2000); return 2; },
    () -> { Thread.sleep(3000); return 3; }
);

// invokeAll - Wait for ALL tasks to complete
List<Future<Integer>> futures = executor.invokeAll(tasks);
for (Future<Integer> future : futures) {
    System.out.println(future.get());  // 1, 2, 3
}

// invokeAny - Wait for ANY task to complete (cancel others)
Integer result = executor.invokeAny(tasks);  // Returns 1 (first to complete)

executor.shutdown();
```

**Common Pitfall #4: Forgetting to Shutdown Executor**

```java
// ❌ WRONG: Executor keeps JVM alive
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(() -> System.out.println("Task"));
// JVM doesn't exit! (non-daemon threads still running)

// ✅ CORRECT: Always shutdown
try {
    // ... submit tasks
} finally {
    executor.shutdown();
    executor.awaitTermination(1, TimeUnit.MINUTES);
}
```

---

## CompletableFuture

Asynchronous programming with callbacks (Java 8+).

### Basic Usage

```java
import java.util.concurrent.CompletableFuture;

// Create CompletableFuture
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // Runs in ForkJoinPool.commonPool()
    return "Hello";
});

// Get result (blocking)
String result = future.get();  // "Hello"

// With custom executor
ExecutorService executor = Executors.newFixedThreadPool(5);
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    return "World";
}, executor);
```

### Chaining Operations

```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")  // Transform result
    .thenApply(String::toUpperCase)  // "HELLO WORLD"
    .thenAccept(System.out::println)  // Consume result
    .thenRun(() -> System.out.println("Done"));  // Run after completion

// Async versions (run in different thread)
future
    .thenApplyAsync(s -> s + " World")  // Runs in ForkJoinPool
    .thenAcceptAsync(System.out::println, customExecutor);  // Custom executor
```

### Combining Multiple Futures

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 20);

// Combine two futures
CompletableFuture<Integer> combined = future1.thenCombine(future2, (a, b) -> a + b);
System.out.println(combined.get());  // 30

// Wait for both (ignore results)
CompletableFuture<Void> both = CompletableFuture.allOf(future1, future2);
both.join();  // Waits for both to complete

// Wait for any (race)
CompletableFuture<Object> any = CompletableFuture.anyOf(future1, future2);
System.out.println(any.get());  // Whichever completes first
```

### Error Handling

```java
CompletableFuture.supplyAsync(() -> {
    if (true) throw new RuntimeException("Error!");
    return "Success";
})
.exceptionally(ex -> {
    System.out.println("Error: " + ex.getMessage());
    return "Default";  // Fallback value
})
.thenAccept(System.out::println);  // Prints: "Default"

// Or use handle() for both success and error
future.handle((result, ex) -> {
    if (ex != null) {
        return "Error occurred";
    }
    return result;
});
```

### Real-World Example: Parallel API Calls

```java
public CompletableFuture<UserDetails> getUserDetails(String userId) {
    CompletableFuture<User> userFuture =
        CompletableFuture.supplyAsync(() -> userService.getUser(userId));

    CompletableFuture<List<Order>> ordersFuture =
        CompletableFuture.supplyAsync(() -> orderService.getOrders(userId));

    CompletableFuture<Address> addressFuture =
        CompletableFuture.supplyAsync(() -> addressService.getAddress(userId));

    // Combine all results
    return userFuture.thenCombine(ordersFuture, (user, orders) -> {
        return addressFuture.thenApply(address -> {
            return new UserDetails(user, orders, address);
        });
    }).thenCompose(Function.identity());  // Flatten nested CompletableFuture
}
```

---

## Common Concurrency Problems

### 1. Race Condition
**Problem:** Multiple threads access shared data without proper synchronization.
**Solution:** Use synchronized, locks, or atomic variables.

### 2. Deadlock
**Problem:** Two threads wait for each other's locks.
**Solution:** Lock ordering, tryLock with timeout, avoid nested locks.

### 3. Livelock
**Problem:** Threads keep changing state in response to each other without making progress.
**Solution:** Add randomness to retry logic.

### 4. Starvation
**Problem:** Thread never gets CPU time (low priority, other threads monopolize).
**Solution:** Fair locks, proper thread priority management.

### 5. Memory Visibility
**Problem:** Changes made by one thread not visible to others.
**Solution:** Use volatile, synchronized, or atomic variables.

```java
// ❌ PROBLEM: Memory visibility
class SharedFlag {
    private boolean flag = false;  // May be cached by thread!

    public void writer() {
        flag = true;  // May not be visible to reader thread
    }

    public void reader() {
        while (!flag) {  // May loop forever!
            // Wait
        }
    }
}

// ✅ SOLUTION: Use volatile
class SharedFlag {
    private volatile boolean flag = false;  // Guaranteed visibility

    public void writer() {
        flag = true;  // Immediately visible to all threads
    }

    public void reader() {
        while (!flag) {
            // Will exit when flag becomes true
        }
    }
}
```

---

## Best Practices

1. **Prefer higher-level concurrency utilities** over wait/notify
2. **Use ExecutorService** instead of manually creating threads
3. **Always shutdown executors** to prevent resource leaks
4. **Use concurrent collections** instead of synchronized wrappers
5. **Minimize synchronization scope** - lock only what's necessary
6. **Avoid nested locks** - prevents deadlocks
7. **Use AtomicXXX** for simple atomic operations
8. **Document thread-safety** of your classes
9. **Use CompletableFuture** for asynchronous programming
10. **Test with different thread counts** and loads

---

## Interview Questions

### Q1: What is the difference between process and thread?
**Answer:** Process is an independent program with its own memory space. Thread is a lightweight unit within a process, sharing memory with other threads. Threads are faster to create and context-switch than processes.

### Q2: What is the difference between synchronized method and synchronized block?
**Answer:** Synchronized method locks the entire object (this). Synchronized block can lock specific object, providing finer-grained control and better performance.

### Q3: What is deadlock? How to prevent it?
**Answer:** Deadlock occurs when two threads wait for each other's locks indefinitely. Prevention:
- Lock ordering (always acquire locks in same order)
- Use tryLock() with timeout
- Avoid nested locks
- Use deadlock detection tools

### Q4: Difference between wait() and sleep()?
**Answer:**
- wait(): Releases lock, must be called in synchronized block, woken by notify()
- sleep(): Doesn't release lock, can be called anywhere, wakes after timeout

### Q5: What is volatile keyword?
**Answer:** volatile ensures visibility - changes made by one thread are immediately visible to others. Prevents compiler/CPU from caching variable. Use for flags and state variables. Doesn't provide atomicity (use AtomicXXX for that).

### Q6: Explain happens-before relationship.
**Answer:** Guarantees visibility and ordering in multithreading:
- Write to volatile happens-before read of same variable
- Unlock happens-before lock of same monitor
- Thread.start() happens-before any action in started thread
- All actions in thread happen-before join() returns

### Q7: What is thread pool? Why use it?
**Answer:** Thread pool is a collection of reusable threads. Benefits:
- Reuses threads (no creation overhead)
- Limits concurrency (prevents resource exhaustion)
- Task queue management
- Better performance for many short-lived tasks

### Q8: Difference between CountDownLatch and CyclicBarrier?
**Answer:**
- CountDownLatch: One-time use, threads wait for count to reach zero
- CyclicBarrier: Reusable, threads wait for each other at barrier point

---

## Summary

Multithreading is complex but powerful. Key takeaways:

1. Use high-level utilities (ExecutorService, CompletableFuture)
2. Understand synchronization and locks
3. Be aware of concurrency problems (deadlock, race conditions)
4. Use concurrent collections when possible
5. Always properly shutdown thread pools
6. Test thoroughly with different scenarios

Would you like me to continue with more guides or create a summary document?
