---
layout: post
title: "Grapevine, Leases"
---

### [Grapevine: An Exercise in Distributed Computing] (http://pages.cs.wisc.edu/~sschang/OS-Qual/distOS/grapevine.htm)
* Compared with hardware, software failure could be more arduous. Same version software could get crashed by the same bug which results in crash totally
    * `nohup` is a must to alleviate from having all the servers crash in a short period before anyone notices that

#### Design
* do not interpret the content of the message(MIME), let clients do
* eventual consistency
    * server propagates the change to the replicas of the entry in other servers
        * other servers would merge change message with their own copy in the future
    * two conflicting updates (e.g., add and delete) with different ordering in two different hosts is acceptable
        * designer thinks it is not a problem since it does not communicate with outside system
	* the clients should cope with transient inconsistencies
* independent failure mode
* separate naming and addressing
* separate service from interface

#### Failure
* Change message get destroyed
    * each registration server would resynchronize the data with others periodically
        * use timestamp to conduct merges


### [Leases: An Efficient Fault-Tolerant Mechanism for Distributed File Cache Consistency] (https://en.wikipedia.org/wiki/Lease_(computer_science))
* Leases give great management flexibility.
    * The server could control the term of leases, and it could choose from seeking write approval or wait for lease expire
	* A heavily write-shared file might be given a lease term of zero
	* A heavily read-shared file could be optimized by using a smaller number of leases, e.g., one lease per directory
	    * to eliminate the need for clients to request lease extensions
    * The client could choose when to request extension, relinquish and approve write

#### Design
* provide strict consistency in spite of non-Byzantine failures, including partitions
* a key assumption is that clocks are reasonably accurate
    * use physical clocks, real clock

#### Leases
* lease extenstion overhead: extension after lease expires
    * on-demand extension rather than periodic extension
* false sharing: leaseholder hold the lease without accessing the file, while others could not write
* [Scaling Memcache at Facebook, thundering herds] (https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf)
    * without lease, all cache misses would go to database query
    * with lease, I would wait for a short time to let other leaseholder to set the data, so that I will not query the DB
