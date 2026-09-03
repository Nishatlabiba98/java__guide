# 11 — Concurrency Basics

Running more than one thing at a time — and the ways that goes wrong if you're not careful.

## Threads

The basic unit of concurrent execution:

```java
Thread t = new Thread(() -> {
    System.out.println("running on: " + Thread.currentThread().getName());
});
t.start();     // starts a new thread — never call run() directly, that just runs it on the current thread
t.join();      // blocks the calling thread until t finishes
```

Creating raw `Thread`s directly is rare in real code — almost everything below exists to avoid it.

## Executors

A managed pool of reusable threads — you submit work, the pool handles thread lifecycle:

```java
ExecutorService pool = Executors.newFixedThreadPool(4);

Future<Integer> result = pool.submit(() -> {
    return expensiveComputation();
});

int value = result.get();     // blocks until the result is ready

pool.shutdown();               // stop accepting new tasks; let running ones finish
```

Always shut a pool down when you're done with it — a leaked, unbounded thread pool is a common source of production resource leaks.

## Concurrency vs. parallelism

Related but distinct: **concurrency** is structuring a program to handle multiple tasks that make progress independently (can be on one core, interleaved). **Parallelism** is actually running things simultaneously (needs multiple cores). A single-core machine can be concurrent but never truly parallel.

## Common race-condition traps

Two threads reading/writing shared state without coordination:

```java
class Counter {
    private int count = 0;
    void increment() { count++; }   // NOT atomic — read, add, write are 3 separate steps
}
```

Two threads calling `increment()` "simultaneously" can both read the same value before either writes back, losing an update. Fixes, in order of preference:

```java
// 1. Use an atomic type — no locking needed, built for exactly this
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();

// 2. Synchronize the critical section — only one thread at a time inside
synchronized void increment() { count++; }

// 3. Use a higher-level concurrent collection instead of a plain HashMap/ArrayList
Map<String, Integer> safeMap = new ConcurrentHashMap<>();
```

Never use a plain `HashMap` or `ArrayList` from multiple threads without external synchronization — corruption is silent and hard to reproduce, not a clean crash.

## Deadlock

Two threads each holding a lock the other needs:

```
Thread A: locks resource1, waits for resource2
Thread B: locks resource2, waits for resource1
// neither ever proceeds
```

Avoid by always acquiring locks in the same order everywhere in your codebase, and keeping locked sections as small as possible.

## Timers

Scheduling work to run later or repeatedly. `ScheduledExecutorService` is preferred over the older `java.util.Timer` (single-threaded, one bad task can block everything else scheduled on it):

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
scheduler.scheduleAtFixedRate(() -> System.out.println("tick"), 0, 5, TimeUnit.SECONDS);
```

## Exercise: parallel word counter

Given a list of file paths, count the total number of words across all files using an `ExecutorService` — submit one task per file (each returning its word count as a `Future<Integer>`), then sum the results once every future completes. Compare the wall-clock time against a single-threaded version reading the files sequentially.

## Checklist

- [ ] Can explain the difference between concurrency and parallelism
- [ ] Know why `count++` isn't thread-safe and at least two ways to fix it
- [ ] Comfortable submitting tasks to an `ExecutorService` and retrieving results via `Future`
- [ ] Can explain what causes a deadlock and one way to avoid it
- [ ] Parallel word counter produces the same total as the sequential version
