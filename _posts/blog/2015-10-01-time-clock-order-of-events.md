---
layout: post
title: "Time, Clocks, and the Ordering of Events"
categories: blog
---

### Time, Clocks, and the Ordering of Events in a Distributed System
* Process: stream of events
* Communicate: send message, receive
    * Model: message send takes time, reliable
* Ordering:
    * a, c: a is before c

#### syntax
"happens before"

* A -> B: A can causally affect B
* if A, B in same process, and A is before B (in timeline): A -> B
* if A send, B recv of msg: A -> B
* if A->B, B->C: A -> C

concurrent: if A !-> B, B !-> A: A || B

#### Clocks (logical not actual)
* per process: there is one clock
* Ci: clock for process i
* just counters

##### [Lamport timestamps](https://en.wikipedia.org/wiki/Lamport_timestamps)
for a, b: if a->b, c(a) < c(b)

rules:

* init: Clock := 0
* local: increment upon event
* communication:
    * send timestamp(t) with msg
    * upon recv: set time `local = max(timestamp, local) + 1`
* There is loss of information in msg->time update
* when come to total ordering
    * using some tie-broker (e.g., process ID when time is the same)

##### [Vector clock](https://en.wikipedia.org/wiki/Vector_clock)
* each process should track its view of **all** clocks
* each processi has:
    * array of clocks: [c1, c2, ..., ci, ..., cn]

rules:

* init: all vectors = 0
* local: inc ci by 1
* communication:
    * recv msg from Ps, contain s's vector-clock
    * when message is received, inc local clock first then set vector clock to piecewise max of incoming vector and local vector
        * update all local entries with max(local, sent_msg)
    * when message is sent, inc local clock first then send msg with vector

    if Ci[s] <= Cs[s]:
	Ci[s] = Cs[s] + 1
    for all j:
	Ci[j] = max(local, msg)


#### Reference
* [PPT](http://www.slideshare.net/payberah/time-43900531)
