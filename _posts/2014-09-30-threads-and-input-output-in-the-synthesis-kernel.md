---
layout: post
title: "Synthesis kernel"
---

### design goal
1. high performance
    * combine kernel code synthesis, which decreases kernel call overhead through specialization
    * reduce synchronization
- self-tuning capability to dynamic load and configuration changes
- a simple, uniform and intuitive model of computation with a high-level interface


### Reviews
* [self-modifying code] (http://en.wikipedia.org/wiki/Self-modifying_code#Massalin.27s_Synthesis_kernel)
* principle of frugality: use the least powerful solution to a given problem
* [constant folding] (http://en.wikipedia.org/wiki/Constant_folding)
* [common subexpression elimination] (http://en.wikipedia.org/wiki/Common_subexpression_elimination)
