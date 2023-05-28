#dsa #todo

TODO : rewrite

### Main principles

Assuming, key:value db
- Writes key:value on append only file (log), each record on a new line
- Maintains a hashmap where hashmap_key = key and hashmap_value = byte offset

### What happens on read/write/update/delete?
- on read, access of key in hashmap => byte offset => get value
- on update/write, appends key:value to log file, updates the hashmap
- on delete, appends key:tombstone (tombstone is special char indicating deletion) and updates the hashmap

### How to limit storage size?
- When log file becomes too large, another file is created (multiple segments)
- Each file maintains its own hashmap
- on read, accesses hashmap in reverse chronological order to get value
- Compaction process runs in the background to merge segments:
    - new segment file is created
    - old segments are parsed to determine the most recent value for each key which is then written to new file
    - Once process is finished, old segments are deleted and new segment is used

![compact](hash_index_merge_and_compact.jpg)

### How to recover after crash?
- Either read all of the segments to recreate the hashmaps or read from a hashmap snapshot
saved on disk

### Concurrency control
- Usually only one writer thread to avoid conflicts
- Multiple reader threads as log files are append only

### Limitations
- Hashmap must fit in memory
- Range queries are inefficient
