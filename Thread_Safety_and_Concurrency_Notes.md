# 🧠 Java Thread Safety & Concurrency — Oct 20 2025 (David Lu Detailed Notes)

---

## ⚙️ 1. Core Concept — Thread Safety
- **Thread Safety Definition:**  
  When multiple threads access shared data, the program ensures correctness and consistency regardless of thread scheduling.
- **Where issues occur:**  
  → When multiple threads *read and write* shared data in the **heap memory** (shared region).  
  → Each thread has its own **stack**, so no cross-thread interference happens there.
- **Memory areas reviewed:**  
  - Stack  →  per-thread, stores local variables, method frames.  
  - Heap  →  shared, objects and instance fields.  
  - Method Area  →  class metadata, static fields, constants (shared).  
  - PC Register  →  per-thread instruction pointer.  
  - Native Method Stack  →  for JNI calls.  
- **Key takeaway:**  
  Thread safety issues → always about **shared mutable state in heap or method area**.

---

## 🧩 2. How Threads Interact with JVM Memory
- **Runnable execution model:**
  - `Runnable` object → created in heap.  
  - Its class code → stored in Method Area.  
  - Reference to the Runnable → pushed onto each thread’s stack.  
- **Each thread has independent frames:**  
  - Main thread → main method frame.  
  - New threads → own frames; stack data not visible to others.  
- **Shared resources = heap objects only**.

---

## 🧮 3. Understanding Thread Safety Problems
- **Cause:**  
  Concurrent read/write access to shared objects without proper synchronization.  
- **Analogy (teacher’s example):**  
  High number of customers ≠ unsafe; unsafe only when they modify shared data simultaneously.  
- **Occurs in:**  
  - Heap area (shared objects).  
  - Static fields (method area).

---

## 🔒 4. Solutions to Thread Safety Issues
### Pessimistic Locking (assume conflict)
- **Keyword:** `synchronized` or `Lock` interface.  
- **Mechanism:**  
  One thread locks the critical section; others wait.  
- **Forms:**  
  - Method-level synchronization.  
  - Code-block synchronization (`synchronized(this){}` or on shared object).  
- **Traits:**  
  - Ensures mutual exclusion (Mutex).  
  - Reentrant → a thread can re-acquire the same lock.  
- **Trade-offs:**  
  - Too-wide locking (scope = whole method) → performance drop.  
  - Lock only critical section for efficiency.  
  - Context switch cost increases with contention.

### Optimistic Locking (assume no conflict)
- **Used in:** high-read, low-write systems (e.g., caches, user profiles).  
- **Technique:** **Compare-And-Swap (CAS)** + **versioning**.  
- **Steps:**  
  1. Read value and version.  
  2. Compute new value.  
  3. Update if version unchanged.  
- **Problem:** **ABA Problem**  
  - Value A → B → A again; CAS sees same A and incorrectly succeeds.  
  - Fix: use **AtomicStampedReference** or timestamped version.  
- **When to use:**  
  - High concurrency with few writes.  
  - e.g., stock price reads, user profile views.

### Choosing Lock Type Based on Traffic
| Traffic Pattern | Recommended Lock | Reason |
|-----------------|-----------------|---------|
| Frequent reads + writes | Pessimistic | Avoid constant CAS failures |
| Many reads, few writes | Optimistic | Minimize blocking |
| Heavy writes | Pessimistic | Guaranteed atomicity |
| Light traffic | Either | Performance difference minimal |

---

## 🧱 5. Critical Section Optimization
- Lock only the smallest part that modifies shared state.  
- Example:  
  ```java
  synchronized(list) {
      list.add(item); // small critical section
  }
  ```
- Avoid:
  ```java
  public synchronized void process() { ... } // entire method locked
  ```
- Too-broad locking → thread contention → reduced throughput.

---

## 🔐 6. `Lock` Interface (Advanced Pessimistic Lock)
- Provides greater control than `synchronized`.  
- Typical usage:  
  ```java
  Lock lock = new ReentrantLock();
  try {
      lock.lock();
      // critical section
  } finally {
      lock.unlock();
  }
  ```
- Advantages:  
  - Can try to acquire (`tryLock()` non-blocking).  
  - Condition variables → precise thread coordination.  
  - Fair locks (`new ReentrantLock(true)`).

---

## ⚙️ 7. Thread Lifecycle and CPU Execution
- **Six states:** New → Runnable → Running → Blocked → Waiting → Terminated.  
- **CPU scheduling:** OS-controlled, non-deterministic.  
- **Thread priority (1–10):** hints CPU but no guarantee.  
- **Key concepts:**  
  - Runnable ≠ Running (state ≠ execution).  
  - CPU cycles (frequency) depend on flip-flop latches (1s / 0s).

---

## 🧭 8. Condition Variables and Thread Coordination
- **Problem:** `Lock` alone cannot control execution order.  
- **Solution:** `Condition` object for signaling.  
  ```java
  Condition condition = lock.newCondition();
  lock.lock();
  try {
      condition.await();    // wait
      condition.signal();  // notify
  } finally { lock.unlock(); }
  ```
- Allows precise thread sequencing beyond `synchronized` mechanism.

---

## 🧩 9. Singleton Pattern Thread Safety
| Loading Mode | Thread Safety | Explanation |
|---------------|---------------|--------------|
| Eager Loading | ✅ Safe | Instance created in Method Area at class load time → no race condition |
| Lazy Loading | ⚠️ Unsafe | Instance created on first call → needs `synchronized` protection |
| Double-checked Locking | ✅ Safe (Java 5+) | Combine lazy loading + volatile instance |
| Enum Singleton | ✅ Recommended | JVM ensures thread safety and serialization |

**Eager Singleton example:**
```java
public class Singleton {
  private static final Singleton INSTANCE = new Singleton();
  private Singleton() {}
  public static Singleton getInstance() { return INSTANCE; }
}
```

---

## 🧠 10. Interview & Practical Takeaways
- Understand *heap vs stack vs method area sharing*.  
- Distinguish *pessimistic vs optimistic locks*.  
- Know *CAS and ABA problem solutions*.  
- Explain how `synchronized` and `Lock` interface differ.  
- Discuss *when and why to use volatile*.  
- Show awareness of *trade-offs between performance and safety*.  
- Be able to draw JVM memory layout + thread interaction.

---

## 📊 11. Practical Design Guidelines
- Identify shared mutable state → protect it.  
- Use atomic classes (`AtomicInteger`, `AtomicReference`).  
- Avoid nested locks (deadlock risk).  
- Use concurrent collections (`ConcurrentHashMap`, `CopyOnWriteArrayList`).  
- Log thread IDs for debugging (`Thread.currentThread().getName()`).  
- Tune thread pools according to hardware cores and I/O behavior.

---

## 🧩 12. Decision Framework for Lock Selection
| Scenario | Read/Write Ratio | Recommended Lock | Example Use |
|-----------|-----------------|------------------|-------------|
| Bank transaction updates | Many writes | Pessimistic | DB update, fund transfer |
| E-commerce product views | Many reads, few writes | Optimistic | Inventory check |
| Realtime caching | Mostly reads | Optimistic | User session cache |
| Logging system | Sequential writes | Pessimistic | File I/O |
| Financial auditing | Critical accuracy | Atomic Stamped + Pessimistic | Balance verification |

---

## 🧭 13. Next Steps
- Implement Singleton thread safety demo.  
- Benchmark `synchronized` vs `ReentrantLock`.  
- Practice JVM memory diagram labeling in interviews.  
- Study Java Memory Model (JMM) rules — visibility, ordering, atomicity.
