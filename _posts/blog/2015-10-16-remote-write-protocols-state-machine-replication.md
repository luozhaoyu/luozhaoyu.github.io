---
layout: post
title: "remote write protocols, state machine replication"
categories: blog
---
### ReplicatedService
2 nodes at first

Hard: keep both up to date

Assume: one-copy semantics or consistency or linearizability

#### Hard
* getting distributed systems to agree is hard ~ consensus problem: many forms of consensus:
    * agree on value
    * leader election
    * ordered, atomic multicast
* **Consensus**: is it possible?
    * Two generals problem

#### Why hard?
* Efficiency
* Failure: how to detect? (timing)
    * fail-stop model
        * it may be working
	* if failed, easy to detect
	* once failed, stays that way
    * byzantine failures: failed nodes may turn against you
* Concurrency

##### Approach #1
just send all requests to each replica

* Problems:
    * communication isn't reliable: message loss leads to *replica divergence*
    * order: different replica would receive in different order
    * failure/ recovery: how to **catchup**
    * deterministic: different replica with same input would generate different output

### [Optimal Primary-Backup Protocols](https://en.wikipedia.org/wiki/Consistency_model#Consistency_and_Replication)
* a globally unique time
* blocking protocols guarantee that processes view the result of the last write operation

1. client sends request: all requests sent to **primary**
- primary execute request
- gather **state** that has changed, send to backup
- backup stores state
- primary ack client

#### Good
* ordering is easy: only one entry
* determinism is no longer a problem

#### Issues
* how to set timeout on failure detector?
    * too long: reduce availability
    * too short: split brain
* when to ack client?
    * ack early:
        * latency is low
	* hard to maintain consistency: backup has not seen the updates
* recovery time
    * tend to be conservative with timeout based failure detector
        * it takes a while before backup takeover (because of the timeout)
* multiple backups
    * good: more fault tolerance
    * but:
        * more communication to keep replicas up to date
	* who takes over as **primary**?
	    * could have fixed ordering among backup nodes
	    * election(consensus)

#### NodeFailure
* Backup: catch up problem: how?
    * do logging to record what we have
    * entire state restore
* Primary:
    * how to detect primary failure?
        * backup/client pings to primary: "are you alive?"
    * **Failure detectors**
        * timeout based

#### Conclude
* P/B: basic approach to replicated services but seems simple
* issues exist
    * failure detection
    * still have to think about leader election


### [Implementing Fault-Tolerant Services Using the State Machine Approach: A Tutorial](https://en.wikipedia.org/wiki/State_machine_replication)

#### Reference
* [PPT](http://www.powershow.com/view1/160934-ZDc1Z/Implementing_FaultTolerant_Services_Using_the_State_Machine_Approach_A_Tutorial_powerpoint_ppt_presentation)
* [mtu.edu](http://www.cs.mtu.edu/~aebnenas/teaching/fall2010/cs5090/present/Steve/Steve-SMSurvey-presentation.pdf)
