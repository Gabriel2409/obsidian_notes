#sd 

Double-Checked Locking (DCL) is a technique used specifically to implement the [[Design patterns - Singleton|Singleton]] pattern in a multithreaded environment while maintaining high performance.
Its goal is to ensure that a Singleton instance is created only once, minimizing the time spent under a synchronization lock.

DCL involves two checks of the instance variable:
 * First Check (Without Lock):
   * The code checks if the instance is null before acquiring the lock.
   * Purpose: If the instance is already created, the thread avoids the overhead of acquiring and releasing the lock, allowing rapid access.
 * Acquire Lock:
   * If the instance is null, the thread proceeds to acquire a [[Mutex]] (lock) to prevent other threads from entering the critical creation section.
 * Second Check (Inside Lock):
   * The code checks if the instance is still null after acquiring the lock.
   * Purpose: This prevents a race condition. If two threads try to acquire the lock simultaneously, the first thread creates the instance, and the second thread is stopped until block is released, then when it finally acquires the lock, this inner check prevents creating a duplicate.
   
Important Note on Volatility
In many languages (like Java and C++), DCL requires the shared instance variable to be declared as volatile. This is necessary to prevent compiler and processor optimizations that could reorder memory writes, ensuring that the fully initialized object is visible to all threads after creation.
