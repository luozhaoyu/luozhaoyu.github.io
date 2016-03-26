---
layout: post
title: "Disk failures, data corruptions"
categories: blog
---

### An Analysis of Data Corruption in the Storage Stack
* Table 1: Corruption classes summary
    * This summary overturns my illusion that "once I have checksum and verified, I would trust the data". In fact, a block of data may pass the checksum verification but is still inconsistent since the firmware returns false negative or there is logic error due to misdirected writes

### Disk failures in the real world: What does an MTTF of 1 million hours mean to you?
* Figure 2: Lifecycle failure pattern for hard drives
    * It is surprising there is a "infant mortality" phenomena which indicates a high failure rate at the early age of hard drives, which contradicts to my intuition that the failure rate should look like an exponential distribution. In fact the article proposes the field replacement data looks more like [Weibul distribution](https://en.wikipedia.org/wiki/Weibull_distribution)

### An Analysis of Latent Sector Errors in Disk Drives
* Figure 10: The distribution of observed latent sector errors per day
    * The advantage of Enterprise disks over Nearline disks becomes clear only after the age around 1 year. So in fact, we do not need to worry too much about the sector error of Nearline disk at its first year, we may upgrade the cheap disk every year, and plus we would have additional software layer checksum to ensure data correctness
