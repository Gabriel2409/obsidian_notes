---
reviewed: 2023-09-06
sr-due: 2025-07-22
sr-interval: 323
sr-ease: 270
---

#sd

## Definition

A file system is used to navigate the data inside the storage

- controls how data is stored and retrieved
- metadata at files and folders
- manages permission and security
- manages storage efficiently

## Examples of local file systems

Microsoft

- FAT32: 4GB file limit, 32GB volume limit
- NTFS (New Technology File System): 16EB / 16EB

Apple

- HFS (Hierarchical File System): 2GB / 2TB
- HFS+: 8EB / 8EB

Linux

- ext3: 2TB / 32TB
- ext4: 16TB / 1EB
- XFS: 8EB / 8EB

In Linux, check the FS by running `df -t`


## High level Overview of ext4

- System contains an inode table. An inode (index node) is a datastructure that stores metadata about the file/directory and a pointer to the contents
- For small file, the inode points directly to the datablocks (or extents= continuous ranges of blocks)
- For larger files, it points to a B+tree like datastructure that allows to manage datablocks more easily.
- For directory, extents contains among other thing the list of child inodes number, for efficient retrieval in the inode table


## Example distributed file systems

- [[HDFS]]
