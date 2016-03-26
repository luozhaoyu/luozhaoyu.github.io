---
layout: post
title: "Eventual Consistency"
categories: blog
---

### Strong Consistency
* One copy, up to date
* Why we need it? it is easier to implement certain applications
* Why not? scale, latency, availability

### Weaken Consistency
* procuedure:
    1. client update to one server
    - commit update locally
    - later propagate updates
* good: latency, availability
* bad: harder to think about. e.g.,
    * weired results on read
    * **conflicts** on writes
        * ordering
	* conflicts is app-specific, hard to merge or resolve them

### Bayou
* design for mobile distributed computing
* **push app logic into updates function**
* guarantee: **eventually** all updates are applied in the same order on all servers
* goals:
    * substrate for distributed apps
    * not transparent: apps aware of distributed system
    * auto detect/resolution of **write conflict**
* bayou's stable log is "immutable"

#### What can we learn?
* Integrating application logic is key to "correct" execution of "AP" distributed systems. R/W is bad
* Lack of app-specific mechanisms in a coordination-free system is a recipe for data corruption
* Merge/repair is a narrow API for developers to express their application conflicts
* Alternatives like commutativity, I-confluence, and research tools like Bloom can help limit overhead

#### Bayou model
* Data Collections: replicated at servers: machines in basement or laptops
* Operations: queries/inserts of DB (read, write, insert, delete)
* Client: interact with **any**(high availability) server
* Provie developer with code: detect and react conflict
* assume node would fail then recover

##### How to propagate updates?
* Anti-entropy (epidemic)
* Basic idea:
    * periodically, each server picks some other server randomly
    * do exchange of updates

#### Conflict Detect/Resolve
    if Dependency_check(): # do you have $100 in account?
        Update_function() # transfer money
    else:
	Merge_function() # log error

* Dependency check, Merge protocol: establish state of DB
    * (query -> check for expected result, e.g., check existed primary ID)
	* if DB in expected: do update
    * e.g.
        * room reservation: expect room X is free @ 1~2 pm
	* bibliography DB: two editors input two very similar paper
* Dependency check fails => merge, what does merge do?
    * try different forms of update are app specific "acceptable" (e.g., different rooms, times)
        * but at somepoint, can fail and log it

#### Replica Consistency
* **keys**: well-defined **order**: unique timestamps
    * is this always needed? e.g., increment function
* detect/merge: deterministic
    * internal and external (malloc)
* how the replicas converge?
    * update: two states
        * tentative: might revert/change: so that we need to make a decision:
	    * use majority/consensus approach
	    * **primary** server
	* committed: final(stable)

##### anti-entropy protocol
* primary server as a serialization point: make decisions for the cluster, even though it would accept request which generated later
* if two nodes communicates, the older request win, while the result is still tentative

#### How to build?
* **write log**, **undo log** (allow us to rewind, apply all old updates again)


### Reference
* [papers-we-love: Peter Bailis](https://github.com/papers-we-love/papers-we-love/issues/193)
    * [Papers We Love Meetup - Managing Update Conflicts in Bayou](https://www.youtube.com/watch?v=txP7CI0PjO4)
    * [PPT](https://speakerdeck.com/paperswelove/pwlsf-number-10-equals-peter-bailis-on-managing-update-conflicts-in-bayou)
* [Cornell University](http://slideplayer.com/slide/5075267/)
* [Northwestern](http://www.aqualab.cs.northwestern.edu/classes/EECS345/eecs-345-w10/lectures/Bayou.pdf)
