#sd 
**Mutex (Mutual Exclusion Object)**

A synchronization primitive that ensures only one thread can access a shared resource (critical section) at a time.

## Key Operations

| Operation | Action | Outcome |
|---|---|---|
| **Acquire (Lock)** | Thread attempts to take ownership | If available, enters critical section. If locked, blocks until released. |
| **Release (Unlock)** | Owner relinquishes the mutex | Mutex unlocked; one waiting thread (if any) can now acquire it. |

## Core Feature: Ownership

**Only the thread that acquired the lock can release it.** This prevents accidental unlocks by other threads and is the defining characteristic of a mutex.

## Mutex vs. Binary Semaphore

While both Mutex and [[Semaphore]] can limit access to one thread, their semantics differ:

| Feature | Mutex | Binary Semaphore |
|---|---|---|
| **Ownership** | YES - Only owner can unlock | NO - Any thread can signal |
| **Safety** | High - Prevents mismatched lock/unlock | Lower - Vulnerable to misuse |
| **Primary Use** | Protecting shared data | Signaling between threads |

**Key Insight**: Use mutexes for protecting resources, semaphores for coordinating/signaling between threads.

Example: [[Double check locking]]