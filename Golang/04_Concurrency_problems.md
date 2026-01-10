# For a given n print 0102030405….n

- **Thread A:** calls `zero()` that should only output `0`'s.
- **Thread B:** calls `even()` that should only output even numbers.
- **Thread C:** calls `odd()` that should only output odd numbers.

Modify the given class to output the series `"010203040506..."` where the length of the series must be `2n`.

```
Input: n = 5
Output: "0102030405"
```

CODE: 

```go
package main

import (
	"fmt"
	"sync"
)

type ZeroEvenOdd struct {
	n          int
	currentNum int
	zero       chan bool
	even       chan bool
	odd        chan bool
	wg         sync.WaitGroup
}

func newZeroEvenOdd(n int) *ZeroEvenOdd {
	return &ZeroEvenOdd{
		n:          n,
		currentNum: 1,
		zero:       make(chan bool, 1),
		even:       make(chan bool, 1),
		odd:        make(chan bool, 1),
	}
}

func (z *ZeroEvenOdd) printZero() {
	defer z.wg.Done()

	for i := 0; i < z.n; i++ {
		<-z.zero
		fmt.Println(0)
		if z.currentNum%2 == 0 {
			z.even <- true
		} else {
			z.odd <- true
		}
	}
}

func (z *ZeroEvenOdd) printOdd() {
	defer z.wg.Done()

	for i := 1; i <= z.n; i += 2 {
		<-z.odd
		fmt.Println(i)
		z.currentNum++
		z.zero <- true
	}
}

func (z *ZeroEvenOdd) printEven() {
	defer z.wg.Done()

	for i := 2; i <= z.n; i += 2 {
		<-z.even
		fmt.Println(i)
		z.currentNum++
		z.zero <- true
	}
}

func main() {
	z := newZeroEvenOdd(10)

	z.zero <- true

	z.wg.Add(3)

	go z.printZero()
	go z.printOdd()
	go z.printEven()

	z.wg.Wait()
}
```

### **🔹 Explanation**

1. **`zeroChan` (buffered)** starts with a signal to **print 0 first**.
2. **Three goroutines**:
    - `zero()` → Prints `0` and signals either `odd()` or `even()`.
    - `odd()` → Prints the **next odd number** and signals `zero()`.
    - `even()` → Prints the **next even number** and signals `zero()`.
3. **Synchronization**:
    - **Channels (`zeroChan`, `oddChan`, `evenChan`)** control execution order.
    - **WaitGroup (`sync.WaitGroup`)** ensures all goroutines complete.

### **Why is returning a pointer necessary here?**

Returning a pointer (`*ZeroEvenOdd`) instead of a value (`ZeroEvenOdd`) is beneficial because:

1. **Avoids unnecessary copying** – If you return a struct by value, a copy of the entire struct is created every time it is returned.
2. **Allows shared state modification** – Since goroutines are modifying `currentNum` and using channels for synchronization, returning a pointer ensures all goroutines operate on the same instance of `ZeroEvenOdd`.
3. **Better memory efficiency** – Passing around pointers is more efficient than copying large structs, especially in concurrent programs.


-----

# For given n print [1 2 fizz 4 buzz fizz 7 8 fizz buzz] | leetcode: 1195

You are given an instance of the class `FizzBuzz` that has four functions: `fizz`, `buzz`, `fizzbuzz` and `number`. The same instance of `FizzBuzz` will be passed to four different threads:

- **Thread A:** calls `fizz()` that should output the word `"fizz"`.
- **Thread B:** calls `buzz()` that should output the word `"buzz"`.
- **Thread C:** calls `fizzbuzz()` that should output the word `"fizzbuzz"`.
- **Thread D:** calls `number()` that should only output the integers.

- `"fizzbuzz"` if `i` is divisible by `3` and `5`,
- `"fizz"` if `i` is divisible by `3` and not `5`,
- `"buzz"` if `i` is divisible by `5` and not `3`, or
- `i` if `i` is not divisible by `3` or `5`.

```
Input: n = 15
Output: [1,2,"fizz",4,"buzz","fizz",7,8,"fizz","buzz",11,"fizz",13,14,"fizzbuzz"]
```

```go
package main

import (
	"fmt"
	"sync"
)

type fizzBuzz struct {
	n        int
	fizz     chan bool
	buzz     chan bool
	fizzbuzz chan bool
	number   chan bool
	wg       sync.WaitGroup
	result   []string
}

func NewFizzBuzz(n int) *fizzBuzz {
	return &fizzBuzz{
		n:        n,
		fizz:     make(chan bool, 1), // Buffered to prevent deadlock
		buzz:     make(chan bool, 1),
		fizzbuzz: make(chan bool, 1),
		number:   make(chan bool, 1),
		result:   []string{},
	}
}

func (fb *fizzBuzz) AddFizz() {
	defer fb.wg.Done()

	for range fb.fizz { // Wait for a signal, exit when the channel closes
		fb.result = append(fb.result, "fizz")
		fb.number <- true // Continue processing
	}
}

func (fb *fizzBuzz) AddBuzz() {
	defer fb.wg.Done()

	for range fb.buzz {
		fb.result = append(fb.result, "buzz")
		fb.number <- true
	}
}

func (fb *fizzBuzz) AddFizzBuzz() {
	defer fb.wg.Done()

	for range fb.fizzbuzz {
		fb.result = append(fb.result, "fizzbuzz")
		fb.number <- true
	}
}

func (fb *fizzBuzz) AddNumber() {
	defer fb.wg.Done()

	for i := 1; i <= fb.n; i++ {
		<-fb.number // Wait for signal to proceed

		if i%3 == 0 && i%5 == 0 {
			fb.fizzbuzz <- true
		} else if i%3 == 0 {
			fb.fizz <- true
		} else if i%5 == 0 {
			fb.buzz <- true
		} else {
			fb.result = append(fb.result, fmt.Sprintf("%v", i))
			fb.number <- true // Continue processing
		}
	}

	// Close channels to prevent goroutines from waiting indefinitely
	// Other functions channels written in for range of the channel, it will be waiting forever
	close(fb.fizz)
	close(fb.buzz)
	close(fb.fizzbuzz)
}

func main() {
	fb := NewFizzBuzz(10)

	fb.wg.Add(4)

	fb.number <- true // Start processing

	go fb.AddNumber()
	go fb.AddBuzz()
	go fb.AddFizz()
	go fb.AddFizzBuzz()

	fb.wg.Wait() // Wait for all goroutines to finish

	fmt.Println(fb.result)
}
```

## **Fixing the Deadlock**

### ✅ **Fix 1: Close Channels at the Right Time**

- We must **close** the `fizz`, `buzz`, and `fizzbuzz` channels when `AddNumber` completes.
- This ensures waiting goroutines **exit gracefully** instead of hanging.

### ✅ **Fix 2: Use `select` for Safe Channel Handling**

- Instead of blindly reading from channels, we should use `select` to **avoid blocking indefinitely**.

### ✅ **Fix 3: Ensure `number` Passes Control Back Correctly**

- `number` should trigger the **next number's processing** to keep the loop running.


-----
# For a given n print foobarfoobar….n | leetcode: 1115

```
Input: n = 2
Output: "foobarfoobar"
Explanation: "foobar" is being output 2 times.
```

```go
package main

import (
	"fmt"
	"sync"
)

type foobar struct {
	n   int
	foo chan bool
	bar chan bool
	wg  sync.WaitGroup
}

func NewFooBar(n int) *foobar {
	return &foobar{
		n:   n,
		foo: make(chan bool, 1), // Buffered to prevent deadlock
		bar: make(chan bool, 1),
	}
}

func (fb *foobar) Foo() {
	defer fb.wg.Done()
	for i := 0; i < fb.n; i++ {
		<-fb.foo
		fmt.Printf("foo")
		fb.bar <- true
	}
}

func (fb *foobar) Bar() {
	defer fb.wg.Done()

	for i := 0; i < fb.n; i++ {
		<-fb.bar
		fmt.Printf("bar \n")
		fb.foo <- true
	}
	close(fb.foo)
	close(fb.bar)

}

func main() {
	fb := NewFooBar(5)

	fb.wg.Add(2)

	fb.foo <- true // Start processing

	go fb.Foo()
	go fb.Bar()

	fb.wg.Wait() // Wait for all goroutines to finish
}

```