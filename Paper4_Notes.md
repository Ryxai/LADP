# Notes

Causal Memory: Definitions, Implementation and Programming

## Introduction

Shared memory is becoming an important abstraction. Distributed memories ensure the processes agree on some common order of operations on the memory.  Weakening of memory consistency can reduce the costs of providing the consistency while maintaining a viable target model for programmers.  

Causal memory is an abstraction that ensures that processes agree on the relative ordering of operations that are causally related. This is based on the concept of potential causality defined by Lamport. Causal memory requires that reads return values consistent with causally related reads and writes. Causality provides a partial order, and readings may disagree on the ordering of concurrent writes, allows for independent concurrent writers. Synchronization required for a program is often specified explicitly and not necessary for memory to provide additional guarantees. 

Causal memory is based on the use of vector timestamps (as is ISIS causal broadcast), both are nonblocking.  Memory has overwrite semantics and messages have queueing semantics.  Message recipients can be assured it will receive all messages sent to it, but repeated reads cannot be guaranteed to read all values written. 'Hidden writes' are possible. In message passing systems reading in any order violates causal ordering. 

Programs that are sequentially consistent are also causally consistent (weaker definition), data-race free programs make use of explicit synchronization to prevent problems that arise from concurrent access to shared memory. 

The tradeoff for strongly consistent memories are that they require costly blocking implementations while cheap weak memories are more difficult to program in. 

## Shared Memory Systems

A system is a finite set of process that interact via a shared memory that consists of a finite set of locations. Let $\mathscr{P} = \{p_1, p_2, \ldots, p_n \}$ be the set of processes. A process interacts with a location in memory through read and write operations. Each operation has a named location and associated value. A write op for a process $p_i$ storing the value $v$ in location $x$ is denoted $w_i(x)v$. A read for the same location reporting $v$ is in location $x$ is denoted $r_i(x)v$. A local history of a process $p_i$ is denoted $L_i$ and is a sequence of read/write ops where $o_1$ precedes $o_2$ in program order is written $o_1 \rightarrow o_2$.  An execution history $H$ is a collection of local histories for each process and is defined $H = <L_1, L_2, \ldots, L_n>$. An operation is in $H$ if it is in one of the local histories $L_i$ in $H$. Different memories are defined by considering serializations of certain sets of operations. If $A$ is a history then $S$ is a serialization of $A$ if $S$ is a linear sequence containing exactly the operations of $A$ such that each read operation from a location returns the value written by the most recent preceding write to that location. Assume each location has an initial value $\bot$ and is returned if there is no preceding write at a location. Serialization order respects $\rightarrow$ order. 

## Earlier Memory Models

A history is sequentially consistent if there is a serialization $S$ of $H$ that respects all the program orders $\rightarrow_i$. A memory is sequentially consistent if it only admits sequentially consistent histories. Pipelined RAM (PRAM) is an alternative that requires only the write operation be seen in program order at all other processes, each process sequences it own operations and the writes of other processes. Given history $H$ and process $p_i$ then $A^H_{i + w}$ be the history that comprises of all operations in $L_i$ and all write operations in $H$. A history is PRAM if for each process $p_i$ there is a serialization $S_i$ of $A^H_{i + w}$ that respects all the program orders $\rightarrow_j$. 

A memory is PRAM if it only admits PRAM histories.  Both  PRAM and sequential consistency require serializations that respect program order. PRAM is weaker as each process can perceive a different serialization (writes for different processes may be in different orders and does not contain the read ops of other processes).  Slow memory and processor consistency can be defined in the same way. 

## Causal Memory

Causal memory can be positioned between PRAM and sequential consistency.  Let $H = <L_1, L_2, \ldots, L_n>$. A causality order of operations is determined by program order and *writes-into* order that associates a write-operation with a read operation (except for the initial value). The write-into order is analogous to the order in which message-passing systems relate the sending of messages to its corresponding recipient.  There may be more than one possible writes-into order if there are multiple writes of a value to a location.

A writes-into order $\mapsto$ on $H$ is any relation with the following principles:

* if $o_1 \mapsto o_2$ then there are $x$ and $v$ such that $o_1 = w(x)v$ and $o_2 = r(x)v$. 
* for any operation $o_2$ there is at most one $o_1$ such that $o_1 \mapsto o_2$
* if $o_2 = r(x)v$ for some $x$ and there is no $o_1$ such that $o_1 \mapsto o_2$ then $v = \bot$

A causality order $\rightsquigarrow$ induced by $\mapsto$ for $H$ is a partial order, that is a transitive closure of the union of the history's program order and the order $\mapsto$. Thus $o_1 \rightsquigarrow o_2$ iff:

* $o_1 \rightarrow_i o_2$ for some $p_i$($o_1$ precedes $o_2$ in $L_1$)
* $o_1 \mapsto o_2$ ($o_2$ reads the value written by $o_1$)
* or there is some other operation $o'$ such that $o_1 \rightsquigarrow o' \rightsquigarrow o_2$

If the relation $\rightsquigarrow$ is cyclic then it is not a causality order. If for two operations $o_1 \not \rightsquigarrow o_2$ and $o_2 \not\rightsquigarrow o_1$ then the operations are concurrent with respect to $\rightsquigarrow$. 

A history $H$ is causal if it has a causality order such that for each process $p_i$ there is a serialization $S_i$ of $A^H_{i + w}$ that respects $\rightsquigarrow$.  A memory is causal if it only admits causal histories. It is weaker then sequential consistency as each process can perceive a different serialization.  All causal histories are PRAM but not all PRAM histories are causal. 

## An Implementation of Causal Memory

Each process maintains 4 local data structures:

* A private copy $M$ of the abstract shared causal memory $\mathscr{M}$ 
* A vector clock $t$, a vector of nats, one for each process in the system.
* Two queues: FIFO queue (outqueue) and a priority queue (inqueue). 

The vectors can be compared such that vector $t_1 \leq t_2$ if each of $t_1$'s components is less than or equal to its corresponding component in $t_2$. 

Each queued item includes a vecto rclock value (timestamp). The priority queue is maintained so that the ordering is a leq min priority heap. A process consists of an initialization routine and five basic actions which are locally and atomically executed:

* Read a location, gets $M[x]$

* Write to a location :

  * $p$ increments $t[i]$ 
  * writes $v$ to $M[x]$ 
  * adds the tuple $<i, x,v,t>$ to the outqueue

* Send removes a nonempty prefix from the outqueue and sends it to the other processes

* Recieve adds all the write-tuples in the message received to the inqueue

* Apply updates the process's view of memory:

  ​	Compare the vector clock of $p_i$ to that of top of the inqueue write-tuple. If all of the components of the vector clock are less than their components in the write-tuple (exlcuding the *i*th component which can only be one more) then the update to $M$ is applied and the write-tuple is removed from the inqueue. 

For proof facilitation given an operation $o$ of a process $p$ , the timestamp of $o$ is denoted $ts(o)$ , and is the value of $p_i$'s vector clock immediately after $o$ completes. $H$ is a history of the implementation if $\forall L_i \in H$, $L_i$ is the ordered sequence of read and writes performed by process $p_i$. 

**Lemma** : *Let $H$ be a history of the implementation and let $o_1$ and $o_2$ be two operations such that $o_1 \rightsquigarrow o_2$. Then $ts(o_1) \leq ts(o_2)$. Furthermore, if $o_2$ is a write operation by $p_i$ then $ts(o_1)[i] < ts(o_2)[i]$; thus $ts(o_1) < ts(o_2)$.*

*Proof*: Induction on the structure of the order $\rightsquigarrow$. There are three cases:

* $o_1 \rightarrow_i  o_2$ for some $p_i$. SInce vector clocks are never decremented, $ts(o_1) \leq ts(o_2)$. Furthermore if $o_2$ is a write operation then $p_i$ increments its local component during $o_2$ so $ts(o_1)[i] < ts(o_2)[i]$. 
* $o_1 \mapsto o_2$ implies $o_1$ is a write operation ($w_i(x)v$) whose write tuple has timestamp $ts(o_1)$.  It is clear from the implementation of memory that $p_j$ cannot read $v$ from $x$ before the write is applied to its memory. The process does not apply the write until its own clock is $\geq$ to $ts(o_1)$.  Since no component of the process's timestamp is ever decremented it is still $\geq ts(o_1)$ when it reads $v$ so that $ts(o_1)  \leq ts(o_2)$. 
* There is an operation $o'$ such that $o_1 \rightsquigarrow o' \rightsquigarrow o_2$. By induction it implies that $ts(o_1) \leq ts(o') \leq ts(o_2)$. By the transitivity of $\leq$, the desired result also holds. If $o_2$ is a write by $p$ then $ts(o')[i] < ts(o_2)[i]$ by induction. Since $ts(o_1) \leq ts(o')$ implies $ts(o_1)[i] \leq ts(o')[i]$ we have $ts(o_1)[i] < ts(o_2)[i]$. 

**Lemma**: *Let $H$ be a history of the implementation and suppose that $w$ is a write operation of process $p_i$. Then each process $p_j$ eventually applies $w$ to its memory.*

*Proof*: If $i = j$ then the write is applied immediately. Assume that $i \neq j$. Let $s = ts(w)$. Based on the implementation, once $p_i$ has executed $w$ it is always true that one of the following cases holds for $w$'s write-tuple:

*  in $p_i$'s outqueue.  
*  in transit from $p_i$ to $p_j$
* in $p_j$'s inqueue

Since $p_i$ performs the send operation infinitely often and the outqueue if FIFO, the message will eventually be sent. Furthermore since channels are reliable the message will eventually be received and added to $p_j$'s inqueue. Finally showing that $p_j$ will eventually apply any write-tuple added to the inqueue. 

