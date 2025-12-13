---
title: >-
  33. Lock-Free Concurrency | Compare-and-Swap | Atomic & Volatile Variables |
  Multithreading Part5 - YouTube
description: >-
  ➡️ Notes link: Shared in the Member Community Post (If you are Member of this
  channel, then pls check the Member community post, i have shared the Notes
  link...
source: >-
  https://www.youtube.com/watch?v=JGb4qNEBW6Q&list=PL6W8uoQQ2c63f469AyV78np0rbxRFppkx&index=35
created: "2025-12-13"
tags:
  - hover-notes
  - youtube
---

### ## Concurrency
- Achieved via two main mechanisms:
- **Lock-based mechanism**:
    - `Synchronized`
    - `Reentrant`
    - `Stamped`
    - `ReadWrite`
    - `Semaphores`
- **Lock-free mechanism**:
    - `CAS Operation (Compare-and-Swap)`
    - `AtomicInteger`
    - `AtomicBoolean`
    - `AtomicLong`
    - `AtomicReference`
- Lock-free mechanism is faster than lock-based.
- Lock-free mechanism is not an alternative to lock-based mechanism.
- Lock-based mechanisms are used in scenarios with high complexity and business logic.
- Lock-free mechanisms are used in very specific areas.
- Lock-free mechanisms are used in specific use cases and can achieve faster results than lock-based.
- **CAS Operation (Compare-and-Swap)**: A technique used to achieve lock-free mechanism.
- Java provides four classes (`AtomicInteger`, `AtomicBoolean`, `AtomicLong`, `AtomicReference`) that use CAS operation.
- **Optimistic Concurrency Control**: A lock-free protocol.
- Example: A row in a table with `roll number` 123 and name `Raj`.
- A row in a table has `roll number` 123, name `Raj` and `row version` 1.
- Thread 1 and Thread 2 read the same row with `roll number` 123 and record the `row version` as 1.
- Thread 1 modifies the name to `Rajk` and Thread 2 modifies the name to `Raja`.
- Both threads are trying to update the DB.
- Thread 1 updates the DB by running an update query.
- The query sets the name to `Rajk` where the `roll number` is 123 and the `row version` is 1.
- Since the row version matches, the update is successful.
- When Thread 1 successfully updates the row, the `row version` is also incremented (e.g., from 1 to 2). This is a crucial part of the optimistic concurrency control mechanism.
- Now, when Thread 2 attempts its update, its query still specifies `row version = 1`.
- Since the actual `row version` in the database is now 2 (due to Thread 1's update), Thread 2's update query will *fail* because the `WHERE` clause condition (`row version = 1`) is no longer met.
- This prevents Thread 2 from overwriting Thread 1's changes, as it detects that the data has been modified by another thread since it last read it.
## Optimistic Concurrency Control (continued)
- When Thread 2's initial update fails:
    - It must **re-read** the data from the database.
    - It will then obtain the new `row version` (e.g., `2`).
    - It modifies the name to `Raja` again.
    - It attempts to update the database with the new `row version` (`UPDATE table SET Name = 'Raja' WHERE RollNo = 123 AND RV = 2`).
    - If successful, the `row version` is incremented again (e.g., to `3`).
### CAS (Compare and Swap) Technique
- This technique is fundamentally the same as optimistic locking but operates at a lower level.
- It is a **low-level operation**.
- It is **atomic**.
- It is supported by **modern processors (CPU)**.
- It involves **3 main parameters**:
    -   `Memory location`: The specific location where the variable is stored.
    -   `Expected Value`: The value that is *expected* to be currently present at the memory location.
        -   **Note**: The ABA problem can be solved by using a version number or a timestamp with the expected value.
    -   `New Value`: The value to be written to the memory location, but *only if* the current value matches the `Expected Value`.
### CAS (Compare and Swap) Technique (continued)
- **Atomic**: Means single unit.
- No matter how many cores the CPU has (one core, two core, three core, four core CPU).
- Even if multiple threads are running in parallel, it provides atomicity of the CAS operation.
- All modern processors support CAS.
### CAS (Compare and Swap) Technique (continued)
- CAS operation takes three parameters:
    - `Memory`: Loads the variable data.
        - Load or read variable from memory, e.g. M1.
    - Compare
- `Compare`: Compares the `Memory data` with the `Expected Value`. If they *don't match*, it means the value has been changed by another thread, and no update occurs. If they *do match*, then the `New Value` is written to memory. This is why it's called "Compare and Swap".
### Atomic Variables
- **What ATOMIC means:** "Single" or "all or nothing". The operation completes entirely or not at all, ensuring no partial updates.
- Optimistic locking is inspired by the CAS operation.
- CAS (Compare and Swap) operation is a low-level operation inside a CPU.
### Optimistic Concurrency Control
- Primarily works with databases (DB).
### CAS (Compare and Swap) Technique
- The Java `Atomic` classes (like `AtomicInteger`, `AtomicBoolean`, `AtomicLong`, `AtomicReference`) provide ways to *access* and utilize the low-level CAS operation.
- The ABA problem is a scenario that *can happen* in concurrent programming, where a value changes from A to B and then back to A, leading to a false positive in a simple CAS check. This is why versioning or timestamps are often used.
- **Example of CAS operation:**
- Imagine a memory location `M1` currently holds the value `10`.
- A CAS operation might look like this:

```javascript
CAS(MemoryLocation: M1, ExpectedValue: 10, NewValue: 12)
```
- This operation first `Read`s the value from `M1`.
- Then, it `Compare`s the read value with the `ExpectedValue` (`10`).
- If they match, the `NewValue` (`12`) is `Swap`ped into `M1`.
- If they don't match, the operation fails (meaning another thread changed the value), and no swap occurs.
### CAS (Compare and Swap) Technique (continued)
- CAS operation:
    - Reads the memory location `M1` value.
    - Compares the `M1` value with the `Expected Value`.
    - If they match, updates `M1` to the `New Value`.
- **ABA Problem**: A scenario where a value changes from A to B and then back to A.
    - A thread might read the value as A, and by the time it attempts a CAS operation, the value is again A, even though it has been modified in the interim.
    - This can lead to incorrect updates.
    - To solve the ABA problem, use version numbers or timestamps.
- **ABA Problem**: Can be resolved by adding a version number to the memory location.
- Example:
    - Initial value in memory location `M1` is `10` (Version 1).
    - Value changed to `12` (Version 2).
    - Value changed back to `10` (Version 3).
    - If a thread attempts a CAS operation expecting `10` with Version 1, the operation will fail because the current version is `3`.
- Optimistic concurrency is similar to CAS.
### Atomic Variables
- **Atomic**: Means single or "all or nothing".
- An operation completes entirely or not at all, ensuring no partial updates.
- Example: `int counter = 10;` is an atomic operation.
- It is a single operation and no matter how many threads execute this, it would be consistent.
- Consider a `SharedResource` class with a counter variable and an `increment` method (`counter++`).
- The `increment` method increments the counter.
### Atomic Variables (continued)
- `counter++` is NOT an atomic operation.
    - It involves three steps:

        1.  Load counter value.
        2.  Increment.
        3.  Update the counter.
- `counter++` is NOT an atomic operation because it involves three steps:

    1. Load counter value.
    2. Increment by 1.
    3. Assign back.
- `counter++` is equivalent to `counter = counter + 1`.
- The three operations (load, increment, assign) of `counter++` are not atomic and not a single unit.
- If two threads (T1 and T2) run in parallel and both try to increment the counter:
    - Initially, `counter = 0`.
    - Both threads load the counter value (0).
    - Both threads increment the counter value to 1.
- After both threads increment the counter value to 1, and assign the value back, the counter value is 1. However, it should be 2.
- This is because `counter++` is not an atomic operation and not thread-safe.
- When two threads load the counter value, they might read the same value, increment it by one, and assign it back leading to loss of data.
- When the code from the previous example is run, the output is 371 instead of 400 because `counter++` is not an atomic operation (not a single operation).
- `counter++` is a three-step process and not thread-safe.
- To make it thread-safe, there are two solutions:

    1.  Using lock like `synchronized`
    2.  Using lock free operation like `AtomicInteger`
- If the `increment` method is marked as `synchronized`, the problem will be resolved.
- When the `increment` method is marked as `synchronized`, no matter how many threads try to increment, only one thread will go inside at a time.
### Atomic Variables (continued)
- To make `counter++` thread-safe, there are two solutions:

    1.  Using lock like `synchronized`
    2.  Using lock free operation like `AtomicInteger`
- If the `increment` method is marked as `synchronized`, only one thread will go inside at a time.
- **Lock-free operation**: Using `AtomicInteger`.
    - `AtomicInteger` internally uses **CAS** (Compare and Swap).
    - Change the shared resource `integer` to `atomic integer`.
    - `AtomicInteger counter = new AtomicInteger(0);`
    - Instead of `counter++`, use `counter.incrementAndGet();`
### Atomic Variables (continued)
- Example of using `AtomicInteger` to make `counter++` thread-safe:
- Change the shared resource `integer` to `atomic integer`.
- `AtomicInteger counter = new AtomicInteger(0);`
- Instead of `counter++`, use `counter.incrementAndGet();`
- `incrementAndGet()` increments the counter by one.
- If you need to increment by a specific value, use `addAndGet(int delta)`. This method adds delta to the current value.
- The `incrementAndGet()` method internally uses `compareAndSwapInt()` (CAS) operation.
### Atomic Variables (continued)
- `compareAndSwapInt()` method internally uses **CAS** (Compare and Swap) operation.
- **CAS** operation takes three parameters:
    - `Object o`: object/array to update the field/element in.
    - `long offset`: field/element offset.
    - `int expected`: the expected value.
    - `int x`: the new value.
- **When to use Atomic**: When you have a simple use case of:
    - Read
    - Modify
    - Update
- Example: `counter = counter + 1` involves three steps:
    - Read the value
    - Modify it (add +1)
    - Update
### Atomic Variables (continued)
- If you need to use `Read`, `Modify`, and `Update`, use `Atomic`.
- `AtomicInteger counter = new AtomicInteger(0);`
    - Initial atomic integer that is passed is zero.
- It is lock-free and faster.
- The initial value is set with `new AtomicInteger()`.
- `AtomicInteger` is an atomic integer class.
- The value is set to zero and it is in memory.

\`\`\`
### AtomicInteger and Memory
- `val` is a reference to memory.
- The `incrementAndGet()` method:
    - Reads the value from memory.
    - Has an expected value.
### AtomicInteger and Memory (continued)
- `val` is marked as `volatile`.
- Because `val` is volatile, it always reads from memory, not cache.
- The `incrementAndGet()` method:
    - Reads the value from memory.
    - Has an expected value.
    - Has a new value.
- The CAS operation will continue to try until it is successful.
- Code Snippet:

```java
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!compareAndSwapInt(o, offset, v, v + delta));
    return v;
}
```
- `o`: object/array to update the field/element in
- `offset`: field/element offset
- `v`: the value to add
### AtomicInteger and CAS Details
- When `incrementAndGet()` is called:
    - It reads the value from memory.
    - The `delta` is nothing but +1.
- The `incrementAndGet()` method:
    - Reads the value from memory. The initial value read from memory is zero.
    - Passes CAS until the memory, expected value, and new value are correct.
    - The expected value is zero.
    - The new value is `delta + 1` (0 + 1 = 1).
- `AtomicInteger` provides a way to access the CPU-level low-level operation (CAS operation).
### AtomicInteger and CAS Operation
- `incrementAndGet()` internally uses CAS operation.
- Steps of CAS (Compare and Swap) Operation:

    1. Load data from memory.
    2. Compare the loaded data with the expected value.
    3. If the values match, update the memory with the new value.
### AtomicInteger and CAS Operation (continued)
- Let's consider a scenario with two threads, T1 and T2.
- Both threads call the `increment` method at the same time.
- The `increment` method involves the following steps (internally within `incrementAndGet()`):
    - Read the value from memory.
    - Compare (using CAS) the value in memory with the expected value.
    - If the values match, update the memory with the new value.
### AtomicInteger and CAS Operation (continued)
- Two threads (T1 and T2) call the `increment` method at the same time.
- The `increment` method involves the following steps (internally within `incrementAndGet()`):
    - Read the value from memory.
    - Compare (using CAS) the value in memory with the expected value.
    - If the values match, update the memory with the new value.
- Thread 1 reads the value from memory as zero (`l = 0`).
- Thread 2 reads the value from memory as zero (`l = 0`).
- CAS operation is atomic and the CPU provides the guarantee that this operation is atomic.
- Only one thread can go inside the atomic operation.
- Let's say thread 1 goes inside.
- The CAS operation will first read data from memory.
### AtomicInteger and CAS Operation (continued)
- Thread 1 reads the value from memory as zero (`l = 0`).
- Thread 2 reads the value from memory as zero (`l = 0`).
- The CAS operation involves the following steps:

    1.  Read data from memory.
    2.  Compare the loaded data with the expected value.
    3.  If the values match, update the memory with the new value.
- CAS operation is atomic, and the CPU provides the guarantee that this operation is atomic.
- Only one thread can go inside the atomic operation.
- Let's say thread 1 goes inside.
- The CAS operation will first read data from memory.
- Then, it will compare the expected value with the memory value.
- The expected value is zero.
- In the memory, it was also zero, so it will update.
- It will update it to one.
- Thread 1 is a success.
- Thread 2 is doing the CAS operation, and it will:
    - Read the data from memory.
    - The data in memory is now one.
    - Compare the expected with the memory value.
    - Expected is zero, but in the memory it is one.
    - It doesn't match.
    - This will return false.
- If CAS operation fails, the thread reads the value from memory again and retries the CAS operation until successful.
- Thread 2 reads the value from memory again. `l` is now 1 for Thread 2.
- Thread 2 tries to perform a CAS operation again.
- Now for Thread 2, it reads from memory which is 1, and the expected value is also 1. Because 1 == 1, then update the increment is now 2 for thread 2.
- The `incrementAndGet()` method in `AtomicInteger` internally uses `compareAndSwapInt()` (CAS) operation.
- The `compareAndSwapInt()` method is a native method implemented in C++ or C.
- Java provides access to this CPU-level operation through the `Atomic` class.
- If there's a scenario like read, modify, and update, go for atomic, otherwise go for lock.
- Many engineers get confused with atomic with volatile.
- Atomic provides thread-safe operations.
- Volatile has no relation with concurrency.
### Volatile vs Atomic
- **Atomic**: Thread-safe operation.
- **Volatile**: No relation with concurrency (not thread-safe).
- **CPU Core**: Each CPU core has its own L1 cache.
- There can be an L2 cache.
- Memory (RAM).

```javascript

```

```javascript

```
### Volatile vs Atomic (continued)
- Two threads (T1 and T2) each running on different CPU cores.
- Both threads are working on a shared variable `x`, initially set to 10 (`x = 10`).
- Thread 1 increments `x` (`x++`).
- The incremented value (11) is stored in Thread 1's L1 cache (`x = 11`).
- Thread 2 attempts to read the value of `x`.
- Thread 2 first checks its L1 cache, but `x` is not present.
- Thread 2 then checks the L2 cache, but `x` is also not present.
- Thread 2 finally reads the value of `x` from main memory (RAM), which is still 10.
### Volatile vs Atomic (continued)
- L1 cache first stores the data, and periodically this L1 cache gets synced up with each other.
- If the sync doesn't happen, and another thread tries to read the `x` value, it will check its L1 cache, then L2 cache, and then eventually goes to memory, which has the older value (10 instead of 11).
- **Volatile**: Ensures that read and write operations happen directly from main memory (RAM).
- When Thread 1 increments `x` and Thread 2 tries to read `x`:
    - If `x` is declared as `volatile`, Thread 2 will read the updated value (11) directly from main memory, bypassing the caches.
- If `x` is not `volatile`, Thread 2 might read the old value (10) from its cache or main memory before the caches are synchronized.
- When a variable `x` is declared as `volatile`, any thread trying to read `x` will read it directly from main memory (RAM), bypassing L1 and L2 caches.
- Similarly, any write operation to a `volatile` variable will be written directly to main memory.
- **Volatile** makes sure that any changes which are done by one thread is visible to another thread also.
### Volatile vs Atomic (continued)
- When a variable is declared as `volatile`, read/write operations happen directly from main memory (RAM), not the cache.
- **Volatile**: Ensures that any changes made by one thread are immediately visible to other threads.
- Multiple threads are running in separate environments, without interfering with each other.
- **Atomic**: Thread-safe operation.
- **Volatile**: No relation with concurrency (not thread-safe).
- Don't mix atomic and volatile.
### Volatile vs Atomic (continued)
- Volatile ensures that any changes made by one thread are immediately visible to other threads, but it doesn't guarantee concurrency.
- Concurrency is guaranteed by atomicity through CAS operation.
- If you understand `AtomicInteger`, then `AtomicBoolean` and `AtomicLong` are the same.
### Thread-safe Collection Alternatives
- For each standard collection, there exists a thread-safe alternative.
- Examples:
    - `PriorityQueue` -> `PriorityBlockingQueue`
    - `LinkedList` -> `ConcurrentLinkedDeque`
    - `ArrayDeque` -> `ConcurrentLinkedDeque`
    - `ArrayList` -> `CopyOnWriteArrayList`
    - `HashSet` -> `ConcurrentHashMap` (using `newKeySet()`)
    - `TreeSet` -> `Collections.synchronizedSortedSet()`
    - `LinkedHashSet` -> `Collections.synchronizedSet()`
    - `Queue Interface` -> `ConcurrentLinkedQueue`
- Thread-safe alternatives often use:
    - `ReentrantLock`
    - `Compare-and-swap operation`
    - `Synchronized`
### Thread-safe Collection Alternatives
- Let's look at what mechanisms (locks or CAS) these concurrent collections actually use:

| Collection | Concurrent Collection | Lock Mechanism |
| --- | --- | --- |
| PriorityQueue | PriorityBlockingQueue | ReentrantLock |
| LinkedList | ConcurrentLinkedDeque | Compare-and-swap operation |
| ArrayDeque | ConcurrentLinkedDeque | Compare-and-swap operation |
| ArrayList | CopyOnWriteArrayList | ReentrantLock |
| HashSet | newKeySet() inside ConcurrentHashMap | Synchronized |
| TreeSet | Collections.synchronizedSortedSet() | Synchronized |
| LinkedHashSet | Collections.synchronizedSet() | Synchronized |
| Queue Interface | ConcurrentLinkedQueue | Compare-and-swap operation |
- For `PriorityBlockingQueue`, the `offer` method uses a `ReentrantLock` (`final ReentrantLock lock = this.lock; lock.lock();`).