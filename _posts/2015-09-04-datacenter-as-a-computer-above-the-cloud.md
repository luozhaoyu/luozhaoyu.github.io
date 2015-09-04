---
layout: post
title: "The Datacenter as a computer, Above the Clouds"
---
### The Datacenter as a Computer
#### What is the theme
This article gives us a fundamental introduction about a new class of computing system *warehouse-scale computers*. This system has massive scale of their software infrastructure, data repositories and hardware platform.

This system is innovative since it focuses on scale-out rather than scale-up.

##### Features
* large scale
    * large numbers of component faults
    * high availability
* run a smaller number of very large applications
* significant deployment flexibility
* replicas between datacenters
* emphasis on cost efficiency

#### How to do
* One user query tends to be fully processed within one datacenter
    * view multi datacenters as seperate computing resources
    * query across datacenters is expensive
* A fault-tolerant file system
    * it could lower hardware costs and networking fabric utilization
        * but difficult to implement
    * it could support different machines
    * it would use more networking bandwidth to complete write operations
    * Otherwise, use NAS, which provides extra reliability but more expensive
* exploit rack-level networking locality
    * Higher port counts would cost exponential higher cost
    * intra rack connectivity is often cheaper than inter rack connectivity
* WSC is often organized as two-level hierarchy
    * rack level
    * cluster level


#### Challenge
* Size is too big to experiment with or simulate efficiently
* New programming challenge over individual servers
* How to smooth out the discrepancies which lie in latency, bandwidth and capacity of different racks

#### Comments
* Larger computing resource is always needed, while cost-efficiency is always the key to design large WSC
* Large hardware saving is usually achieved by implementing a software level application. In the long run, the software solution would become cheaper. Even though the software and hardware approaches may share the same idea to achieve higher availability or capacity, the software is always more easy to do modification and maintenance
* To lower the cost of purchasing machines, people tend to procure more machines with a same type at a time. While datacenter upgrade may be conducted partially at a time. So it is the high level application which would take the responsiblity to handle the isomerism
    * In this way, no matter how the hardware techniques evolves (such as biology computing or quantum computing), each new computing unit should support basic stream read and write. An unified low level communication protocol is more important than other design assumptions. And this kind of protocol would last far more longer than the hardware revolution.
* As the size of network increasing, there will be more hierarchies adopt inside WSC. If the estimated over-subscription is low, we may use some software level switch to transmit data above some uniformed switch device
* Datacenter itself could also evolve into different kinds of use cases. e.g., some datacenters or their smaller unit, cluster could bind together to handle some specific computing problem

### Above the Clouds
* Data Lock-In
    * This is definitely the small clould providers' opportunity to combat against AWS or other large cloud providers. I could say for sure those large clould providers such as AWS would not have any lucrative incentive to think for its potential competitors. AWS's (just raise it as an example) best strategy is to keep its API exceptional and make the future "standard" looks as similar as its'.
    * Whatever happens, those small would be more enthusiasstic in promoting standarlization
* Data Transfer Bottlenecks
    * It is surprising that CPU develops much faster than the bandwidth
    * Transmitting efficiency would always be the central topic. HTTP2, long live connection, multi-push, compression, new media format would be hotly discussed. Hacking on the TCP/IP would continue
* Bugs in Large-Scale Distributed Systems
    * Personally I don't think the explanation is viable. Simulating the actual distributed enviroment is still exhausting and costly, even though one could create those VMs as reliable as real servers
    * Fuzz test may be an alternative way to find bugs
* Reputation Fate Sharing
    * Blacklisting of IP address itself is wrong. Please blacklist the domains instead
    * I am optimistic that if there exists lawsuit against cloud provider about spaming, the court would refer the [DMCA] (https://en.wikipedia.org/wiki/Digital_Millennium_Copyright_Act)'s principle inovation: **exemption from direct and indirect liability** of ISP and other intermediaries

