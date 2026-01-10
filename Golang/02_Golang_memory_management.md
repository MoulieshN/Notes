# Topics Index

| No | Topic | YouTube Reference |
|----|------|------------------|
| 1 | [Stack vs Heap in Go](#stack-vs-heaps-in-go) | https://www.youtube.com/watch?v=ZMZpH4yT7M0 |
| 2 | [How variables are allocated (Stack vs Heap)](#how-do-we-know-when-a-variable-is-allocated-to-the-heap) | nil |
| 3 | [Escape Analysis](#escape-analysis) | nil |
| 4 | [Dangling Pointers](#what-is-dangling-pointers) | nil |
| 5 | [Why Go Has No Dangling Pointers](#why-go-does-not-have-dangling-pointers) | nil
| 6 | [Garbage Collector (GC) in Go](#gc-garbage-collector-link-to-gc) | https://www.youtube.com/watch?v=q4HoWwdZUHs |



# Stack vs Heaps in Go

### Stack Memory:
- Stack memory is used for storing local variables and function call data, including function parameters and return addresses.
- Variables allocated on the stack have a fixed size and are typically short-lived, as they are deallocated automatically when the function that created them exits.
- Go automatically manages the stack memory, and you don’t have to worry about memory deallocation for stack-based variables.
Basic data types, such as integers, floats, and pointers, are often stored on the stack.
- Variables with a known, fixed size at compile time are generally allocated on the stack.

### Heap Memory:
- Heap memory is used for dynamically allocated data that doesn’t have a fixed or known size at compile time.
- Variables allocated on the heap are typically longer-lived, and their memory must be explicitly managed by the programmer (e.g., through garbage collection).
- Complex data structures like slices, maps, and structs containing slices or maps often use heap memory.
- Strings in Go are implemented as read-only byte slices (immutable), and the actual string data is typically stored in the heap.
- When you use the make function to create slices, maps, or channels, they are allocated on the heap.

---

# How do we know when a variable is allocated to the heap?

Go compilers will allocate variables that are local to a function in that function’s stack frame. However, if the compiler cannot prove that the variable is not referenced after the function returns, then the compiler must allocate the variable on the garbage-collected heap to avoid dangling pointer errors. Also, if a local variable is very large, it might make more sense to store it on the heap rather than the stack.

If a variable has its address taken, that variable is a candidate for allocation on the heap. However, a basic escape analysis recognizes some cases when such variables will not live past the return from the function and can reside on the stack.


# Escape analysis

The escape analysis carried out by the Go compiler determines whether a variable should be allocated on the stack or heap. A variable is allocated on the heap if it “escapes” its scope, or if it can be accessed after its function completes.

```go
package main

import "fmt"

// This function returns an integer pointer.
// The integer i is created within the function scope,
// but because we're returning the address of i, it "escapes" from the function.
// The Go compiler will decide to put this on the heap.
func escapeAnalysis() *int {
    i := 10 // i is initially created here, within the function's scope
    return &i // The address of i is returned here, which means it "escapes" from the function
}

// This function also returns an integer, but the integer does not escape
// This integer will be stored on the stack as it doesn't need to be accessed outside the function.
func noEscapeAnalysis() int {
    j := 20 // j is created here, within the function's scope
    return j // The value of j is returned here, but it doesn't escape from the function
}

func main() {
    // Call both functions and print the results
    fmt.Println(*escapeAnalysis()) // Output: 10
    fmt.Println(noEscapeAnalysis()) // Output: 20
}

```

In the escapeAnalysis() function, the variable i "escapes" because its address is returned by the function. This means that the variable i needs to be available even after the function has finished executing. Therefore, it will be stored on the heap.

In contrast, in the noEscapeAnalysis() function, the variable j does not escape because only its value is returned. Therefore, it can be safely disposed of after the function finishes, and it will be stored on the stack.

The Go compiler automatically performs escape analysis, so you don’t need to explicitly manage stack and heap allocation. This simplifies memory management and helps to prevent memory leaks and other errors.


You can use escape analysis to determine where a specific variable will be stored.

For example, you can analyze your application by compiling it from the command line with the -gcflags=-m flag:

> `go build -gcflags=-m main.go`

### What is dangling pointers?
A dangling pointer is a pointer that still points to a memory location whose lifetime has ended.
Accessing it leads to undefined behavior because the memory may already be reused or freed.

In other languages like (C/C++)

```c
int* foo() {
    int x = 10;
    return &x; // ❌ x is destroyed after function returns
}
```

- x lives on the stack
- Stack frame is destroyed after return
- Pointer now points to invalid memory

### Why Dangling Pointers Are Dangerous

- Causes segmentation faults
- Leads to data corruption
- Security vulnerabilities
- Extremely hard to debug
- No compile-time detection in C/C++

### Why Go Does NOT Have Dangling Pointers

Go avoids dangling pointers by design:

- Garbage collection
- No manual free
- Escape analysis
- No pointer arithmetic
- Safe memory lifetime tracking

```go
func foo() *int {
    x := 10
    return &x // safe: x moved to heap
}
```
----

# GC (Garbage Collector) [Link to GC](https://medium.com/better-programming/memory-optimization-and-garbage-collector-management-in-go-71da4612a960)

Go provides **automatic memory management** using **Garbage Collection (GC)**, which efficiently reclaims unused memory to prevent memory leaks and manual deallocation.

- **How Go’s GC Works**
    
    Go’s GC is a **concurrent, tricolor, non-blocking garbage collector**, optimized for **low-latency applications**.
    
    ### **Key Features:**
    
    - **Concurrent** → Runs alongside goroutines, minimizing **stop-the-world (STW) pauses**.
    - **Tricolor Mark-and-Sweep** → Uses **white, gray, and black** sets to track object reachability.
    - **Non-Compacting** → Frees memory but does not move objects (avoiding fragmentation).
    - **Write Barriers** → Ensures consistency during concurrent GC operations.
  
- **The Tricolor Mark-and-Sweep Algorithm**
    
    ### **1️⃣ Mark Phase**
    
    - Objects are categorized into three **colors**:
        - **White** → Unreachable (will be collected).
        - **Gray** → Discovered but not fully processed.
        - **Black** → Reachable and fully processed.
    - GC scans **roots (global vars, stack, heap)** and moves reachable objects from **white → gray → black**.
    
    ### **2️⃣ Sweep Phase**
    
    - Unreachable **white** objects are **deleted** (memory freed).
    - Uses **background threads** to minimize impact on application performance.
    
    ### **3️⃣ Scavenge Phase (Heap Management)**
    
    - Releases **unused memory** back to the **OS**.
    - Optimizes memory usage for better efficiency.
- **Tuning GC Performance in Go**
    
    Go’s GC is **self-optimizing**, but you can fine-tune it for performance.
    
    ### **1️⃣ Setting GC Target with GOGC**
    
    `GOGC` (Garbage Collection Target Percentage) controls **how often GC runs**.
    
    - **Higher value (e.g, 200)** → Runs less often, uses more memory.
    - **Lower value (e.g., 50)** → Runs more often, reduces memory footprint.
    
          **Example: Setting `GOGC`**

```bash
GOGC=100 go run main.go
```

Programmatically set: 

 ```golang
import "runtime"

func main() {
    runtime.GC() // Force GC (not recommended for frequent use)
    runtime.SetGCPercent(50) // More frequent GC
}
```
  
**NOTE:** For a GC to run and do its work, it needs CPU time but frequent GC makes the program run slow. But, if GC executes in long intervals, the memory in the heap might pile up leading to out-of-memory for newly coming memory allocations. So, GOGC maintains a trade-off between CPU and memory with GC frequency. It's very complex to define how GC works because there are numerous parameters to control the frequency of GC depending on the application's needs. Simply put, we can think GC will execute every 2 seconds (let's assume), and if in the meantime, when the heap memory is increased than the defined threshold at 1.5 seconds, then GC will trigger automatically.

**NOTE2:** Goroutines are not garbage collected while blocked or running, so leaked goroutines result in memory leaks even in a garbage-collected language. A leaked goroutine is a goroutine that is blocked or running indefinitely without any way to exit, causing a resource leak.

```golang
var ch chan int

go func() {
    <-ch // blocks forever
}()
```

