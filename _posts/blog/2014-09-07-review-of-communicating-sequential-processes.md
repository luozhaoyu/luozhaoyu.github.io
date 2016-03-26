---
layout: post
title: "Review of Communicating Sequential Processes"
categories: blog
---
### Introduction
Essential Proposals

1. Dijkstra's guarded commands are adopted as sequential control structures(nondeterminism)
- Parallel command is based on Dijkstra's parbegin. All the processes start simultaneously, and the parallel command ends only when they are all finished
- Input and output command are used between concurrent processes
- **No automatic buffering, process has to be blocked**
- Input commands may appear in guards
- Repetitive command may have input guards
- A pattern-matching feature is used to inhibit input of mismatched messages

Serious Problems

1. Fails to suggest any proof method to assist in the development and verification of correct programs
- It may be slow on a traditional sequential computer
    * imposition of restrictions in the proposed features
    * reintroduction of distinctive notations for the most common and useful special cases
    * develop automatic optimization techniques
    * appropriate hardware

### Discussion
1. The notation is few and brief
- Every input or output command must name its source or destination explicity
- No port names for simplicity
- No automatic buffering
    * it is less realistic to implement in multiple disjoint processors
    * when buffering is required on a particular channel, it can readily be specified using the given primitives?
- Unbounded programs(array with no a priori bound) could be achieved by bounded programs with increasing bounds
- Programming language needs not to be fair
    * implementation should try to be reasonably fair
    * programmer should prove the correctness of terminations
- CSP is more machine-oriented imperative(procedural) approach than [the semantics of a simple language for parallel programming](http://www.tik.ee.ethz.ch/education/lectures/hswcd/papers/2_KahnProcessNetworks.pdf) which is more abstract applicative(functional) approach
- Output Guards needs further discussion
- Restriction: Repetitive Command With Input Guard needs further discussion

### Reviews
* Is the square symbol represent the nondeterministic choice?
* CSP is superior, because it forms a view from the problem other than the machines
* what forms event? event=(channel, source, destination, message)

#### Why CSP
* It could be included in a programming language(Go) as a special notation
* Expressibility

### Reference
#### about Hoare
* We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil
* There are two ways of constructing a software design: One way is to make it so simple that there are obviously no deficiencies, and the other way is to make it so complicated that there are no obvious deficiencies. The first method is far more difficult
* [CSP](http://en.wikipedia.org/wiki/Communicating_sequential_processes)
* Slogan of Go: Do not communicate by sharing memory; instead, share memory by communicating.
* [Share Memory By Communicating](http://blog.golang.org/share-memory-by-communicating)

### Notes
* messages has types
* how to implement output guard when deal with multiple clients in system view?
* any timeout is bad
* message just for simple communication, others for threads
