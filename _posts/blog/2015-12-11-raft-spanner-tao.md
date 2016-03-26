---
layout: post
title: "spanner tao paxos raft"
categories: blog
---

### Raft
* [Raft user study] (https://ramcloud.stanford.edu/~ongaro/userstudy/)

### Spanner
* [Linearizability] (https://en.wikipedia.org/wiki/Linearizability)
    * In concurrent programming, an operation (or set of operations) is atomic, linearizable, indivisible or uninterruptible if it appears to the rest of the system to occur instantaneously.
* [Google全球级分布式数据库Spanner原理] (http://blog.jobbole.com/28096/)
* [从Google Spanner漫谈分布式存储与数据库技术] (http://history.programmer.com.cn/14015/)
* [Google NewSQL之Spanner] (http://www.leafonsword.org/google-spanner/)
* [GOOGLE分布式数据库技术演进研究--从Bigtable、Dremel到Spanner（三）] (http://802796.blog.51cto.com/8476804/1355302)
* [Wilson Hsieh - Spanner: Google's Globally-Distributed Database - OSDI 2012] (https://www.youtube.com/watch?v=NthK17nbpYs)

### TAO
* [TAO: Facebook’s Distributed Data Store for the Social Graph] (https://www.usenix.org/conference/atc13/technical-sessions/presentation/bronson)
    * Write-through cache
    * Objects are sharded by their ids
    * Why use invalidate/refill message to update rather than update follower tier cache directly?
	* followers may not store this cache
	* make cache consistency message idempotence
    * Leader cache could be used to deal with [thundering herd problem] (https://en.wikipedia.org/wiki/Thundering_herd_problem)