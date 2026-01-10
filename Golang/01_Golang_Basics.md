# Questions

| No | Topic |
|----|------|
| 1 | [Golang Lifecycle](#golang-lifecycle) |
| 2 | [What is the purpose of the init() function?](#what-is-the-purpose-of-the-init-function) |
| 3 | [Exported vs UnExported identifies in Golang](#exported-vs-unexported-identifies-in-golang) |
| 4 | [Go Workspace, go.mod, go.sum](#go-workspace-gomod-gosum) |
| 5 | [Compilation vs Interpretation](#compilation-vs-interpretation) |
| 6 | [Cross Compilation](#cross-compilation) |
| 7 | [Zero Values in Go](#zero-values-in-go) |
| 8 | [int vs int32 vs int64](#int-vs-int32-vs-int64) |
| 9 | [byte vs rune](#byte-vs-rune) |
| 10 | [Embedded structs](#embedded-structs) |
| 11 | [var vs := vs const](#explain-the-differences-between-var--and-const-in-go) |
| 12 | [new() vs make()](#explain-the-difference-between-new-and-make) |
| 13 | [interface{} vs any](#what-is-the-difference-between-interface-and-any-in-go) |
| 14 | [Slices internal implementation](#what-are-slices-and-how-are-they-implemented-internally) |
| 15 | [* and & operators](#Combination-example-with-*and&) |
| 16 | [Pointer to Slice / Map Behavior](#pointer-to-slice--map-behavior-in-go) |




# Golang LifeCycle
```
.go files
   ↓
Compilation & Linking
   ↓
Binary Execution
   ↓
Runtime Init
   ↓
Package init()
   ↓
main()
   ↓
Goroutines + GC + Scheduler
   ↓
main returns / os.Exit
   ↓
Process exits
```

### go build / go run**

* Go toolchain parses all .go files
* Packages are resolved using go.mod
* Dependency versions verified via go.sum

### Package Initialization Phase
**Package Initialization Order**
* Imported packages (depth-first)
* Current package

**Initialization Includes**

Global variable initialization

init() functions execution

Rules:
* Multiple init() allowed per package
* init():
    * Cannot take arguments
    * Cannot return values
    * Runs before main()
    * Execution order:
        * Variables → init() → next package
  
----

# What is the purpose of the init() function?
The init() function in Go is a special function used for initialization before the main execution of a program. It is automatically executed before the main() function and is typically used for setting up configurations, initializing variables, or performing setup tasks.

- Key Characteristics of `init()`:
    1. **No Parameters & No Return Values**
        - The `init()` function cannot take arguments or return values.
    2. **Automatically Called**
        - Go **automatically** calls `init()` before `main()`, so you don't invoke it explicitly.
    3. **Multiple `init()` Functions**
        - A package can have **multiple** `init()` functions, even in different files.
        - Execution order follows the order in which Go initializes the package dependencies.
    4. **Runs Once Per Package**
        - Each package can have an `init()`, which executes when the package is imported.

```golang
package main

import "fmt"

func init() {
   fmt.Println("Init function 1")
}

func init() {
   fmt.Println("Init function 2")
}

func main() {
   fmt.Println("Main function")
}
```

```
o/p: 

Init function 1
Init function 2
Main function
```


-----

# Exported vs UnExported identifies in Golang

**If an identifier starts with a capital letter -> Exported**

**If an identifier starts with a lowercase letter -> Unexported**

This rule applies to everything:

- Variables
- Constants
- Functions
- Methods
- Struct types
- Struct fields
- Interfaces

### Functions
```golang
func StartServer() {}  // exported
func startServer() {}  // unexported
```

### Structs & Fields
```golang
type User struct {
    ID   int    // exported field
    name string // unexported field
}
```

### Variables
```golang
var Count int      // exported
var count int      // unexported
```

### Visibility
**Exported:**
Accessible from any package that imports the package where it is declared.

**Unexported:** Only accessible within the package where it is defined (package-private).

### Usage
**Exported:** Form the public API of a package, defining what other parts of a program can interact with. 

**Unexported:** Form the public API of a package, defining what other parts of a program can interact with.


----

# Go Workspace, go.mod, go.sum

### What is Go Workspace?
A Go workspace is the environment where Go source code, dependencies, and binaries are managed. With Go modules, code can live anywhere on the filesystem and is no longer restricted to the GOPATH directory.

### What is GOPATH and why is it mostly obsolete?
GOPATH was the old workspace model where all Go code had to live inside a fixed directory structure. It is mostly obsolete because Go modules allow versioned dependencies and flexible project locations.

### What is go.mod?
go.mod is the module definition file that declares the module path, Go version, and the direct dependencies required by the project.

### What information does go.mod contain?
It contains the module name, Go version, required direct dependencies, and optional directives like replace and exclude.

### What is a Go module?
A Go module is a collection of Go packages versioned together and defined by a go.mod file at its root.

### What is go.sum?
go.sum is a file that stores cryptographic checksums of module dependencies to ensure dependency integrity and reproducible builds.

### Why is go.sum needed when go.mod already exists? (`!important`)
go.mod defines which dependencies to use, while go.sum ensures the downloaded dependency content has not changed and is secure.

The go.sum file is a security measure. It stores cryptographic hashes (checksums) of the exact content of all direct and indirect dependencies. 

### Should go.sum be committed to version control?
Yes, go.sum should be committed to ensure consistent and secure dependency resolution across all environments.

### What happens if go.sum is deleted?
Go will re-download dependencies and regenerate go.sum, but builds may fail if dependency content has changed.

### What is the replace directive in go.mod?
replace allows overriding a dependency with another version or a local path, commonly used for local development or testing fixes.

### What is the require directive in go.mod?
require lists the direct dependencies that the module depends on, along with their versions.

### Does go.mod list transitive dependencies?
No, go.mod lists only direct dependencies; transitive dependencies are resolved automatically by the Go toolchain.

### Can multiple projects use different versions of the same dependency?
Yes, Go modules allow each project to use its own dependency versions independently.

### What command initializes a Go module?
go mod init

### How are dependencies downloaded in Go modules?
Dependencies are downloaded into the module cache, usually located in $GOPATH/pkg/mod.


----

# Compilation vs Interpretation

Compiled languages translate the entire source code into machine code before execution and produce a standalone binary, 

whereas interpreted languages execute source code line by line at runtime without generating a binary; 

`Go is a compiled language that produces a native executable before running.`

---

# Cross Compilation

Cross compilation is the process of building a binary on one operating system or CPU architecture that is intended to run on a different operating system or architecture, which Go supports natively using environment variables like GOOS and GOARCH.

To cross-compile a Go program, set the GOOS and GOARCH environment variables to your desired target platform before running go build. 

```
# Compile for Windows (amd64)
GOOS=windows GOARCH=amd64 go build -o myapp-windows.exe

# Compile for Linux (arm64, e.g., Raspberry Pi 4 on 64-bit OS)
GOOS=linux GOARCH=arm64 go build -o myapp-linux-arm64

# Compile for macOS (Apple Silicon M1/M2/M3)
GOOS=darwin GOARCH=arm64 go build -o myapp-mac-arm64
```

----

# Zero Values in Go

Zero values are the default values assigned to variables in Go when they are declared without explicit initialization.

- Numeric types (`int`, `float`, etc.) → `0`
- Boolean type (`bool`) → `false`
- String type (`string`) → `""` (empty string)
- Pointer types → `nil`
- Slices → `nil`
- Maps → `nil`
- Channels → `nil`
- Interfaces → `nil`
- Functions → `nil`

Zero values ensure variables are always in a usable, predictable state without requiring explicit initialization.

The types for which nil serves as the zero value include channel, map, slice, pointer, function, and interface. These types may behave differently when uninitialized, potentially leading to runtime errors. For instance, attempting to write to an uninitialized map variable will result in a runtime panic (though reading from a nil map is safe and returns the zero value).

```golang
// uninitialized map
var myMap map[string]int

// This will cause a runtime panic
myMap["foo"] = 1
```

-----

# int vs int32 vs int64

In Go, `int`, `int32`, and `int64` are integer types that differ mainly in size and platform dependency.

- `int`
  - Size is platform-dependent
  - 32-bit on 32-bit systems, 64-bit on 64-bit systems
  - Fastest for general-purpose integer operations
  - Commonly used for array indexing and loops

- `int32`
  - Always 32 bits
  - Platform-independent size
  - Less memory compared to `int64`
  - Often used when exact integer size matters or for interoperability

- `int64`
  - Always 64 bits
  - Platform-independent size
  - Can store much larger values
  - Commonly used for timestamps, IDs, and large counters

Use `int` for general logic, `int32` or `int64` when you need a fixed-size integer or are interacting with external systems.


----

# byte vs rune

In Go, `byte` and `rune` are aliases for integer types used to represent characters, but they serve different purposes.

- `byte`
  - Alias for `uint8`
  - Represents a single raw byte
  - Commonly used for binary data and ASCII characters
  - Used in `[]byte` for byte-level operations

- `rune`
  - Alias for `int32`
  - Represents a Unicode code point
  - Can represent any Unicode character
  - Used in `[]rune` for Unicode-safe string operations

Use `byte` for raw data and ASCII processing, and `rune` when working with Unicode characters.


# Embedded structs

Embedded Structures are a way to reuse a struct's fields without having to inherit from it. It allows you to include the fields and methods of an existing struct into a new struct. The new struct can add additional fields and methods to those it inherits.

```golang

package main

import (
    "fmt"
)

type Address struct {
    City  string
    State string
}

type Person struct {
    Name    string
    Address //embedded struct
}

func main() {
    p := Person{
        Name: "John Doe",
        Address: Address{
            City:  "New York",
            State: "NY",
        },
    }

    fmt.Println("Name:", p.Name)
    fmt.Println("City:", p.City)
    fmt.Println("State:", p.State)
}
```

-----

# Explain the differences between var, :=, and const in Go?

### `var` (Explicit Variable Declaration)

- Used to declare variables with an explicit type or without an initial value.
- Can be used inside and outside functions.
- Supports zero-value initialization.

### `:=` (Short Variable Declaration)

- Only used inside functions (not at the package level).
- Implicitly infers the type from the right-hand side.
- Cannot be used to re-declare existing variables in the same scope.

### `const` (Constant Declaration)

- Used for defining immutable values (cannot be changed after assignment).
- Must be assigned a compile-time constant.
- Type can be explicitly specified or inferred.
----

# Explain the difference between new() and make()

- **Use `new()`** → When you need a pointer to a new, zeroed value of any type.
- **Use `make()`** → When creating slices, maps, or channels (to avoid `nil` references).

### new()

- `new(T)` **allocates memory** for a **zeroed value** of type `T` and returns a pointer `T`.
- It **does not initialize** the value, only **allocates** memory.

```golang
package main

import "fmt"

func main() {
    p := new(int)  // Allocates memory for an int, initializes to 0
    fmt.Println(*p) // Prints: 0 (default zero value)

    *p = 42
    fmt.Println(*p) // Prints: 42
}
```

### make()

- `make(T, args...)` **allocates and initializes** a **slice, map, or channel** (not other types).
- It **returns a value of type `T` (not a pointer)**.

```golang
package main

import "fmt"

func main() {
    s := make([]int, 5) // Creates a slice of length 5
    fmt.Println(s) // Prints: [0 0 0 0 0] (zeroed values)

    m := make(map[string]int) // Creates an empty map
    m["age"] = 25
    fmt.Println(m) // Prints: map[age:25]

    c := make(chan int, 3) // Creates a buffered channel
    fmt.Println(cap(c)) // Prints: 3 (capacity of the channel)
}
```

---

# What is the difference between `interface{}` and `any` in Go?

In **Go 1.18**, the **`any` type** was introduced as a **type alias** for `interface{}`.

## **1. `interface{}` (Empty Interface)**

- **Before Go 1.18**, `interface{}` was used as a **generic type** to hold **any value**.
- It **accepts all types** because every type in Go implicitly implements an empty interface.
- Example (interface)
    
    ```go
    package main
    
    import "fmt"
    
    func printValue(v interface{}) {
        fmt.Println(v)
    }
    
    func main() {
        printValue(42)      // int
        printValue("hello") // string
        printValue(3.14)    // float64
    }
    
    ```
    

## **2. `any` (Introduced in Go 1.18)**

- `any` is a **type alias** for `interface{}`, making the code **more readable**.
- It is **not a new feature**, just a naming convenience.
- Example(any)
    
    ```go
    package main
    
    import "fmt"
    
    func printValue(v any) { // Same as interface{}
        fmt.Println(v)
    }
    
    func main() {
        printValue(42)
        printValue("hello")
        printValue(3.14)
    }
    ```


----

# What are slices, and how are they implemented internally?

A **slice** is a dynamically-sized, flexible view into an **underlying array**. Unlike arrays, slices do **not have a fixed size** and can grow or shrink as needed.

Slices are **reference types**, meaning they do not hold data themselves but instead reference an **underlying array**.

When appending elements beyond **capacity**, Go **allocates a new array** (usually **doubles the size**), copies the elements, and updates the slice’s pointer.


# Combination example with *and&

In Go-lang, `*` (pointer de-reference) and `&` (address-of operator) are used together to work with pointers effectively. Let’s break it down with detailed explanations and examples.

## **1. Understanding `&` (Address-of Operator)**

The `&` operator is used to get the **memory address** of a variable. This means it creates a **pointer** that points to the given variable.

```go
package main
import "fmt"

func main() {
    x := 10
    p := &x // `p` now holds the memory address of `x`

    fmt.Println("Value of x:", x)        // 10
    fmt.Println("Address of x:", &x)     // Some memory address
    fmt.Println("Value of p:", p)        // Same memory address as &x
}
```

## 2. Understanding `*` (Pointer Dereference)

The `*` operator is used to **dereference a pointer**, meaning it retrieves the value stored at the memory address the pointer is pointing to.

```go
package main
import "fmt"

func main() {
    x := 10
    p := &x

    fmt.Println("Value of x:", x)     // 10
    fmt.Println("Value at p:", *p)    // Dereferencing `p` gives the value of `x` (10)
}
```

Here, `*p` retrieves the value stored at the memory address in `p`, which is `10`.

## 3. Combining `&` and `*` (`*&x` and `&*p`)

Now, let’s see how `*` and `&` interact when used together.

### **Case 1: `&x` → Getting the Value of x**

Expression: `*&x`

- `&x` gets the address of `x`
- `(&x)` dereferences that address, giving back `x`

```go
package main
import "fmt"

func main() {
    x := 10
    fmt.Println(*&x) // 10 (same as x)
}
```

**Explanation:**

- `&x` → Address of `x`
- `(&x)` → Dereference that address → Gives back `x`

### **Case 2: `&*p` → Getting the Address Back**

Expression: `&*p`

- `p` gets the value stored at pointer `p` (which is `x`)
- `&(*p)` gets the address of that value, which is just `p` again

```go
package main
import "fmt"

func main() {
    x := 10
    p := &x
    fmt.Println(&*p) // Address of x (same as p)
}
```

**Explanation:**

- `p` stores the address of `x`
- `p` gives the value of `x`
- `&(*p)` gives back the address of `x`, which is just `p`


----

# Pointer to Slice / Map Behavior in Go

Slices and maps in Go are reference types, meaning they already contain pointers to underlying data structures.

## Slice Behavior
```go
func updateSlice(s []int) {
    s[0] = 100
}

func replaceSlice(s []int) {
    s = []int{9, 9, 9}
}

func replaceSliceWithPointer(s *[]int) {
    *s = []int{9, 9, 9}
}

arr := []int{1, 2, 3}

updateSlice(arr)
fmt.Println(arr) // [100 2 3]

replaceSlice(arr)
fmt.Println(arr) // [100 2 3] (no change)

replaceSliceWithPointer(&arr)
fmt.Println(arr) // [9 9 9]

```

* Modifying slice elements affects the caller
* Reassigning the slice does NOT affect the caller
* Pointer to slice is required to replace the slice itself

```golang
func updateMap(m map[string]int) {
    m["a"] = 10
}

func replaceMap(m map[string]int) {
    m = map[string]int{"b": 20}
}

func replaceMapWithPointer(m *map[string]int) {
    *m = map[string]int{"b": 20}
}

data := map[string]int{"a": 1}

updateMap(data)
fmt.Println(data) // map[a:10]

replaceMap(data)
fmt.Println(data) // map[a:10] (no change)

replaceMapWithPointer(&data)
fmt.Println(data) // map[b:20]
```

* Maps are reference types
* Updating entries affects the caller
* Reassigning the map requires a pointer
