---
sr-due: 2024-06-09
sr-interval: 8
sr-ease: 254
---

#sd

Very good explanation in the context of rust: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html

Stack and Heap (not to be confused with their datastructure counterpart [[Stack (datastructure)]] and [[Heap (datastructure)]]) are parts of memory available to your code to use at runtime, but they are structured in different ways.

## The Stack

The **stack** is a structured region of memory where values are stored and retrieved in a Last-In, First-Out (LIFO) order. This means that values are added to the stack in the order they are received and removed in the reverse order. Key stack operations include:

- **Pushing onto the stack:** Adding data to the stack.
- **Popping off the stack:** Removing data from the stack.

A critical requirement of the stack is that all data stored on it must have a known, fixed size. For example, a fixed size array will be stored on the stack
If the size of the data is unknown at compile time or may change, it must be stored on the heap. For example, strings are stored on the heap.

## The Heap

In contrast, the **heap** is a less structured region of memory. When data is placed on the heap, you request a specific amount of space. The memory allocator finds a suitable empty spot in the heap, marks it as in use, and returns a **pointer** that represents the address of that location. Important heap-related concepts include:

- **Allocating on the heap:** The process of requesting memory on the heap.
- **Pointer:** A reference to the actual data on the heap.

Notably, while the pointer to the heap has a known, fixed size, to access the actual data, you must follow this pointer.

## Performance Considerations

Pushing data onto the stack is faster than allocating on the heap since the stack's top location is always available for new data. In contrast, allocating space on the heap requires more effort as the allocator must locate an appropriate space and perform necessary bookkeeping.

Accessing data on the heap is generally slower than accessing data on the stack. This is because accessing data on the heap involves following a pointer, which introduces additional memory access overhead. Modern processors are more efficient when they have to jump around less in memory, as is the case with data stored on the stack.

## Function Call Handling

In typical programming, when a function is called, the values passed into the function, including pointers to data on the heap, and the function's local variables are pushed onto the stack. When the function execution is completed, these values are popped off the stack.

## Threads and processes

Each thread within a process typically has its own stack. A process can have multiple threads, and each thread will have its stack space. When a thread is created, a certain amount of stack space is allocated to it, and the stack size is usually limited. If the stack space is exhausted (e.g., due to deep recursion), it can lead to a stack overflow.

The heap memory is typically shared among all threads within a process. It's a region of memory where data with dynamic lifetimes can be allocated and deallocated. In a multi-threaded environment, managing the heap can be a significant challenge due to the potential for concurrent access by multiple threads. This concurrent access can lead to data races, race conditions, and other synchronization issues, which can result in unexpected behavior, crashes, or data corruption.

