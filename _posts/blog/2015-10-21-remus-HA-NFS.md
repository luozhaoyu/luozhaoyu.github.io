---
layout: post
title: "Remus and HA NFS"
---

### Failures
* What kind of failures in hardware/software can it deal with?
    * Fail-stop failure and only one of primary or backup fails
        * it could handle both of them crash temporarily, since the data will remain crash-consistent
* What kind can't it deal with?
    * Can not recover from nonfail-stop conditions 
    * Both primary and backup fails permanently
    * Software bugs, which could propagate application errors to backup
* What happens when an unexpected failure arises?
    * if primary fails
	1. the backup node will try to wait the new checkpoint transmission until time-out
	- the alive node will upgrade itself as primary and resume execution from most recent checkpoint
    * if backup fails
        1. the primary will try to wait for a time-out of the backup responding to commit requests
	- the primary would have to replicate another backup

### Data Center
1. each service with it's own host (is wastful)
- so do virtualization
- then we need consolidation (another layer: VMM or hypervisor)
    1. However, x86 virtualization was "hard" (for trapping emulate)
    - then vmware: use software simulation of hardware
    - now hardware could offer more virtualization support

### Another problem: Fixed placement
* load imbalance
* fault tolerant?
* machine service/upgrade/etc
    * but, migration at VMM level is easier
* how to migrate?
    * CPU: easy, just register
    * Memory: large and changes quickly
    * Disk: shared storage
    * Net: redirect flows (networks are lossy)

#### Transfer memory in migration
* **Pre-copy**: before migration, start copying pages to target
    * Problem: redundant copies
* **Stop-and-copy**: stop service, copy, start target
    * Problem: slow
* **Post-copy**: lazy fetch, copy-on-write: fetch on demand
    * Problem: longer term dependency on Source host, slow

Hypothesis: Have migration, can we buil HA easier?

* similarity: running backup is similar to migration (which running at target)
* different:
    * Frequency: HA > state transfer
    * migration has acceptable service degradation
    * planned vs unplanned

### Remus
* Good:
    * **Transparent** HA via Primary/Backup
    * unmodified systems can use this
* Basic: Primary -> Basic
    * 同步: slow (CPU, mem speed >> net)
    * 异步: backup large primary
        * need to prevent observation of this lag
	* network indirection: buffer merge primary -> useful until backup has checkpointed state
* N-1 mode: N primaries share 1 backup

#### Failure model
* one down: fail stop failure
* both down: crash-consistent disk image
* could not handle: software bug, since that is non-determism

#### Mechanism
* memory transfer: frequent stop-and-copy
    * all irrelevant memory activities are replicated: very low efficiency
* net/disk buffering:
    1. periodic checkpoint (10~100ms)
    - net/disk buffering
    - copy **mem state** P -> B
        * page-table based state tracking: only send delta
    - resume primary execution
* disk: write through: backup commits checkpoint from mem to disk before resuming execution


### HA NFS
* make 2 servers, each be the bakcup of each other
    * dual-ported disks that are accessible to two servers
    * mirroring files on different disks
* one server down may put more works on the remain server
* [DRBD] (https://en.wikipedia.org/wiki/Distributed_Replicated_Block_Device)
