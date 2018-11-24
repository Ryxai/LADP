# Notes

Linearizability : A Correctness Condition for Concurrent Objects

## Introduction

Concurrent systems informally are a set of sequential processes communicating through shared type objects. Each object has a type and a possible set of values it ranges over and primitive operations that are the only means of creating/ manipulating those objects. The meaning of operations is given by their pre and post conditions. In concurrent systems it is necessary to give a meaning to possible interleaving of operation invocations.  A computation is linearizable if there is a legal sequential computation that can be derived from it. A  datatype's axiomatic specification only permits linearizable interleavings. Linearizability is a local property (a system is linearizable if all its objects are linearizable)  as compared to sequential consistency/serializability.  Locality is :thumbsup: for modularity and concurrency as objects can be implemented, verified separately and scheduling can be decentralized. It is also nonblocking, totally defined operations are never forced to wait (is :thumbsup: for concurrency and real-time systems). 

Two major kinds of problems that can be addressed:

* Correctness of linearizable object implementation using invariants and abstraction functions
* Reasoning about computations using linearizable objects by transforming assertions about concurrent computations into simpler assertions about sequential computations. 

## Motivation

Linearisability is useful in describing acceptable concurrent behavior in the form of acceptable sequential behavior.  To decide if a concurrent history is acceptable it is necessary to take into account the object's intended semantics. 

## System Model and Definition of Linearisability 

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

A history that is not sequential is concurrent. A process subhistory   $H \mid P$  of a history $H$ is a subsequence of all the events in $H$ whose process name is $P$. An object subhistory $H \mid x$ is defined for an object $x$. Two histories $H$ and $H'$ are equivalent if for every process $P$, $H \mid P = H' \mid P$. A history is well-formed if for each process the subhistory $H \mid P$ is sequential. Process histories of a well-formed history are sequential, object subhistories need not be. 

An operation $e$ is a pairing of an invocation and response $inv(e)$ and $res(e)$. An operation is written *[q inv/res A]* where:

* *q* is an object
* *A* is a process

An operation $e_0$ lies within another operation if  $inv(e_1)$ precedes $inv(e_0)$ and $res(e_0)$ precedes $res(e_1)$.  A set $S$ of histories is prefix-closed if whenever $H$ is in $S$, every prefix of $H$ is also in $S$. A single-object history is one with a single object. A sequential specification for an object is a prefix-closed set of single-object sequential histories for that object. A sequential history is legal if each subhistory $H \mid x$ belongs to the sequential specification for $x$. Object's sequential histories are summarized using a value which reflect the state of the object at the end of the history. Values are used in axioms giving the pre/post conditions on object operations. An operation is total if it defined for every object value, partial otherwise. 

### Definition of Linerizability

A history $H$ induces a irreflexive partial order $<_H$ on operations $e_0 <_H$ if $res(e_0)$ precedes $inv(e_1)$ in $H$.  Informally $<_H$ captures the real-time precedence ordering of operations in $H$. Operations unrelated by $<_H$ are concurrent. If $H$ is sequential $<_H$ is a total order.

A history is linearizable if it can be extended (by appending zero or more response events) to some history $H'$ such that

* L1 : *complete(H')* is equivalent to some legal sequential history $S$. i.e processes act as if there are interleaved at the granularity of complete operations. 
* L2: $<_H \subseteq <_S$, the apparent sequential interleaving respects the real-time ordering of operations

Extending $H$ to $H'$ includes the idea that some pending invocation may have taken effect even if the response has not been returned to the caller.  Focusing on *complete(H)* is used to establish pending invocations haven't had an effect yet. 

$S$ is a linearization of $H$. Nondeterminism is inherently part of linearizability:
1. For each $H$ there may be more then one extension $H'$ that satisfies L1 and L2.
2. For each extension $H'$ there may be more then one linearization $S$. 

A linearizable object is defined as one whose concurrent histories are linearizable with respect to a sequential specification.

## Properties of Linerizability

**Theorem 1** *$H$ is linearizable if and only if, for each object$x$, $H \mid x$ is linearizable.*

**Proof**:  The only if part is obvious. For each $x$ pick a linearization $H \mid x$. Let $R_x$ is the set of responses appended to $H \mid x$ to construct a linearization and let $<_x$ is the corresponding linearization order. Then let $H'$ be the history constructed by appending to $H$ each response in $R_x$. Construct a partial order on the operations of *complete(H')* such that:

1. For each $x$, $<_x \subseteq <$
2. $<_H \subseteq <$

Let $$ be the sequential history constructed by ordering the operations of *complete(H')* in any total order that extends <. Condition 1 implies that is legal and that L1 is satisfied and condition 2 implies L2 is satisfied.

Let < be the transitive closure of the union of all $<_x$ with $<_H$. From the construction of <, the conditions 1 and 2 are satisfied. If not, then there exists a set of operations $e_1, \ldots e_n$ such that $e_1 < e_2 < \ldots < e_n, e_n < e_1$ and each pair is directly related by some $<_x$ or $<_H$. 

Choose a cycle whose length is minimal. Suppose all operations are associated with the same object $x$. Since $<_x$ is a total order, there must exist two operations $e_{i-1}$ and $e_i$ such that $e_{i - 1} < _H e_i$ and $e_i <_x e_{i - 1}$ contradicting the linearizability of $x$.

  Thus the cycle must include at least two objects.  Let $e_1$ and $e_2$ be operations from distinct objects. Let $x$ be the object associated with $e_1$, claim $e_2, \ldots e_n$ can be an operation of $x$. This holds for $e_2$ by construction. Let $e_1$ be the first operation in $e_3, \ldots, e_n$ associated with $x$. Since $e_{i - 1}$ and $e_i$ are unrelated by $<_x$ they must be related by $<_H$, the response of $e_{i - 1}$ precedes the invocation of $e_i$, otherwise $e_{i - 1} <_H e_2$, yielding the shorter cycle $e_2, \ldots, e_{i - 1}$. Finally the response of $e_1$ precedes the invocation of $e_2$ since $e_1 <_H e_2$ by construction. It follows that the response to $e_1$ precedes the invocation of $e_i$ hence $e_1 <_H e_i$, yielding the shorter cycle $e_1, e_i, \ldots, e_n$. Since $e_n$ is not an operation of $x$ but $e_n < e_1$ it follows that $e_n <_H e_1$ but since $e_1 <_H e_2$ by construction and because $<_H$ is transitive, $e_n <_H e_2$ yielding the shorter cycle $e_2, \ldots, e_n$. $\square$

By this proof, only single object histories need to be considered. Locality allows concurrent systems to be designed in a modular fashion. Concurrent systems based on nonlocal correctness must used a centralized scheduler or use additional constraints to make sure scheduling is done properly. 

### Blocking versus Non-Blocking

A pending invocation of a totally-defined operation is never required to wait for another pending invocation to complete. 

**Theorem** *Let inv be an invocation of a total operation. If $<x \space inv \space P>$ is a pending invocation in a linearizable history H, there is exists a response $<x \space res \space P$ such that $H \cdot <x \space res  \space P>$ is linearizable.*

**Proof**  Let $S$ be any linearization of $H$. If $S$ includes a response $<x \space res \ space P>$ to $<x \space inv \space P>$ $S$ is also a linearization of $H \cdot <x \space res \space P>$. Otherwise $<x \space inv \space P>$does not appear in $S$ either since linearization by definition does not include pending invocations. As the operation is total, there exists a response $<x \space res \space P>$ such that $S' = S \cdot <x \space inv \space P> \cdot <x \space res \space P>$ is a legal linearization. However, $S'$ is a linearization of $H \cdot <x \space res \space P>$ and hence is a linearization of $H$.  $\square$

The result is that linearizability does not force a process with a pending invocation of a total operation to block.  Blocking/deadlock may occur still occur as a result of how linearizability is implemented. Furthermore, it does not prevent blocking it situations it is intended. The most natural concurrent interpretation of the partial sequential specification is t simply wait until the object reaches a defined state for the given operation. 

 ### Comparison to Other Correctness Conditions

Sequential consistency (as defined by Lamport) requires a history to be equivalent to a legal sequential history. Sequential consistency is weaker than linearizability, because it does not require the original history's precedence ordering to be preserved.  Furthermore sequential consistency is not a local property.  

In serializability, a transaction is a thread of control that applies a finite sequence of primitive ops to set of objects shared with other transactions. A history is serializable if it is equivalent to one in which the transactions appear to execute sequentially without interleaving. A partial precedence order can be defined over non-overlapping transactions.  A history is strictly-serializable if the order of the transactions in the sequential history is compatible with their precedence order.  It can be enforced by different synchronization mechanisms. 

Linearization can be viewed as a special case of strict serializability where transactions are restricted to consist of a single operation applied to an object.  Concurrency mechanisms that work for strict serializability  are not usually appropriate for linearizability.  Serializability (strict or otherwise) is not a local property.  Serializability is a blocking property. Special mechanisms used to roll back and restart transactions are needed to handle the blocking cases.  Linearization and serialization are used for different problem domains: 

* Serialization is used to allow programmers to reason about concurrency at serial applications.
* Linearizability is intended for applications where concurrency is of primary interest and programmers are willing to apply special-purpose synchronization protocols. 

## Verifying that Implementation are Linearizable

### Definition of Correctness

An implementation is a set of histories in which events of two objects, a representation object and abstract object (of type *rep* and *abs* respectively) are interleaved in  a constrained way. For each history $H$:

* the subhistories $H \mid rep$, and $H \mid abs$  satisfy the usual well-formedness conditions
* the processes $P$, each rep operation in $H \mid P$ lies within an abstract operation in $H \mid P$

An abstract operation is implemented by a sequence of rep operations that occur within it.  An implementation is correct with respect to the specification of *abs* if for every history $H$ in the implementation, $H \mid \it{abs}$ is linearizable. 

### Representation and Abstraction Fucntion

The subset of *rep* values that are legal are characterized by the rep invariant, $I : \it{rep} \rightarrow \it{bool}$ . The meaning of the legal representation is given by $A : \it{rep} \rightarrow \it{abs}$ the abstraction function. An abstract operation $\alpha$ is implemented by a sequence $\rho$  if rep operations that carries the representation from one legal variable to another, passing potentially though intermediate values where the abstract function is undefined. The rep invariant is part of the pre/post condition of every operation implementation.  An implementation of an abstract op $\alpha$ is correct if there exists a rep invariant $I$ and an abstraction function $A$ such that whenever $\rho$ carries one legal rep $r$ to another $r'$, $\alpha$ carries the value from $A(r)$ to $A(r')$. 

Concurrent objects, if allowed may have operations in progress continually and under these circumstances cannot assume that between operations it will assume a meaningful value, the rep invariant needs to be preserved by each rep operation in  the sequence implementing the abstract op. 

Defining an abstraction function can be difficult, ones where the operation 'takes effect' immediately fail due to the issues caused from linearization order depending on race conditions. The abstraction function is instead defined by mapping a rep value to a set of abstract values that represent the possible linearizations permitted, these can be fine-tuned depending on the level of concurrency (smaller set = less concurrency).  

### Verification Method

#### Linearizaed Values

The value at the end of a linearized history is a linearized value. An object can have more then one linearized value at the end of a given history. *Lin(H)* gives all the linearized values of $H$. 

#### Proof Method

The verification technique for sequential implementations is generalized: 

Assume the implementation of $r$ is correct, thus $H \mid \it{rep}$ is linearizable for all $H$ in the implementation. Show the following property: $\forall r \in \it{Lin}(H \mid \it{rep})$, $I(r)$ holds and $A(r) \subseteq \it{Lin}(H \mid \it{abs})$. This implies $\it{Lin}(H \mid \it{abs})$ is nonempty and thus linearizable. 

It is possible to linearized *abs* values with no corresponding *rep* values. 

#### Critical Sections

If an object's implementation includes critical sections, it may not be possible to define a continuous abstract function, the rep invariant may be violated in the critical section. Hidden data is used to overcome this, using an extended representation as the abstraction function's domain to overcome this.  This can be done by retaining lost data temporarily in auxiliary variables, after which the above proof method can be used. 

## Reasoning About Linearizable Objects

