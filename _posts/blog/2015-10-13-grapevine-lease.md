---
layout: post
title: "Grapevine, Leases"
categories: blog
---

### [Grapevine: An Exercise in Distributed Computing](http://pages.cs.wisc.edu/~sschang/OS-Qual/distOS/grapevine.htm)
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

#### Registration Service
implementing distributed database

* authentication
* lookups/additions/deletions to database
    * every entity has *name* (RName): two types of entity:
        * **groups**: list of RNames
	* **individual**: person(inbox sites), machine(network address)


* Grapevine Registry: GV (fully replicated)
    * GV is group contains all names of registry servers
* Bootstrap problem: how to find other servers?: manually input message servers

##### Replication
* replicates *registry*: all on particular registry server
    * Goal: eventually consistent (can observe inconsistencies, but after time T, they will resolve)
* internal representation: (e.g., group)
    * group: two lists: active_list, deleted_list (entity can be on **one** list at a time)
        * garbage collection on deleted_list, maybe every 2 weeks

##### Operations
* update: add(), delete()
    1. remove entity from active/deleted list
    - generate timestamp, unique ID
    - put entity on active/deleted list
* merge
    1. process do local update, then generate change messages for other replicas
    - takes two lists, merge into one using **timestamps**
        * change messages may get lost: we may need to sync the servers after a period of time

#### Delivery Service
* implemented by group of *message servers*
    * `accept()`: here is some mail to deliver
    * `poll/check()`: do I have mail?
    * `retrieve`: get my mail

##### Message Delivery
1. prepare email: body, list of recipients (inbox sites)
- **Accept**: pick **any** message server and give it the message
    * nothing else is **replicated**, but only the service replicated, you could pick **any** server
- message server: **store** locally and **say OK**


### [Leases: An Efficient Fault-Tolerant Mechanism for Distributed File Cache Consistency](https://en.wikipedia.org/wiki/Lease_(computer_science))
* Leases give great management flexibility.
    * The server could control the term of leases, and it could choose from seeking write approval or wait for lease expire
	* A heavily write-shared file might be given a lease term of zero
	* A heavily read-shared file could be optimized by using a smaller number of leases, e.g., one lease per directory
	    * to eliminate the need for clients to request lease extensions
    * The client could choose when to request extension, relinquish and approve write
* lease is contract: rights for **limited time**
    * allow to make sensible progress (slow client will not block others too much, we have expiration)
* client wants to access file F
    1. request lease(time period) from server for F
    - if re-access file before lease expires: use it without server interaction
        * otherwise, when expires, renew it before re-use
* different clients want to modify F
    1. have to contact all lease holders to get permission
    - It is OK if one client doesn't reply

#### Design
* provide strict consistency in spite of non-Byzantine failures, including partitions
* a key assumption is that clocks are reasonably accurate
    * use physical clocks, real clock

#### Leases
* lease extenstion overhead: extension after lease expires
    * on-demand extension rather than periodic extension
* false sharing: leaseholder hold the lease without accessing the file, while others could not write
* [Scaling Memcache at Facebook, thundering herds](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf)
    * without lease, all cache misses would go to database query
    * with lease, I would wait for a short time to let other leaseholder to set the data, so that I will not query the DB

##### Lease Length
* shorter term is good
    * *false sharing*: reduces "false" conflicts
    * server availability: server could reboot quickly (after crash)
* longer term is good
    * reduces overheads for extensions

#### Crash Recovery
* simple:
    * client: forget all leases upon reboot
    * server: **wait**(until all leases have expired)
