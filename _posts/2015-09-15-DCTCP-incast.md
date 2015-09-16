---
layout: post
title: "DCTCP, incast"
---

### Basics of Distributed Computing
* communication
* failure
* time

#### Type of Distributed System
* local area (within datacenter)
* wide area (across datacenter)

    Similar                                 Different
    standard set of software protocols      homogeneity (level) control over design
    similar hardware                        performance
                                            security /workloads
                                            how they failure

#### Communication within datacenter
* RPC: do not use "TCP"
* U-Net: low latency, lightweight, specialized protocols, minimize overhead
* Reality:
    * TCP is commonly used
    * Curse of generality: TCP for everyone is not great for anyone
    * Workloads have evolved
        * 1 ~ N pattern (from single server to parallelism, bigger capacity)

##### Why TCP?
* Robustness (over many years)
* Reliability
* In-order
* End-to-end flow control (memory in end host problem)
    * speed match between sender and receiver
* congestion control, basic approaches
    * detect congestion
    * when happens, slow down (aggressively)
    * when no congestion, ramp up (conservatively)


### What happends in modren datacenter under modern workloads
* 99.91% traffic is TCP
* small traffic is large but small
* median concurrency level is 1, 75% is under 2
* incast: many-to-one response cause congestion
* TCP tends to be greedy, which could cause traffic fluctruation
* switch buffer is not enough
    * switch buffers are shared, short flow could be influenced by large flows on other ports
    * switch would detect if its queue size exceeds certain threshold in case buffer overflow
* receiver could tell the traffic congestion, while it is difficult for sender to figure out
    * However, it is still hard to figure out current bandwidth and RTT

### Incast
problem

* network switch memory is expensive
* limited memory, send big response size -> take up the queue

scenario

1. 1 -> N, N reply
- one reply lost
- client time-out, retry request
    * time-out is deadtime, no one is doing
- server reply

#### Solution to incast
* App-level: staggered send/reply
    * is good alternative
* Network-level: why wait so long to retry?
    * problems with quick retry?
* delayed ack
* timer resolution
    * traditional: coarse-grained (O(many milliseconds))
    * hardware support or "soft" timers
* desynchronized replies

primary difference, how they detect congestion?

* time-outs
* explicit notifications from switches
    * hard to know the parameterization

### Solutions
* ECN is used to limit the size of waiting queue to achieve lower latency

### [DCTCP] (https://en.wikipedia.org/wiki/Explicit_Congestion_Notification#DCTCP)
* [TCP Performance] (http://www.cisco.com/web/about/ac123/ac147/ac174/ac196/about_cisco_ipj_archive_article09186a00800c8417.html)

### If you are building a datacenter-based replicated storage system, what are the most important one or two lessons you learned from the two TCP papers you just read?
* Since frequent replication would eat up many bandwidth, I would agree with the delayed ACK solution to reduce the time spent in dealing ACKs. Anyway the network condition is rather reliable within the same datacenter
* The latency within datacenter is largely different from the outside web. So it is important to optimize the TCP default parameter, such as the RTO, windows size and so on. And so is the bandwidth inside the datacenter, we should take the switch buffer size into account since it becomes limited resource under high traffic. As we found there are so many problems in TCP, it maybe a good idea to design our own transmission strategy on top of the UDP protocol.

### Reference
* [DCTCP论文读书报告] (http://wenku.baidu.com/view/b57094573b3567ec102d8a40.html?re=view)
