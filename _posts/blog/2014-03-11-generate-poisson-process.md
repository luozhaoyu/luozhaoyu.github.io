---
layout: post
title: "泊松过程的生成"
---
为了模拟客户到访时间或者服务器请求到达时间，通常使用的模拟方式是[泊松过程](http://zh.wikipedia.org/zh-cn/%E6%B3%8A%E6%9D%BE%E8%BF%87%E7%A8%8B)

下面给出python源代码

    import random
    arrival_rate = 100
    if arrive_type == "poisson":
        arrive_times = []
        t = 0
        for i in range(arrival_rate):
            t += random.expovariate(arrival_rate)
            arrive_times.append(t)

证明过程参见Sheldon M.Ross的[Stochastic Processes, 2 edition](http://www.amazon.com/Stochastic-Processes-Series-Probability-Statistics/dp/0471120626)的P64的2.2 Interarrival and Waiting Time Distributions最后的remark
