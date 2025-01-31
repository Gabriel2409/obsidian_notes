#sqlite 

https://www.youtube.com/watch?v=86jnwSU1F6Q&list=PLWENznQwkAoxww-cDEfIJ-uuPDfFwbeiJ&index=6
2 main journal modes: Rollback journal and [[Write Ahead Log|WAL]] mode

## Rollback journal

Modification: 
- Pull old data from database to OS file buffer to connection memory
- Save old data to rollback journal buffer
- sync journal with disk
- save updated data to OS File buffer
- Sync with disk
- Flush journal (in memory and disk)

![[sqlite-write-rollback-journal-mode]]

Crash recovery: 
- Look at journal on disk
- If not empty, reapply records on disk

Limitation: Only 1 concurrent writer because you need an exclusive lock

## WAL mode

Very different from [[Postgres - Theory]], here the WAL is not used only for recovery but for reading as well
On write: 
- Save new data to WAL buffer
- sync WAL with disk
- When WAL full, flush to disk and remove entries from WAL
- On read, first read from WAL then from db (WAL is more up to date)

![[sqlite-write-wal-mode]]

On crash recovery, nothing to do. Even if db is in inconsistent state for some records, we read from WAL anyways

WAL mode enables several writers at once (no need to lock)