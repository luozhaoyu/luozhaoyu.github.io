---
layout: post
title: "Unix File System"
---

### [Introduction] (http://en.wikipedia.org/wiki/Unix_File_System)

### Old file system
Structure:

* the boot block
* the superblock
    * number of data blocks
    * count of maximum number of files
    * pointer to free list (linked list to all free blocks)
* a clump of inodes
    * ownership
    * last modification, access timestamps
    * direct blocks - 8
    * indirect blocks - singly, doubly, triply
* the data blocks
* disk drive is divided into partitions
    * each partition may contain one file system
    * a file system never spans multiple partitions

Problems:
* Long seek time from inode to data
* Files in single directory are not allocated consecutively
* 512B block size causes many seeks: next sequential block is not on the same cylinder
    * 1K block size would not solve problem: blocks are allocated randomly, all file system becomes scrambled after files creation and deletion


### New file system organization
* each disk drive contains one or more file systems
    * disk partition is divided into cylinder groups
        * divide block into 2, 4, 8, 16, 32, 64 ... fragments
        * block map is associated with each cylinder group
* super-block is replicated to protect against catastrophic loss

#### Optimizing storage utilization
* space is allocated when a program does a *write* system call

If the file needs to be expanded to hold the new data, one of three conditions exists:

1. There is enough space left in an already allocated block or fragment to hold the new data
- The file contains no fragmented blocks
    * fill new data into blocks
- The file contains one or more fragments
    * fill remaining new data into exisited fragments if they could be filled into

#### Layout policies

Allocator strategy:
1. Use the next available block rotaionally closest to the requested block on the same cylinder
- Use the same cylinder group if there is no block in the same cylinder
- If that cylinder group (break the disk up into smaller chunks) is entirely full, quadratically hash the cylinder group number to choose another cylinder group to look for a free block
- If hash fails, apply an exhaustive search to all cylinder groups

### File system functional enhancements
* Long file names
* File locking
* Symbolic links
* Rename
* Quota



### Reviews
* better locality of reference
* less wastage
* [Thrashing] (http://en.wikipedia.org/wiki/Thrashing_(computer_science))
* [Block suballocation] (http://en.wikipedia.org/wiki/Block_suballocation)
* [fsck] (http://en.wikipedia.org/wiki/Fsck), the right sequence of writing:
    1. allocate block
    - write data
    - update inode

