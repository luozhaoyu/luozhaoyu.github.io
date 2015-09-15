---
layout: post
title: "DCTCP, incast"
---

### Background
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

### Solutions
* ECN is used to limit the size of waiting queue to achieve lower latency

### [DCTCP] (https://en.wikipedia.org/wiki/Explicit_Congestion_Notification#DCTCP)
* [TCP Performance] (http://www.cisco.com/web/about/ac123/ac147/ac174/ac196/about_cisco_ipj_archive_article09186a00800c8417.html)

### If you are building a datacenter-based replicated storage system, what are the most important one or two lessons you learned from the two TCP papers you just read?
* I may not worry about the large file transmission, but I will take care of accessing the small files. They would be delayed by large files access, which cause unfairness. Different files could have different storage, transmission plans
* The latency within datacenter is largely different from the outside web. So it is important to optimize the TCP default parameter, such as the RTO, windows size and so on. And so is the bandwidth inside the datacenter, we should take the switch buffer size into account since it becomes limited resource under high traffic. As we found there are so many problems in TCP, it maybe a good idea to design our own transmission strategy on top of the UDP protocol.

### Reference
* [DCTCP论文读书报告] (http://wenku.baidu.com/view/b57094573b3567ec102d8a40.html?re=view)
