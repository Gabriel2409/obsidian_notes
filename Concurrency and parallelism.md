#sd

Computers use parallelism and concurrency to handle the problems of multitasking

- to have **parallelism**, you need multiple comoutational units (CPUs or cores)
- to have **concurrency**, you need a way of scheduling tasks so that idle ones don't lock resources

```text
Concurrency
-[   ]---[ ]---> Task A
------[  ]--[]-> Task B

Parallelism
-[    ]--[   ]-> Task A
--[       ]-[]-> Task B

[  ] Executing
--- Waiting
```

Exemple of concurrent modules in CPython:

- threading: concurrent
- multiprocessing: concurrent and parallel
- asyncio: concurrent
- subinterpreters: concurrent and parallel
