---
layout: post
title: "Log-Structured File System"
---

### [Introduction] (http://en.wikipedia.org/wiki/Log-structured_file_system)
Principle: collect large amounts of new data in a file cache in main memory, then write the data to disk in a single large I/O that can use all of the disk bandwidth

### Design for file systems of the 1990s
observations in 1990s:

* Memory sizes were growing
* There was a large and growing gap between random I/O performance and sequential I/O performance
* Existing file systems perfrom poorly on many common workloads: create new file needs to deal with inode, inode bitmap, directory data block, directory inode, new data block, data bitmap
* File systems were not RAID-aware: a logical write to a single block causes 4 physical I/O

### Log-structured file systems
* segment == large chunk
#### segment cleaning
* read M existing segments, compact into N new segments (N < M), write N segments to new locations to avoid fragmentations

### Crash recovery
#### Checkpoints

#### Roll-forward



### Reviews
* [OSTEP-LFS] (http://pages.cs.wisc.edu/~remzi/OSTEP/file-lfs.pdf)
* inode map solves the recursive update problem by updating only the directory? (is this a problem in other FS?)
#### disadvantage
* conventional file system normally keep files contiguously with in-place writes, while LSF fragmentes files
* in SSD, many flash based devices can not rewrite part of a block, so they have to erase cycle of each block before being able to re-write
