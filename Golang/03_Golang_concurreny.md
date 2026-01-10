
| No | Topic | YouTube Reference |
|----|------|------------------|
| 1 | [Gorountines, Go-runtime, Go-schedulers](#gorounties-go-runtime-go-runtime-scheduler) | |
| 2 | [Depth understanding M:N scheduler](#depth-understanding-m-n-scheduler) | https://www.youtube.com/watch?v=S-MaTH8WpOM |
| 3 | [Sync package Waitgroup](#sync-package-waitgroup) | |
| 4 | [Sync package: Mutex and RWMutex](#sync-package-mutex-and-rwmutex) | |
| 5 | [Sync package: Once](#sync-package-once) | |
| 6 | [Select Statements](#select-statement) | |
| 7 | [Context package](#context-package) | https://www.youtube.com/watch?v=0x_oUlxzw5A https://www.youtube.com/watch?v=8omcakb31xQ |


--------
# Gorounties, Go-runtime, Go-runtime scheduler

### Gorountine: Lightweight threads

Other languages users OS threads directly, but the lightweight goroutines leverages the OS threads.

### Facts about goroutines:

- Started and managed by **go-runtime**
- Go-routines start with just  ~2kb of memory but have **growable stacks.**
- Seamless **context-switiching** that maximize the efficiency of using **go-runtime scheduler.**
- Inter go-routine communication with **channels.**

### Go-routine stack

- All go-rountines are initially assigned with 2kb of memory.
- Each go function has a small preamble, which calls command **morestack if it runs out of memory**
- Then, runtime allocates a new memory segment with double the size of the old segment and restart the execution.
- Effectively, makes go-routines infinitely growable with efficient shrinking.

Go uses an **M:N scheduling model**, meaning **M goroutines run on N OS threads**. This is a key reason why Go can efficiently handle thousands (or even millions) of goroutines without excessive system overhead.

## **📌 What is the M:N Scheduling Model?**

- **M = Number of Goroutines** (User-space tasks)
- **N = Number of OS Threads** (Kernel-managed execution units)
- Instead of creating a 1:1 mapping (like Java or C++ threads), Go’s **runtime scheduler dynamically maps multiple goroutines onto a small number of OS threads**.

### **🔹 Why This Matters?**

✅ **High Concurrency** → Run many tasks without creating thousands of OS threads.

✅ **Lower Memory Usage** → Go-routines use **~2 KB stack**, while OS threads need **~1 MB**.

✅ **Efficient Scheduling** → Go **does not rely on the OS scheduler**, making context switches cheaper.

-------

# Depth understanding M:N scheduler

Go-routine has three states
![alt text](images/image.png)

Examples of what happens on scheduler when we start or running the go-routine.

In a basic of main fuction, the main function runs on main thread.

![alt text](images/image-1.png)
We can create the processors in go environments

![alt text](images/image-2.png)

Now, let’s see what if main function spawns a Goroutine G1, this G1 wakes up the one of the idle processors

![alt text](images/image-3.png)

Now, the new processor creates an M (os thread)

![alt text](images/image-4.png)

![alt text](images/image-5.png)

What if G1 completed it’s execution, the P3 and M becomes idle

![alt text](images/image-6.png)

![alt text](images/image-7.png)

From these we might be wondering what’s the need for P as we are just mapping OS thread(M) to go routines

Let’s say there are two processor and two OS thread which are happily executing the G0 and G1. Suddenly, what will happen if main() spawns new go routine (G2)

![alt text](images/image-8.png)

We need to find place to put G2 
![alt text](images/image-9.png)

helps to identify what go-routine have yet to be run in their LOCAL RUN QUEUE

If G1 finishes it’s executing, P1 looks in LRQ  and schedules the go routine onto the OS thread and begins executing. (**P knows what G’s are available for an M to run)**

![alt text](images/image-10.png)

### **WHAT ABOUT BLOCKED GOROUTINE?**

The scheduler should maximize execution time on OS threads by moving off the blocked Go-routine from OS threads and brings them once they are ready.

![alt text](images/image-11.png)

There is one go-routine G0 which is on blocked state. (System calls blocking)
![alt text](images/image-12.png)
![alt text](images/image-13.png)

Now, P0 needs some OS threads to continue executing other go-routines, so it goes ahead and wake’s up the OS thread(M) from idle threads.

![alt text](images/image-14.png)

If G0 becomes available, then M0 searches for P, so it wakes up the P1 from idle processors
![alt text](images/image-15.png)
![alt text](images/image-16.png)

### What will happens if M0 can’t find any processors??

![alt text](images/image-17.png)
![alt text](images/image-18.png)

The go-rountine will be added to GLOBAL RUN QUEUE, because, other can take up the go-routine from GRQ if doesn’t have any other go-routine to run

![alt text](images/image-19.png)

![alt text](images/image-20.png)

## BLOCKING ON CHANNELS

![alt text](images/image-21.png)

Inside the channel struct we have receive queue(recvq) which consists of goroutines that are blocking and waiting on a channel received to happen.
![alt text](images/image-22.png)
![alt text](images/image-23.png)
![alt text](images/image-24.png)
![alt text](images/image-25.png)
![alt text](images/image-26.png)

## WORK STEALING BETWEEN THREADS
![alt text](images/image-27.png)
One of threads finished executing all of it’s goroutines faster than other.

![alt text](images/image-28.png)

The P0 goes and look into P1’s LRQ and steals half of the goroutines from there and start exectuing.

![alt text](images/image-29.png)
![alt text](images/image-30.png)
![alt text](images/image-31.png)


--------
# Sync package: Waitgroup

Interview Prep: sync.WaitGroup
A sync.WaitGroup is a mechanism used to synchronize goroutines by waiting for a set of concurrent operations to complete. It is effectively a concurrent-safe counter that tracks how many operations are still running.

--------------------------------------------------------------------------------
### Core Methods
The WaitGroup operates using three primary methods:
* **Add(int):** Increments the internal counter by the integer passed to it. This indicates that a specific number of goroutines are beginning.
* **Done():** Decrements the counter by exactly one. It is customary to call this using the defer keyword to ensure the counter is decremented even if the goroutine exits unexpectedly or at the very end of its closure.
* **Wait():** Blocks the calling goroutine (typically the main goroutine) until the internal counter reaches zero.

--------------------------------------------------------------------------------
### When to Use WaitGroup
* **Use it when:** You need to wait for a set of concurrent operations to finish, but you do not care about the results of those operations, or you have alternative ways to collect their results.
* **Avoid it when:** You need to pass results back from goroutines or handle complex coordination. In those cases, channels and select statements are preferred.

--------------------------------------------------------------------------------
### Critical Implementation Rules
* **Placement of Add:** You must call Add outside the goroutine it is intended to track. If Add is placed inside the goroutine's closure, a race condition is introduced. Because there is no guarantee when a goroutine will be scheduled, the Wait call could be reached and return before any goroutines have even started or incremented the counter.
* **Batching Add:** While you can call Add(1) immediately before starting each goroutine, it is also common to call Add once with the total number of expected goroutines (e.g., wg.Add(numGreeters)) right before entering a loop that spawns them.
* **Passing to Functions:** When passing a WaitGroup into a function or closure, it should be passed as a pointer (e.g., *sync.WaitGroup) to ensure the same counter is being updated across different scopes.

--------------------------------------------------------------------------------
### Common Pitfalls for Interview Questions
* **The Race Condition:** If an interviewer asks why Add isn't inside the go func(), the answer is that Wait might see a counter of zero and finish before the goroutine even starts.
* **Leaking Goroutines:** Forgetting to call Done() will cause Wait() to block forever, leading to a deadlock. This is why defer wg.Done() is considered a best practice.


### Example

```golang
func main(){
    hello := func(wg *sync.WaitGroup, id int) {
        defer wg.Done()
        fmt.Printf("Hello from %v!\n", id)
    }
    const numGreeters = 5
    var wg sync.WaitGroup
    wg.Add(numGreeters)
    for i := 0; i < numGreeters; i++ {
        go hello(&wg, i+1)
    }
    wg.Wait()
}
```
--------
# Sync package: Mutex and RWMutex

In Go, synchronization of memory access is primarily handled through sync.Mutex and sync.RWMutex. These tools ensure that shared resources are accessed by only one goroutine at a time (or a specific set of readers), preventing race conditions and data corruption.

--------------------------------------------------------------------------------
### sync.Mutex (Mutual Exclusion)
A Mutex provides a concurrent-safe way to express exclusive access to a shared resource. It is used to guard critical sections, which are areas of a program that require exclusive access to memory.
* **Core Logic:** Unlike channels, which share memory by communicating, a Mutex shares memory by establishing a convention that developers must follow to synchronize access.
* **Methods:**
    ◦ Lock(): Requests exclusive use of the guarded critical section. If the lock is already in use, the calling goroutine will block until it is available.
    ◦ Unlock(): Signals that the goroutine is finished with the critical section, allowing other goroutines to acquire the lock.
* **Best Practice – defer:** It is a standard idiom to call Unlock() within a defer statement. This ensures the lock is released even if the function panics. Failing to release a lock will likely cause the program to deadlock.

```golang
func main(){
    var count int
	var lock sync.Mutex

	increment := func() {
		lock.Lock()
		defer lock.Unlock()
		count++
		fmt.Println("Count:", count)
	}

	decrement := func() {
		lock.Lock()
		defer lock.Unlock()
		count--
		fmt.Println("Count:", count)
	}

	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		wg.Add(1)

		go func() {
			defer wg.Done()
			increment()
		}()
	}

	for i := 0; i < 5; i++ {
		wg.Add(1)

		go func() {
			defer wg.Done()
			decrement()
		}()
	}

	wg.Wait()
}
```

--------------------------------------------------------------------------------
### sync.RWMutex (Reader-Writer Mutex)
The sync.RWMutex is conceptually similar to a Mutex but offers finer-grained control over memory access. It is specifically designed for scenarios where many processes need to read memory, but only a few need to write to it.
* Reader Locks: An arbitrary number of readers can hold a reader lock simultaneously, provided no one is holding a writer lock.
* Writer Locks: If a goroutine holds a writer lock, no other readers or writers can access the critical section.
* Efficiency: This is highly effective for performance when you have a read-heavy workload, as it reduces the bottleneck of waiting for a standard Mutex.

```golang
type ConfigStore struct {
    mu   sync.RWMutex
    data map[string]string
}


func (c *ConfigStore) Update(key, value string) {
    c.mu.Lock()          // exclusive lock
    defer c.mu.Unlock()

    time.Sleep(50 * time.Millisecond) // simulate expensive write
    c.data[key] = value
}


func (c *ConfigStore) Get(key string) string {
    c.mu.RLock()         // shared lock
    defer c.mu.RUnlock()

    time.Sleep(10 * time.Millisecond) // simulate read work
    return c.data[key]
}


func main() {
    store := &ConfigStore{
        data: map[string]string{
            "host": "localhost",
        },
    }

    var wg sync.WaitGroup

    // Writer goroutine
    wg.Add(1)
    go func() {
        defer wg.Done()
        for i := 0; i < 1000; i++ {
            store.Update("host", fmt.Sprintf("host-%d", i))
        }
    }()

    // 8 concurrent readers
    for i := 0; i < 8; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            val := store.Get("host")
            fmt.Println("Reader", id, "read:", val)
        }(i)
    }

    wg.Wait()
}

output: (could be different also)

Host: localhost reader: 7
Host: host-107 reader: 3
Host: host-108 reader: 4
Host: host-109 reader: 6
Host: host-109 reader: 1
Host: host-109 reader: 5
Host: host-109 reader: 0
Host: host-110 reader: 2


```

--------------------------------------------------------------------------------
### Critical Sections and Bottlenecks
* Bottlenecks: Critical sections are inherently bottlenecks because they restrict concurrent execution.
* Optimization: Because entering and exiting these sections is somewhat expensive, developers should aim to minimize the time spent within them.
* Strategy: One way to improve performance is to reduce the "cross-section" of the critical section—only locking the specific lines of code that absolutely require synchronized access.

--------------------------------------------------------------------------------
### The sync.Locker Interface
Both Mutex and RWMutex satisfy the sync.Locker interface. This interface requires two methods:
1. Lock()
2. Unlock()

By using the sync.Locker type in function signatures, you can write code that is flexible enough to accept either a standard Mutex or an RWMutex.

--------------------------------------------------------------------------------
Summary Table for Interviews

| Feature | sync.Mutex | sync.RWMutex |
|---------|------------|--------------|
| Primary Use | Exclusive access to data. | Read-heavy workloads. |
| Readers | Only one at a time. | Multiple simultaneous readers. | 
| Writers | Only one at a time. | Only one at a time; blocks readers. |
| Key Risk | Deadlock if not unlocked. | Deadlock if not unlocked. | 

--------------------------------------------------------------------------------
Analogy for Understanding: Think of a sync.Mutex like a single-person dressing room. Only one person can enter at a time, regardless of whether they are just checking their outfit (reading) or changing it (writing).
A sync.RWMutex is more like a museum gallery. If people just want to look at the art (readers), hundreds can be inside at once. However, if a maintenance worker needs to move a painting (writer), everyone else must leave the room until the worker is finished.


-----

# Sync package: Once
sync.Once is a synchronization primitive used to ensure that a specific piece of code is executed exactly once, regardless of how many times it is called or how many goroutines are attempting to call it. It is a powerful tool for implementing singleton patterns or performing one-time initializations.

### Core Functionality: The Do Method
* **Execution Guarantee:** The primary method is Do(f func()). When called, sync.Once ensures that the function f is executed only once, even if Do is called hundreds of times across different goroutines.
* **Internal Logic:** sync.Once utilizes internal synchronization primitives to track whether a call has already been completed.
* **Prevalence:** This is not a niche tool; it is used frequently within the Go standard library (approximately 70 times) to handle various initialization tasks.

```golang
func main() {
    var count int
    increment := func() {
        count++
    }
    once sync.Once
    var increments sync.WaitGroup
    increments.Add(100)
    for i := 0; i < 100; i++ {
        go func() {
            defer increments.Done()
            once.Do(increment)
        }()
    }
    increments.Wait()
    fmt.Printf("Count is %d\n", count)
}
```

### Critical behavior:

* **Call Counting vs. Function Counting:** A common misconception is that sync.Once tracks the specific function passed to it. In reality, sync.Once only counts the number of times the Do method itself is called.
* Example: If you have one instance of sync.Once and you call once.Do(increment) followed by once.Do(decrement), only the first function (increment) will run. The decrement function will be ignored because the Once instance has already recorded a successful call to its Do method.
```golang
increment := func() { count++ }
decrement := func() { count-- }
var once sync.Once
once.Do(increment)
once.Do(decrement)
fmt.Printf("Count: %d\n", count)

o/p: 1
```

### Deadlock scenarios with once

consider this example

```golang
func main() {
    var onceA, onceB sync.Once
    var initB func()

    // (2)
    initA := func() { 
        onceB.Do(initB)
    }

    initB = func() {
        onceA.Do(initA)
    }

    // first executed (1)
    onceA.Do(initA)
}
```
If you look at this example, 

`onceA.Do(initA)`

- onceA has not run yet
- initA starts executing
- onceA is now locked internally
- initA is “in progress”

`onceB.Do(initB)`
- onceB has not run yet
- initB starts executing
- onceB is now locked internally
- initB is “in progress”

`onceA.Do(initA)`
- onceA is already executing
- sync.Once behavior:
    - If another goroutine calls Do() → it waits
    - If the same goroutine calls Do() → it also waits

👉 onceA.Do(initA) blocks until the first initA completes

**`Circular wait (DEADLOCK)`**
```
initA → waits for initB
initB → waits for initA
```
This program deadlocks forever


-----

# Types of Channels

1️⃣ **Buffered Channels** (Non-blocking)

2️⃣ **Un-buffered Channels** (Blocking)

3️⃣ **Directional Channels** (Send-only / Receive-only)

4️⃣ **Closing Channels** (To signal completion)

### Buffered channels (Non-blocking)

- A buffered channel has a defined capacity to store values.
- When a sender sends a value on a buffered channel, it blocks only if the channel is full.
- When a receiver tries to receive from a buffered channel, it blocks only if the channel is empty.
- Buffered channels are useful when you want to decouple the timing of sender and receiver, allowing them to operate more independently.
- They can provide a degree of asynchrony, as the sender can continue sending even if the receiver is not immediately ready

```go
ch := make(chan int, 3) // Buffered channel with capacity 3
ch <- 1  
ch <- 2  
ch <- 3  // No blocking until full

fmt.Println(<-ch) // Prints 1
fmt.Println(<-ch) // Prints 2
fmt.Println(<-ch) // Prints 3
```

### Un-buffered channels (Blocking)

- An un-buffered channel doesn't have any capacity to store values.
- When a sender sends a value on an un-buffered channel, it blocks until a receiver receives that value.
- Un-buffered channels are typically used for synchronized communication between go-routines.
- They enforce a direct exchange of data between sender and receiver, ensuring that the sender and receiver are both ready before the value is exchanged.

```go
ch := make(chan int) // Unbuffered channel

go func() {
    ch <- 42 // Blocks until received
}()

fmt.Println(<-ch) // Receives 42
```

### **Directional Channels** (Restrict Send/Receive)

You can specify **send-only** or **receive-only** channels in function parameters.

```go
func sendData(ch chan<- int) { // Send-only
	ch <- 99
}

func receiveData(ch <-chan int) { // Receive-only
	fmt.Println(<-ch)
}

func main() {
	ch := make(chan int)
	go sendData(ch)
	receiveData(ch)
}
```

- If a go-routine writes to a full channel and **no other go-routine is reading**, it **deadlocks**.
- If a go-routine reads from an empty channel and **no other go-routine is writing**, it **deadlocks**.

| Operation | Channel State            | Result                                                                 |
|----------|--------------------------|------------------------------------------------------------------------|
| Read     | nil                      | Block                                                                  |
|      | Open and Not Empty       | Value                                                                  |
|      | Open and Empty           | Block                                                                  |
|      | Closed                   | `<default value>, false`                                               |
|      | Write Only               | Compilation Error                                                      |
|          |                          |                                                                        |
| Write    | nil                      | Block                                                                  |
|     | Open and Full            | Block                                                                  |
|     | Open and Not Full        | Write Value                                                            |
|     | Closed                   | **panic**                                                              |
|     | Receive Only             | Compilation Error                                                      |
|          |                          |                                                                        |
| Close    | nil                      | **panic**                                                              |
|     | Open and Not Empty       | Closes channel; reads succeed until channel is drained, then default |
|     | Open and Empty           | Closes channel; reads produce default value                            |
|     | Closed                   | **panic**                                                              |
|     | Receive Only             | Compilation Error                                                      |


-----

# Select Statement

### Core Concept: The "Glue" of Concurrency
The select statement is considered the "lingua franca" of Go concurrency. While channels bind goroutines together, the select statement binds channels together, allowing you to compose them into larger abstractions and coordinate complex system components.
### Key Mechanics & Syntax
* **Simultaneous Evaluation:** Unlike a standard switch block which tests cases sequentially, a select block considers all channel reads and writes simultaneously.
* **Blocking Behavior:** If none of the channels in the case statements are ready (e.g., no data to read, no space to write), the entire select statement blocks until one becomes ready.
* **Execution:** Once a channel is ready, that specific operation proceeds, and its corresponding code block executes.

```golang
var c1, c2 <-chan interface{}
var c3 chan<- interface{}

select {
case <- c1:
    // Executes if c1 has data or is closed
case <- c2:
    // Executes if c2 has data or is closed
case c3 <- struct{}{}:
    // Executes if c3 has capacity for a write
}
```

### Handling multiple ready channels
A common interview question is: "What happens if multiple channels are ready at the same time?"
* **Pseudo-random Selection:** The Go runtime performs a pseudo-random uniform selection. Each ready case has an equal chance of being selected.
* **Design Philosophy:** This prevents "starvation" and ensures Go programs perform well in the "average case," as the runtime cannot know the programmer's specific intent for the channels.

Uniform Selection Example:
```golang
func main(){
    c1 := make(chan interface{}); close(c1)
    c2 := make(chan interface{}); close(c2)

    var c1Count, c2Count int
    for i := 1000; i >= 0; i-- {
        select {
        case <-c1:
            c1Count++
        case <-c2:
            c2Count++
        }
    }
}

o/p: 
c1Count: 505
c2Count: 496

// As you can see, in a thousand iterations, roughly half the time the select statement
// read from c1, and roughly half the time it read from c2
```


### Non-Blocking Operations and the default Clause
If you want to exit a select block immediately when no channels are ready, you use a default clause.
* **Immediate Execution:** The default case runs almost instantaneously if all other cases are blocking.
* **The for-select Pattern:** This is frequently used to allow a goroutine to perform work in a loop while occasionally checking for a signal (like a cancellation or completion signal)

```golang
func main(){
    done := make(chan interface{})
    go func() {
        time.Sleep(5 * time.Second)
        close(done)
    }()

    workCounter := 0
    loop:
    for {
        select {
        case <-done:
            break loop // Stop when signaled
        default:
        }
        // Simulate work
        workCounter++
        time.Sleep(1 * time.Second)
    }
}
```

### Implement timeouts

Because a select block can block indefinitely, it is often paired with time.After to implement timeouts. time.After returns a channel that sends the current time after a specified duration, effectively "racing" against your other operations

```golang
func main(){
    var c <-chan int // A nil channel that will always block
    select {
    case <-c:
        // This will never happen
    case <-time.After(1 * time.Second):
        fmt.Println("Timed out.")
    }
}
```

#### **NOTE:**
* Nil Channels: Reading from or writing to a nil channel in a select statement will block forever for that specific case.
* Empty Select: The statement select {} (a select with no cases) will block the goroutine forever

-------

# Prevention of go-routine leaks

While goroutines are lightweight and easy to create, they are not garbage collected by the Go runtime. If a goroutine is started and cannot terminate, it will remain in memory for the lifetime of the process, leading to memory "creep".
Termination Paths: A goroutine naturally terminates when it:
* Completes its work.
* Encounters an unrecoverable error.
* Is explicitly told to stop working.

### Leak scenario: Blocking on a receive
A leak often occurs when a goroutine is waiting to receive data from a channel that will never be populated.
```golang
func main() {
    doWork := func(strings <-chan string) <-chan interface{} {
        completed := make(chan interface{})
        go func() {
            defer fmt.Println("doWork exited.")
            defer close(completed)
            for s := range strings {
                fmt.Println(s)
            }
        }()
        return completed
    }

    // Passing nil means the 'strings' channel never receives data.
    // The goroutine inside doWork blocks forever.
    doWork(nil) 
    fmt.Println("Done.")
}

// strings channel nil, hence, the go rountine will be blocked on for loop expecting a value from the channel (leak scenario)
```

### Mitigation: done channel pattern
The standard way to prevent leaks is to establish a signal between a parent goroutine and its children. By convention, this is a read-only channel usually named done. The parent closes this channel when it wants the child to terminate.

```golang
func main(){
    doWork := func(done <-chan interface{}, strings <-chan string) <-chan interface{} {
        terminated := make(chan interface{})
        go func() {
            defer fmt.Println("doWork exited.")
            defer close(terminated)
            for {
                select {
                case s := <-strings:
                    fmt.Println(s)
                case <-done: // Signal received to stop
                    return
                }
            }
        }()
        return terminated
    }

    done := make(chan interface{})
    terminated := doWork(done, nil)

    go func() {
        time.Sleep(1 * time.Second)
        close(done) // Closing signals the child to exit
    }()

    <-terminated // Joining the goroutine
    fmt.Println("Done!")
}

o/p:
Canceling doWork goroutine...
doWork exited.
Done.

// now, we passed a done, and fired a seperate go-routine which closes the done after 1 second, 
// hence, for select in dowork function, will be expecting some value from strings channel (which is nil), so it checks done, after 1 seconds done will be closed, so for select reads default value from done returns the from the funtion

// after return is done, defer will be executed and close the terminated channel as well

// from the main fucnition, default value will be read from terminated channel and prints Done
```


### Leak scenario: Blocking on send

Leaks can also occur in "producer" goroutines that are blocked while trying to write a value to a channel that no longer has any readers.

```golang
func main(){
    newRandStream := func() <-chan int {
        randStream := make(chan int)
        go func() {
            defer fmt.Println("newRandStream closure exited.")
            defer close(randStream)
            for {
                randStream <- rand.Int()
            }
        }()
        return randStream
    }
    randStream := newRandStream()
    fmt.Println("3 random ints:")
    for i := 1; i <= 3; i++ {
        fmt.Printf("%d: %d\n", i, <-randStream)
    }
}

o/p:

3 random ints:
1: 5577006791947779410
2: 8674665223082153551
3: 6129484611666145821

// You can see from the output that the deferred fmt.Println statement never gets run.
// After the third iteration of our loop, our goroutine blocks trying to send the next random integer to a channel that is no longer being read from. 
// We have no way of telling the producer it can stop.
```

### Mitigation: 

```golang
func main(){
    newRandStream := func(done <-chan interface{}) <-chan int {
    randStream := make(chan int)
        go func() {
            defer fmt.Println("newRandStream closure exited.")
            defer close(randStream)
            for {
                select {
                case randStream <- rand.Int(): // Proceed if channel has a reader
                case <-done: // Stop if signaled
                    return
                }
            }
        }()
        return randStream
    }
    done := make(chan interface{})
    randStream := newRandStream(done)
    fmt.Println("3 random ints:")
    for i := 1; i <= 3; i++ {
        fmt.Printf("%d: %d\n", i, <-randStream)
    }
    close(done)
    // Simulate ongoing work
    time.Sleep(1 * time.Second)
}

o/p:
3 random ints:
1: 5577006791947779410
2: 8674665223082153551
3: 6129484611666145821
newRandStream closure exited.
```

```text
Now that we know how to ensure goroutines don’t leak, we can stipulate a convention: 
If a goroutine is responsible for creating a goroutine, 
it is also responsible for ensuring it can stop the goroutine.
```


------

# Context Package:
## youtube links (must watch)
* https://www.youtube.com/watch?v=0x_oUlxzw5A
* https://www.youtube.com/watch?v=8omcakb31xQ

NOTES YET TO BE ADDED