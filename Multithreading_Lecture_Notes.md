# 🧠 Java Multithreading — Oct 17 2025 (David Lu Detailed Notes)

---

## 🧩 1. Project Context & Purpose
- ~90 % of the course project uses **Spring / Spring Boot / Spring MVC**.  
  - However, **core Java multi-threading** knowledge is **mandatory** for both:
    - backend development in production;  
    - and interview readiness.  
- Real-world backend services rely on **thread pools**, **asynchronous programming**, and **request handling**.
- Multi-threading underpins key frameworks:
  - **Tomcat thread pool**, **Spring Async**, **WebFlux**, **ExecutorService**.  
- Lecture’s aim: understand *how threads actually work* in Java before touching Spring abstractions.

---

## 🧱 2. Program vs Process vs Thread
**Definitions (instructor emphasized practical viewpoint):**
- **Program** = static code on disk (e.g., `.java` / `.class` files).   
  - Contains logic but **does nothing** until loaded into memory.
- **Process** = instance of that program running in memory.  
  - OS allocates resources (RAM, CPU time, file handles).  
  - Multiple processes can coexist but are isolated.  
- **Thread** = execution path **within** a process.  
  - All threads share the same heap and method area but have **their own stack**.  

**Memory visibility keywords:**
- `volatile` → guarantees that updates to a variable are immediately visible to other threads; prevents CPU caching reordering.  
- Without `volatile` → one thread may read stale data.

**Example given:**  
When two threads operate on a shared counter without synchronization, results become inconsistent → classic **race condition**.

---

## ⚙️ 3. Java Virtual Machine (JVM) Execution
- Java’s portability comes from the **JVM** → executes bytecode on any OS.  
- Compilation flow:  
  `.java` → `.class` (bytecode) → loaded by **ClassLoader** → stored in **Method Area** → executed in **JVM process**.  
- Components:
  - **Heap** → shared objects.  
  - **Stack** → local variables & method calls per thread.  
  - **Method Area** → class definitions, static variables, constants.  
  - **PC Register** → next instruction per thread.  
  - **Native Method Stack** → links to C/C++ functions.  

**`native` keyword:**
- Declares methods implemented outside Java (in C/C++).  
- Example:  
  `private native void start0();` in `Thread` class.  
- Allows JVM to call OS-level thread APIs for creation/scheduling.  
- *Critical point:* thread creation always crosses into this native layer.

---

## 💾 4. Hardware Memory Hierarchy
**Speed vs Cost trade-off (lecture example):**
| Type | Speed | Size | Cost | Use |
|------|--------|------|------|-----|
| CPU Register | fastest | few bytes | highest | immediate instruction data |
| Cache (L1/L2/L3) | very fast | MB | high | reduce main-memory latency |
| RAM | fast | GBs | medium | runtime data |
| Disk | slow | TBs | cheap | storage |
| Virtual Memory | slowest | — | — | disk space used when RAM full |

- Virtual RAM (swap file) = disk-based extension of RAM; slower but prevents OOM.  
- JVM heap allocation is bounded by available physical RAM.

---

## 🔄 5. Concurrent vs Parallel Programming
- **Concurrent** = task switching on limited CPU cores (one core handling multiple threads).  
- **Parallel** = true simultaneous execution on multiple cores.  
- Even single-core programs benefit from concurrency → responsiveness.
- **Context switching** cost ≈ saving & loading thread state → balance performance with design complexity.

---

## 🧵 6. Thread Creation in Java
**Four ways discussed:**
1. **Extend `Thread`**
   ```java
   class MyThread extends Thread {
       public void run() { System.out.println("Hello"); }
   }
   new MyThread().start();
   ```

2. **Implement `Runnable`**
   ```java
   class MyTask implements Runnable {
       public void run() { System.out.println("Task"); }
   }
   new Thread(new MyTask()).start();
   ```

3. **Implement `Callable<V>`**
   ```java
   Callable<Integer> task = () -> 42;
   Future<Integer> f = Executors.newSingleThreadExecutor().submit(task);
   ```

4. **Thread Pool**
   - `ExecutorService` handles thread lifecycle.  
   - *In industry, only this is acceptable* for scalability.

**Instructor quote:**  
> “In a web app, never use `new Thread()`; always use a thread pool.”

---

## 🧩 7. Virtual Threads & JDK Milestones
**JDK Timeline:**
- **JDK 8 (2014)** → Lambdas, Streams, `CompletableFuture`.  
- **JDK 11 (2018)** → Flight Recorder (free).  
- **JDK 17 (2021)** → LTS; required by Spring 6 / Boot 3.  
- **JDK 19 → 21** → Virtual Threads (Project Loom) stabilize.

**Virtual Threads (JEP 425):**
- JVM-scheduled fibers; thousands per OS thread.  
- Lightweight; nearly zero context-switch overhead.  
- Ideal for high-concurrency (HTTP servers, I/O bound tasks).  
- Example:
  ```java
  Thread.ofVirtual().start(() -> handleRequest());
  ```

---

## 🔍 8. Thread Lifecycle & Native Details
**Six States:**
1. New  
2. Runnable  
3. Blocked  
4. Waiting  
5. Timed Waiting  
6. Terminated  

**Key APIs:**
- `start()` → creates OS thread via `start0()` (native).  
- `run()` → executes synchronously → no new thread.  
- Once `start()` called, thread cannot be restarted.  

**Daemon threads:**
- Background services → GC, signal handlers.  
- JVM exits when only daemons remain.

---

## ⚙️ 9. Thread Pools (`ThreadPoolExecutor`)
**Why:**  
- Creating threads at runtime triggers `start0()` (native call) → expensive OS interaction.  
- Pooling creates threads upfront → reuse threads → predictable performance.

**7 config parameters:**
1. `corePoolSize`
2. `maximumPoolSize`
3. `keepAliveTime`
4. `TimeUnit`
5. `workQueue`
6. `threadFactory`
7. `rejectionHandler`

**Lifecycle inside pool:**
1. Task arrives → use idle core thread.  
2. Core full → enqueue task.  
3. Queue full → spawn up to max threads.  
4. All full → apply handler.

**Built-in pool analysis:**
- `Executors.newCachedThreadPool()`  
  - core = 0, max = 2³¹ − 1, keep-alive = 60 s.  
  - Risk: unlimited threads → `OutOfMemoryError`.  

**Best practice:**
- Always **define custom bounded pools** with `ThreadPoolExecutor`.  
- Example:
  ```java
  ExecutorService pool = new ThreadPoolExecutor(
      10, 20, 30, TimeUnit.SECONDS,
      new ArrayBlockingQueue<>(100),
      Executors.defaultThreadFactory(),
      new ThreadPoolExecutor.AbortPolicy()
  );
  ```

---

## ⚡ 10. Asynchronous Programming with `CompletableFuture`
- Before Java 8 → `Future` + `Callable`: `get()` blocks main thread.  
- Java 8 → `CompletableFuture`: async chaining (`thenApply`, `thenAccept`, `thenRun`, `thenCombine`).  
- Uses callbacks; non-blocking.  
- Ideal for dependent microservice API calls.

**Example:**
```java
CompletableFuture.supplyAsync(() -> fetchUser())
    .thenCompose(user -> fetchOrders(user))
    .thenAccept(System.out::println);
```

---

## 🧠 11. Interview & Performance Expectations
- Employers emphasize correctness *and* efficiency.  
- Demonstrate:
  - Fluent understanding of thread life cycle.  
  - Difference between `start()` vs `run()`.  
  - Reasons to prefer pools over raw threads.  
  - Proper pool configuration rationale.  
  - Knowledge of JDK milestones and features used.

---

## 🧮 12. Clarifications & Follow-ups
- **ExecutorService = core Java**, not Spring.  
- **Spring** builds on top of these concepts (`@Async`, `TaskExecutor`).  
- **Action Items:**
  - Post code handler demo.  
  - Research Java feature milestones.  

---

## 🧰 13. Supplementary Concepts
- **Synchronization tools:** `synchronized`, `ReentrantLock`, `ReadWriteLock`.  
- **Thread-safe structures:** `ConcurrentHashMap`, `BlockingQueue`.  
- **Atomic classes:** `AtomicInteger`, `AtomicReference`.  
- **ThreadLocal:** per-thread variable storage.  
- **Deadlock prevention:** lock ordering, `tryLock`, minimize nested locks.  
- **Wait/Notify:** inter-thread communication via monitor.

---

## 🧱 14. Modern Features (JDK 21+)
- **Virtual Threads:** lightweight, cooperative-scheduling threads.  
- **Structured Concurrency:** manage task groups collectively.  
- **Project Panama:** native interop improvements.

---

## 📚 15. Recommended Resources
- *Java Concurrency in Practice* — Brian Goetz.  
- *Effective Java* (Items 78–84).  
- Oracle Docs: `java.util.concurrent`.  
- JEP 425 — Virtual Threads.  
- JEP 453 — Structured Concurrency.  
