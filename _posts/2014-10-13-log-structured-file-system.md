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
* clean code segment rather than hot (frequently used), but it is not the best approach

### Crash recovery
LFS will choose the most recent check point that is consistent, the check point region update procedure:

1. write the header, timestamp
- the body of check point region
- one last block (with a timestamp)

#### Checkpoints
* LFS writes check point region every 30s, so the last seconds of updates would be lost in crash

#### Roll-forward
* start with the last check point region, find the log, read through next segment to find valid update. Update as much as possible



### Reviews
* [OSTEP-LFS] (http://pages.cs.wisc.edu/~remzi/OSTEP/file-lfs.pdf)
* inode map solves the recursive update problem by updating only the directory? (is this a problem in other FS?)
* conventional file system normally keep files contiguously with in-place writes, while LSF tends to use unused disk (Copy-on-write)
    * this is called [Shadow paging] (http://en.wikipedia.org/wiki/Shadow_paging) in database
* in SSD, many flash based devices can not rewrite part of a block, so they have to erase cycle of each block before being able to re-write. This fits the Copy-on-write feature which circumvents in-place rewrite

#### disadvantage
* old copies are scattered throughout the disk, continuous cleaning is important
