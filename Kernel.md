#sd

Kernel is in charge of managing tasks in 4 general system areas:
- [[Process|Processes]]: Kernel is responsible for determining which processes are allowed to use the CPU
- Memory: Kernel keeps track of what is allocated to a particular process, what is shared and what is free
- Device drivers: Kernel acts as an interface between hardware and processes. 
- System calls and support: processes use syscalls to communicate with the kernel

## Process management

Only one process can use the CPU at the same time (in a one core system)
In practice, the kernel is responsible for context switching and runs between time slice. 

Example breakdown: 
- a process run for a time slice, 
- CPU (actual hardware) interrupts process based on internal timer, switch to kernel mode and gives  control to the kernel 
- Kernel records state of cpu and memory (will be useful to resume later)
- Kernel performs pending tasks 
- Kernel analyzes list of processes and choose one, prepares the memory and the CPU, and tells the CPU how long the time slice for the process will be
- Kernel switches CPU into user mode and hands control of the CPU to the process

Note: we still have the impression of multitasking because a time slice is very short. 

## Memory management

In order to manage memory during a context switch:
- the kernel must have its own private area in memory (not accessible by user processes)
- Each user process needs its own section of memory
- One user process can't access the private memory of another process. However, user processes can share memory. Moreover some memory in user processes can be read only
- The system can user more memory than physically present by swapping to disk

Note: modern CPU inlude a memory management unit (MMU) that enable a memory access scheme called virtual memory. The kernel then sets up each process as if it had the entire machine to itself and when the process accesses memory, it is intercepted by the MMU that uses a memory address map. The implementation of a memory address map is called a page table. 

## Device drivers
A device is typically only accessible in kernel mode because improper access could crash the machine. Because different devices rarely have the same programming interface, device drivers are part of the kernel to present a uniform interface to user processes

## System calls and support
User processes can use System calls to perform specific tasks that they can't do well by delegating to the kernel (for ex opening, reading and writing files) 
For ex: 
- fork():  When a process calls fork(), the kernel creates a nearly identical
copy of the process.
- exec() When a process calls exec(program), the kernel loads and starts
program, replacing the current process


Other than init, all new user processes start as a result of fork