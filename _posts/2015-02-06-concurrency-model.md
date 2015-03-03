---
layout: post
title: "Concurrency Models"
---

### [Message Passing] (http://en.wikipedia.org/wiki/Message_passing)
Message passing sends a message to a process (which may be an actor or object) and relies on the process and the supporting infrastructure to select and invoke the actual code to run

* Message passing differs from conventional programming where a process, subroutine, or function is directly invoked by name, while it adds an intermediate layer to make choice:
    * Encapsulation: software objects should be able to invoke services on other objects without knowing or caring about how those services are implemented. IF-THEN statements that determine which subroutine or function to call is evil
    * Distribution: since objects are distributed, the message may not be executed immediately, it needs to be saved in a queue for further processing
    * Synchronous message passing: like OOP, Java, Smalltalk
    * Asynchronous message passing: called "middleware". Buffer size is a problem, full buffer would block sender then cause deadlock, while dropping message would reduce reliability
* Message passing should copy the value of the parameter content, while function call passes the address
* Message passing can be implemented by channels

#### [Channel] (http://en.wikipedia.org/wiki/Channel_(programming))
A channel is a model for interprocess communication and synchronization via message passing. A message may be sent over a channel, and another process or thread is able to synchronously receive messages sent over a channel it has a reference to, as a stream.

Channels are inherently synchronous: a process waiting to receive an object from a channel will block until the object is sent

### CSP
Processes communicate by sending or receiving values from named unbuffered channels. Since the channels are unbuffered, the send operation blocks until the value has been transferred to a receiver, thus providing a mechanism for synchronization. Other paradigms talk more about end-points than channels.

[history of CSP] (http://swtch.com/~rsc/thread)

### Actor
The Actor model is characterized by inherent concurrency of computation within and among actors, dynamic creation of actors, inclusion of actor addresses in messages, and interaction only through direct asynchronous message passing with no restriction on message arrival order.

Recipients of messages are identified by address, sometimes called "mailing address". Thus an actor can only communicate with actors whose addresses it has.

e.g. Electronic mail (e-mail) can be modeled as an Actor system. Accounts are modeled as Actors and email addresses as Actor addresses. Web Services can be modeled with SOAP endpoints modeled as Actor addresses.

### [CSP vs Actor model] (http://en.wikipedia.org/wiki/Communicating_sequential_processes#Comparison_with_the_Actor_Model)
Occam and Erlang are two well known languages that stem from CSP, while Go builds its concurrency primitives on channels which become its first-class object.

A channel is like a pipe. In the CSP model, message delivery is instantaneous and synchronous. The Go implementation is slightly different: Messages are buffered in channels in much the same way that Erlang buffers messages for delivery to processes.

Erlang is not modern CSP, because:

1. Its processes have identities, i.e., can be addressed
- Erlang allows you to send messages to processes, and Go allows you to send messages along channels

* [A Tale of Two Concurrency Models: Comparing the Go and Erlang Programming Languages] (http://thetrendythings.com/read/14522)
* [Effictive Go] (http://golang.org/doc/effective_go.html#concurrency)
