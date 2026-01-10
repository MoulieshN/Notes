# Implement the LinkedList (should be generic)

```go
package main

import (
	"fmt"
)

type Node[T comparable] struct {
	data T
	next *Node[T]
}

type LinkedList[T comparable] struct {
	head *Node[T]
	end  *Node[T]
}

// Insert adds a new node with the given data at the end of the list
func (ll *LinkedList[T]) Insert(data T) {
	newNode := &Node[T]{data: data}
	if ll.head == nil {
		// If the list is empty, set both head and end to the new node
		ll.head = newNode
		ll.end = newNode
	} else {
		// Otherwise, append the new node at the end and update the end pointer
		ll.end.next = newNode
		ll.end = newNode
	}
}

// Delete removes the first occurrence of a node with the given value from the list
func (ll *LinkedList[T]) Delete(value T) {
	if ll.head == nil {
		// If the list is empty, do nothing
		return
	}

	if ll.head.data == value {
		// If the node to delete is the head, move head to the next node
		ll.head = ll.head.next
		if ll.head == nil {
			// If the list becomes empty, reset end as well
			ll.end = nil
		}
		return
	}

	temp := ll.head
	for temp.next != nil && temp.next.data != value {
		// Traverse the list to find the node before the one to delete
		temp = temp.next
	}

	if temp.next != nil {
		// If found, update links to remove the node
		if temp.next == ll.end {
			// If the node to delete is the last node, update the end pointer
			ll.end = temp
		}
		temp.next = temp.next.next
	}
}

// Display prints the elements of the linked list in order
func (ll *LinkedList[T]) Display() {
	temp := ll.head
	for temp != nil {
		// Traverse and print each node's data
		fmt.Printf("%v -> ", temp.data)
		temp = temp.next
	}
	fmt.Println("nil") // Indicate the end of the list
}

func main() {
	// Example with integers
	ll := LinkedList[int]{}
	ll.Insert(10)
	ll.Insert(20)
	ll.Insert(30)
	ll.Display()

	ll.Delete(20)
	ll.Display()

	// Example with strings
	strList := LinkedList[string]{}
	strList.Insert("hello")
	strList.Insert("world")
	strList.Display()
}

```

# Implement the queue using linked list and should support all data types.

```go
package main

import "fmt"

// Node defines a node in the linked list
type Node[T comparable] struct {
    data T
    next *Node[T]
}

// Queue defines a generic queue
type Queue[T comparable] struct {
    front *Node[T]  // Pointer to the front node
    rear  *Node[T]  // Pointer to the rear node
    size  int       // Number of elements in the queue
}

// Enqueue adds an element to the rear of the queue
func (q *Queue[T]) Enqueue(item T) {
    newNode := &Node[T]{data: item}
    if q.rear == nil {
        q.front = newNode  // If the queue is empty, both front and rear will point to the new node
    } else {
        q.rear.next = newNode  // Link the current rear node to the new node
    }
    q.rear = newNode  // Move rear pointer to the new node
    q.size++          // Increment the size
}

// Dequeue removes and returns the element from the front of the queue
func (q *Queue[T]) Dequeue() (T, bool) {
    if q.front == nil {
        var zero T
        return zero, false // If the queue is empty, return zero value and false
    }
    item := q.front.data
    q.front = q.front.next  // Move the front pointer to the next node
    if q.front == nil {
        q.rear = nil // If the queue becomes empty, set rear to nil
    }
    q.size-- // Decrement the size
    return item, true
}

// Size returns the current number of elements in the queue
func (q *Queue[T]) Size() int {
    return q.size
}

func main() {
    // Create a queue of integers
    q := Queue[int]{}

    // Enqueue some elements
    q.Enqueue(10)
    q.Enqueue(20)
    q.Enqueue(30)

    fmt.Println("Queue size:", q.Size()) // Output: Queue size: 3

    // Dequeue an element
    item, ok := q.Dequeue()
    if ok {
        fmt.Println("Dequeued:", item) // Output: Dequeued: 10
    }

    fmt.Println("Queue size after dequeue:", q.Size()) // Output: Queue size after dequeue: 2
}

```