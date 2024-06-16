---
reviewed: 2023-09-06
sr-due: 2024-06-10
sr-interval: 8
sr-ease: 250
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

## Example distributed file systems

- [[HDFS]]
