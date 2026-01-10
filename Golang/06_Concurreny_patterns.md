# Worker-Pool Pattern

The **Worker Pool Pattern** in Golang is useful when you need to efficiently manage and distribute tasks among multiple goroutines while controlling concurrency. Here are some key use cases:

- User Cases
    
    ### **1. Processing Large Number of Tasks in Parallel**
    
    - When you have a large set of independent tasks (e.g., image processing, file parsing), a worker pool can distribute the load efficiently.
    
    **Example:**
    
    - Resizing thousands of images in parallel while limiting resource usage.
    
    ---
    
    ### **2. Rate Limiting API Calls**
    
    - If your application needs to make requests to an external API but has rate limits, a worker pool ensures that the requests do not exceed the limit.
    
    **Example:**
    
    - Sending 100,000 emails while ensuring only 100 concurrent requests are made.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Job represents the task to be executed by a worker
type Job struct {
	ID int
}

// WorkerPool represents a pool of worker goroutines
type WorkerPool struct {
	numWorkers int
	jobQueue   chan Job
	results    chan int
	wg         sync.WaitGroup
}

// NewWorkerPool creates a new worker pool with the specified number of workers
func NewWorkerPool(numWorkers, jobQueueSize int) *WorkerPool {
	return &WorkerPool{
		numWorkers: numWorkers,
		jobQueue:   make(chan Job, jobQueueSize),
		results:    make(chan int, jobQueueSize),
	}
}

// worker function to process jobs from the queue
func (wp *WorkerPool) worker(id int) {
	defer wp.wg.Done()
	for job := range wp.jobQueue {
		// Do the actual work here
		fmt.Printf("Worker %d started job %d\n", id, job.ID)
		time.Sleep(time.Second) // Simulating work
		fmt.Printf("Worker %d finished job %d\n", id, job.ID)
		wp.results <- job.ID
	}
}

// Start starts the worker pool and dispatches jobs to workers
func (wp *WorkerPool) Start() {
	for i := 1; i <= wp.numWorkers; i++ {
		wp.wg.Add(1)
		go wp.worker(i)
	}
}

// Wait waits for all workers to finish and closes the results channel
func (wp *WorkerPool) Wait() {
	wp.wg.Wait()
	close(wp.results)
}

// AddJob adds a job to the job queue
func (wp *WorkerPool) AddJob(job Job) {
	wp.jobQueue <- job
}
Next

// CollectResults collects and prints results from the results channel
func (wp *WorkerPool) CollectResults() {
	for result := range wp.results {
		fmt.Printf("Result received for job %d\n", result)
	}
}

func main() {
	numWorkers := 3
	numJobs := 10

	workerPool := NewWorkerPool(numWorkers, numJobs)

	// Adding jobs to the Job Queue
	for i := 1; i <= numJobs; i++ {
		workerPool.AddJob(Job{ID: i})
	}

	close(workerPool.jobQueue)

	workerPool.Start()
	workerPool.Wait()
	workerPool.CollectResults()
}
```


-----
# Generator + Fan-out/Fan-in + Pipe-Line pattern

```go
package main

import (
	"fmt"
	"math/rand"
	"runtime"
	"sync"
)

func repeatFunc[T any](fn func() T) <-chan T {

	stream := make(chan T)
	go func() {
		defer close(stream)
		for {
			stream <- fn()
		}
	}()
	return stream
}

func primeFinder(randIntSteamChan <-chan int) <-chan int {
	isPrime := func(num int) bool {
		for i := num - 1; i > 1; i-- {
			if num%i == 0 {
				return false
			}
		}
		return true
	}

	primes := make(chan int)
	go func() {
		for {
			randomNum := <-randIntSteamChan
			if isPrime(randomNum) {
				primes <- randomNum
			}
		}
	}()

	return primes
}

func fanIn[T any](channels ...<-chan T) <-chan T {
	var wg sync.WaitGroup
	fannedInStream := make(chan T)

	transfer := func(c <-chan T) {
		defer wg.Done()
		for i := range c {
			fannedInStream <- i
		}
	}

	for _, channel := range channels {
		wg.Add(1)
		go transfer(channel)
	}

	go func() {
		wg.Wait()
		close(fannedInStream)
	}()

	return fannedInStream
}

func take[T any](stream <-chan T, n int) <-chan T {
	taken := make(chan T)
	go func() {
		defer close(taken)
		for i := 0; i < n; i++ {
			select {
			case taken <- <-stream:
			}
		}
	}()

	return taken
}

func main() {

	randNumFetcher := func() int {
		return rand.Intn(5000000)
	}

	// repeatFunc returns steam channel
	randIntSteamChan := repeatFunc[int](randNumFetcher)

	// Fan-Out
	CPUCount := runtime.NumCPU()
	primeFinderchannel := make([]<-chan int, CPUCount)
	for i := 0; i < CPUCount; i++ { 
		// PrimeFinder returns the prime channel
		primeFinderchannel[i] = primeFinder(randIntSteamChan)
	}

	fanInedSteam := fanIn[int](primeFinderchannel...)

	// Fan-In
	for num := range take(fanInedSteam, 13) {
		fmt.Println(num)
	}

}

```