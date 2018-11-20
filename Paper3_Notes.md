# Notes

Linearizability : A Correctness Condition for Concurrent Objects

## Introduction

Concurrent systems informally are a set of sequential processes communicating through shared type objects. Each object has a type and a possible set of values it ranges over and primitive operations that are the only means of creating/ manipulating those objects. The meaning of operations is given by their pre and post conditions. In concurrent systems it is necessary to give a meaning to possible interleaving of operation invocations.  A computation is linearizable if there is a legal sequential computation that can be derived from it. A  datatype's axiomatic specification only permits linearizable interleavings. Linearizability is a local property (a system is linearizable if all its objects are linearizable)  as compared to sequential consistency/serializability.  Locality is :thumbsup: for modularity and concurrency as objects can be implemented, verified separately and scheduling can be decentralized. It is also nonblocking, totally defined operations are never forced to wait (is :thumbsup: for concurrency and real-time systems). 

Two major kinds of problems that can be addressed:

* Correctness of linearizable object implementation using invariants and abstraction functions
* Reasoning about computations using linearizable objects by transforming assertions about concurrent computations into simpler assertions about sequential computations. 

## Motivation

Linearizability is useful in describing acceptable concurrent behavior in the form of acceptable sequential behavior.  To decide if a concurrent history is acceptable it is necessary to take into account the object's intended semantics. 

## System Model and Definition of Linearizability 

### Histories

A concurrent system is a collection of sequential threads of control (processes ) that communicate through shared data structures (objects).  Objects have unique names and types. Types define a possible set of values and operations provide the only means of manipulating that object. Processes themselves are sequential (applies a sequence of operation invoking and receiving responses). Dynamic process creation can be modeled with each child process acting as a process with no operations before the fork or after the join. Each execution is modeled a history ( a set of invocation sand responses). A sub-history is a subsequence of the events in H. 

Invocations are written *<x op(args\*) A>* where:

* *x* is an object name
* *op* is an operation name
* *args\** denotes a sequence of argument values
* *A* is a process name

An argument response is written *<x term(res\*) A>* where:

* *term* is a terminal condition 
* *res\** is a sequence of results

'Ok' is used as normal termination. A response matches an invocation if their object names and process names agree. An invocation is pending in a history if there is no matching response following the invocation. Given a history *H*, *complete(H)* is the maximal subsequence of *H* consisting of invocations and matching responses. 

A history *H* is sequential if:

* First event of *H* is an invocation
* Each invocations (except possibly the last) is matched by an immediately following response. 

A history that is not sequential is concurrent. A process subhistory  $H \mid P$ of a history $H$ is a subsequence of all the events in $H$ whose process name is $P$. An object subhistory $H \mid x$ is defined for an object $x$. Two histories $H$ and $H'$ are equivalent if for every process $P$, $H \mid P = H' \mid P$. A history is well-formed if for each process the subhistory $H \mid P$ is sequential. Process histories of a well-formed history are sequential, object subhistories need not be. 

An operation $e$ is a pairing of an invocation and response $inv(e)$ and $res(e)$. An operation is written *[q inv/res A]* where:

* *q* is an object
* *A* is a process

An operation $e_0$ lies within another operation if $inv(e_1)$ precedes $inv(e_0)$ and $res(e_0)$ precedes $res(e_1)$.  A set $S$ of histories is prefix-closed if whenever $H$ is in $S$, every prefix of $H$ is also in $S$. A single-object history is one with a single object. A sequential specification for an object is a prefix-closed set of single-object sequential histories for that object. A sequential history is legal if each subhistory $H \mid x$ belongs to the sequential specification for $x$. Object's sequential histories are summarized using a value which reflect the state of the object at the end of the history. Values are used in axioms giving the pre/post conditions on object operations. An operation is total if it defined for every object value, partial otherwise. 

### Definition of Linerizability

A history $H$ induces a irreflexive partial order $<_H$ on operations $e_0 <_H$ if $res(e_0)$ precedes $inv(e_1)$ in $H$.  Informally $<_H$ captures the real-time precedence ordering of operations in $H$. Operations unrelated by $<_H$ are concurrent. If $H$ is sequential $<_H$ is a total order.

A history is linearizable if it can be extended (by appending zero or more response events) to some history $H'$ such that

* L1 : *complete(H')* is equivalent to some legal sequential history $S$. i.e processes act as if there are interleaved at the granularity of complete operations. 
* L2: $<_H \subseteq <_S$, the apparent sequential interleaving respects the real-time ordering of operations

Extending $H$ to $H'$ includes the idea that some pending invocation may have taken effect even if the response has not been returned to the caller.  Focusing on *complete(H)* is used to establish pending invocations haven't had an effect yet.