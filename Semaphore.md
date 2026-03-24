#sd 

A Semaphore is a low-level synchronization primitive used to control access to a shared resource by maintaining a count of available permits. It functions as a signaling mechanism for flow control in concurrent programming.

A semaphore holds an integer value (the counter) representing the number of resources or permits currently available.

| Operation | Alias | Action | Effect on Counter |
|---|---|---|---|
| Wait | P / Acquire / Down | Attempts to take a permit. | Decrements the counter. If counter becomes negative, the thread blocks. |
| Signal | V / Release / Up | Returns a permit. | Increments the counter. If there are blocked threads, one is unblocked. |

Types
 * Counting Semaphore: The general form. The counter is initialized to N (number of resources).
 * Binary Semaphore: The counter is restricted to 0 or 1. It is functionally similar to a [[Mutex]] for mutual exclusion, but lacks the ownership enforcement of a true mutex.
 
Key Insight: Semaphores are primarily used for controlling access to a pool of resources (like limited database connections) or for simple inter-thread signaling.
