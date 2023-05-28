#sd #todo

TODO: rewrite

## SSTables and LSM-Trees (Indexing databases)

### What are SSTables?

SSTables = Sorted String tables
LSM-Trees = Log Structured Merge Trees

We assume a db/index where we store key:value

Contrary to Hash index, we don't append a new record to the end of the file. Instead,
we require that all the records are sorted by keys. This can be done with an in memory
data structure such as red-black tree or AVL trees. With these structures, you can
insert keys in any order and read them back in sorted order.

- Maintain an index similar to the one in standard hash indexes but no need to keep all
  of the keys. As they are sorted, the index can be sparsed.
- On write, add key:value to in memory tree (called memtable)
- When memtable gets larger than threshold, write it to disk (this becomes the most recent
  segment)
- On read, first try to find key in memtable then on segments in reverse chronological
  order. Finding a key consists in finding the closest key before it in the index and
  scanning until the next key in the index (as the index is sparse).

### How to limit storage size?

- From time to time, run compaction and merging of segments. Note that merging and
  compacting is faster than in standard hash indexes because records are already sorted
  by keys so we can combine them like it is done in the merge sort algorithm.
  NOTE: Storage engines that are based on this principle of merging and compacting
  sorted files are often called LSM storage engines

### How to recover after crash?

- On crash, we lose the in memory memtable. To avoid that it is possible to keep an
  append only log file (same as hash index) whose sole purpose is to help restore the
  memtable in case of crash. Everytyme the memtable is written, the log is discarded
