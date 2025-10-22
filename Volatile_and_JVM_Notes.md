# 🧠 Java Volatile, JVM Memory Model & Thread-Safe Singleton  
**Lecture — Oct 21 2025 (David Lu)**

---

## 🧩 1. JVM Architecture – Runtime Data Areas

### 🧠 Purpose
The Java Virtual Machine (JVM) divides memory into regions that define what data is visible to each thread. Understanding which regions are **shared** vs **private** is critical for diagnosing thread-safety issues.

| Memory Area | Shared Across Threads | Description | Key Notes |
|--------------|----------------------|--------------|-----------|
| **Stack** | ❌ | Each thread has its own stack containing local variables, method frames, and return addresses. | Thread-isolated; never a source of concurrency issues. |
| **Heap** | ✅ | Stores all dynamically allocated objects and arrays created via `new`. | Shared by all threads → primary source of race conditions. |
| **Method Area (Metaspace)** | ✅ | Stores class metadata, static fields, and constant pools. | Shared; static variables are common thread conflict points. |
| **PC Register** | ❌ | Tracks the current bytecode instruction for each thread. | Private to each thread; ensures separate instruction flow. |
| **Native Method Stack** | ❌ | Used for JNI (C/C++) method calls. | Independent per thread; integrates with OS libraries. |

**Important Behavioral Rules**
- **Heap objects** are shared references; modifying them requires synchronization.  
- **Stack variables** are thread-local; safe without locks.  
- **Static fields** belong to the class, not instances → also shared.  
- **Thread safety problems = shared mutable state (Heap + Method Area).**

---

## ⚙️ 2. `final`, `finally`, `finalize`

### `final`
Used to enforce immutability or prevent extension/override.

| Usage | Example | Effect |
|--------|----------|--------|
| Final Class | `public final class Math {}` | Cannot be subclassed. |
| Final Method | `public final void show() {}` | Cannot be overridden. |
| Final Variable | `private final int age = 18;` | Cannot be reassigned once initialized. |

#### Immutability Pattern (from lecture)
1. Mark the class `final` to prevent inheritance.  
2. Declare all fields `private final`.  
3. Provide **getters only** (no setters).  
4. For mutable fields (e.g. `List`), return a **deep copy** or an **unmodifiable view**.

```java
public final class Person {
    private final String name;
    private final List<String> hobbies;

    public Person(String name, List<String> hobbies) {
        this.name = name;
        this.hobbies = new ArrayList<>(hobbies); // defensive copy
    }
    public List<String> getHobbies() {
        return Collections.unmodifiableList(hobbies); // prevents external modification
    }
}
```

### `finally`
- Used in `try-catch-finally` blocks.  
- Always executes regardless of exceptions, unless the JVM terminates.  
- Used for **resource cleanup** (close streams, sockets, or DB connections).

### `finalize` (Deprecated)
- Called before garbage collection.  
- Replaced by `AutoCloseable` + `try-with-resources` since Java 7.  
- Unreliable for cleanup; execution timing not guaranteed.

---

## 🧱 3. Thread-Safe Singleton Design Patterns

### ⚠️ Non-Thread-Safe Lazy Singleton
```java
public class Singleton {
    private static Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();  // race condition here
        }
        return instance;
    }
}
```
- If two threads enter the `if` block simultaneously, both create instances.  
- Common beginner mistake — **not synchronized during first initialization**.

---

### ✅ Eager Initialization (Thread-Safe by Class Loading)
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() { return INSTANCE; }
}
```
- Created when the class is first loaded into the JVM Method Area.  
- JVM guarantees **class initialization is atomic** and synchronized internally.  
- **Pros:** Simple and always safe.  
- **Cons:** Instance created even if unused → unnecessary memory.

---

### ✅ Synchronized Lazy Singleton
```java
public class Singleton {
    private static Singleton instance;
    public static synchronized Singleton getInstance() {
        if (instance == null) instance = new Singleton();
        return instance;
    }
}
```
- Thread-safe but **slow** due to synchronized access every call.  
- Locks entire method → reduces concurrency.  
- Acceptable only for small-scale or demo applications.

---

## 🧮 4. Double-Checked Locking (DCL) + `volatile`

### Motivation
To avoid locking after initialization while still ensuring thread safety.

### Implementation
```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {                   // 1st check (no lock)
            synchronized (Singleton.class) {
                if (instance == null) {           // 2nd check (inside lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### Why Two Checks?
- **First check:** avoids unnecessary synchronization after initialization.  
- **Second check:** ensures only one thread creates the object.  

### Why `volatile`?
Object creation happens in 3 CPU-level steps:
1. Allocate memory.  
2. Initialize fields.  
3. Assign reference to `instance`.  
Without `volatile`, compiler/CPU may reorder as 1 → 3 → 2 → another thread sees partially constructed object.  
`volatile` enforces write-ordering and visibility guarantees.

---

## ⚡ 5. `volatile` Keyword in Depth

### Core Guarantees
1. **Visibility:**  
   - Every write to a `volatile` variable is immediately visible to all threads.  
   - Reads always fetch from **main memory**, not CPU cache.
2. **Ordering:**  
   - Prevents instruction reordering for reads/writes around volatile variables.  
   - Ensures operations before a volatile write happen-before any subsequent volatile read.

### Limitations
- Does **not** ensure atomicity for compound operations (`count++`).  
- Does **not** provide mutual exclusion → multiple threads can still write concurrently.

### CPU Memory Model Analogy
- Threads keep **local CPU caches** for speed.  
- Normal writes may remain cached → other threads see stale data.  
- Marking a field as `volatile` forces flush to **main memory**.  
- JVM inserts **memory barriers** to guarantee correct synchronization between caches.

### Example – Visibility Guarantee
```java
volatile boolean running = true;

public void worker() {
    while (running) {
        // busy work
    }
}

public void stop() {
    running = false;  // immediately visible to all threads
}
```
Without `volatile`, worker thread might **never stop** if it keeps reading the old cached `true` value.

### Example – Preventing Reordering
```java
private static volatile Singleton instance;
```
- Prevents creation of partially initialized objects (1→3→2 problem).  
- Guarantees `instance` reference points to a fully constructed object before visible to others.

---

## 🔒 6. Critical Section, Lock Granularity, and Synchronization Strategies

### Definition
A **critical section** is a portion of code where shared mutable data is accessed or modified. Only one thread should execute it at a time to maintain data consistency.

### Locking Guidelines
1. Lock the **smallest possible code block** that modifies shared state.  
2. Avoid synchronizing entire methods unless necessary.  
3. Minimize time spent inside locks to reduce contention.

**Example – Fine-grained Lock**
```java
synchronized (list) {
    list.add(item); // only critical operation locked
}
```

**Bad Example – Coarse-grained Lock**
```java
public synchronized void addItem(String item) {
    list.add(item); // entire method locked, even non-critical code
}
```

### Lock Choice Strategy
| Scenario | Lock Type | Benefit |
|-----------|------------|----------|
| Frequent writes | Pessimistic (`synchronized`, `Lock`) | Ensures strict exclusion |
| Many reads, few writes | Optimistic (CAS, Atomic Classes) | Better throughput |
| Balanced load | `ReentrantReadWriteLock` | Concurrent reads, exclusive writes |

### CAS (Compare-And-Swap)
- CPU-level atomic instruction: checks expected value → replaces if matches.  
- Eliminates blocking but may retry multiple times.  
- **Problem:** ABA — value changes A→B→A still passes CAS.  
  - Fix: use `AtomicStampedReference` or timestamped versions.

### ReentrantLock vs `synchronized`
| Feature | `synchronized` | `ReentrantLock` |
|----------|----------------|----------------|
| Lock release | Automatic (JVM) | Manual (`unlock()` in finally block) |
| Try lock | ❌ | ✅ (`tryLock()`) |
| Fair scheduling | ❌ | ✅ (`new ReentrantLock(true)`) |
| Condition variables | ❌ | ✅ (`newCondition()`) |
| Performance | Simpler | Finer control, slightly higher overhead |

**Example – ReentrantLock with Condition**
```java
Lock lock = new ReentrantLock();
Condition cond = lock.newCondition();

lock.lock();
try {
    cond.await();    // waits until signaled
    cond.signal();   // resumes waiting thread
} finally {
    lock.unlock();
}
```

---

## ✅ Summary Table

| Concept | Ensures | Typical Use |
|----------|----------|-------------|
| `synchronized` | Mutual exclusion + visibility | Critical sections |
| `volatile` | Visibility + ordering | Flags, DCL Singleton |
| `Lock` | Manual locking control | Fine-grained coordination |
| `Atomic*` | Lock-free atomic ops | Counters, accumulators |
| `final` immutability | State cannot change | Shared config objects |

---
