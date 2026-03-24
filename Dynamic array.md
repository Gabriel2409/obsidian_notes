#dsa 

## Definition
A dynamic array is a contiguous block of memory that resizes automatically.

Python's `list` is implemented as a dynamic array (CPython: `PyListObject`).

```c
struct PyListObject {
    Py_ssize_t ob_size;   // current length
    PyObject **ob_item;   // pointer to array of pointers
    Py_ssize_t allocated; // allocated capacity (>= ob_size)
}
```

- Python lists store elements in a contiguous block of memory.
- Each slot holds a reference (pointer) to a Python object, not the object itself.
- When a list grows beyond its current capacity, Python allocates a larger memory block and copies all references over.

## Basic operations
### Lookup by index
Direct memory offset calculation, no traversal needed.
Time complexity: O(1)

### Append
Inserts at `ob_item[ob_size]`.
Reallocates if ob_size == allocated.
Time complexity: O(1) amortized

Note: amortized because we may need to reallocate

### Insertion and deletion (arbitrary position)
All elements after the target index must be shifted.
Time complexity: O(n)
