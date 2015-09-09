---
layout: post
title: "U-Net, DCTCP"
---
### Why this paper
* How to improve performance and flexibility of netwroking layer performance?
    * move parts of the protocol processing into user space
    * remove kernel completely from the critical path and allow the communication layers used by each process to be tailored to its demands
    * the entire protocol stack should be placed at user level
    * OS and hardware should allow protected user-level access directly to the network

* Intents:
    * provide low-latency communication in local area settings
        * `processing_overhead = transmission_time + buffer_management + message_copies + checksum + flow-control + interrupt overhead + network_interface_control`
    * exploit the full network bandwidth even with small messages
    * faciliate the use of novel communication protocols

### U-Net
* each process has the illusion of owning the interface to the network

### The U-net paper discusses the notion of zero copy vs. true zero copy. What is the point being made here? Is this important for high performance applications and services?

#### Review
* U-Net has two levels of sophistication
    * *base-level* is *zero copy* requires an intermediate copy into a networking buffer
    * *direct acess* is *true zero copy*, which has no intermediate buffering

#### Comment


### [DCTCP] (https://en.wikipedia.org/wiki/Explicit_Congestion_Notification#DCTCP)
* [TCP Performance] (http://www.cisco.com/web/about/ac123/ac147/ac174/ac196/about_cisco_ipj_archive_article09186a00800c8417.html)
