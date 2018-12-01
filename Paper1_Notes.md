# Notes

Brewers Conjecture and the Feasibility of Consistent, Available and
Partition-Tolerant Web Services

## Intro

Three major pillars of a web application:

- Consistency
- Availability
- Partition tolerance
  Conjecture: it is impossible for a service to have all three.

## Formal Model

Atomic/linearizable consistency there is a total order on all operations
were completed at a single instant. Any read option that returns after a
write option must return the value committed after the write.

A continuously available distributed system requires every request made by
a non-failing node must receive a response. That is any algorithm used by
the service must terminate, allows unbounded computation. When paired with
partition tolerance: results in even when severe network failure occurs,
every request must terminate.

The network must be allowed to lose arbitrarily many messages sent from
one node to another. A partitioned network loses messages in other
partitions, message loss can be modeled as a temporary partition. Atomicity
implies every response will be atomic, although messages may not be delivered.
Availability implies a request from a client must result in a response even
if there is message loss. Similar to wait-free termination, no set of failures
less then total failures can allow the system to respond incorrectly.

## Asynchronous Networks

In asynchronous models there is no clock and nodes must make decisions based on
messages received and local computation

**Theorem** _It is impossible in the asynchronous network model to implement a_
_read/write data object that guarantees the following properties_

- Availability
- Atomic consistency

_in all fair executions_.

Proof: Assume an algorithm A that meets the three criteria. Assume that the
network contains at least two nodes, thus it can be divided into two disjoint
sets ${ G_1, G_2}$. Assume all messages between $G_1$ and $G_2$ are lost. If a
write occurs in $G_1$ and a read occurs in $G_2$ then the read op cannot return
the value from the earlier write op. That is let $a_1$ be a prefix of execution
of $A$ in which a single write of a value not equal to $v_0$ occurs in $G_1$,
ending with the write op. Assume no other client requests occur. Assume no
messages are fully transmitted between $G_1$ and $G_2$. Let $a_2$ be the prefix
of execution in which a single read occurs in $G_2$, with no other client
requests occurring and ending with the read operation. No messages are
transmitted fully between $G_1$ and $G_2$ during this time. Let $a$ be a
execution starting with $a_1$ and continuing with $a_2$. For $G_2$, $a$ is
indistinguishable from $a_2$ while for $G_1$, $a$ is indistinguishable from
$a_1$. Since $a_1$ does not include any requests to $G_2$ from $G_1$ then in
$a$ the read returns the original value of $v_0$. Thus we have a contradiction
of atomicity.

** Corollary ** _It is impossible in the asynchronous network model to_
_implement a read/write data objects that guarantees the following properties:_

- Availability in all fair executions
- Atomic consistency in fair executions in which no messages are lost

Proof: In the asynchronous model, the algorithm has no way of determining if a
message has been lost or arbitrarily delayed in the transmission channel.
If there existed an algorithm that guaranteed atomic consistency in executions
in which no messages are lost, there would exist an algorithm that guaranteed
atomic consistency in all executions which would violate the aforementioned
theorem. Formally, assume that there exists an algorithm A that always
terminates and guarantees atomic consistency in fair executions in which all
messages are delivered. Theorem 1 implies A does not make that guarantee, so
there exists some $a$ in which the response of A is not atomic. Let $a'$ be
the prefix of $a$ ending with an invalid response, then extend $a'$ to a fair
execution $a''$ in which all messages are delivered. The execution $a''$ is
fair but not atomic, thus A does not exist.

### Solutions in Async

If availability can be reduced then easy to achieve atomic + partition
tolerance. Trivial version ignores all requests failing those requirements.
Liveness can be defined more strongly: if all messages are delivered then the
system is available and all operations terminate. A centralized algorithm
works: single node responsible for the value, all messages forwarded it to it,
when acknowledgement received sends a response to the requester.

### Atomic, available

If there is no partition then atomic, available. Centralized algorithm meets
these requirements. Systems that run on intranets and LANs are of this type.

### Available, partition tolerant

If atomic consistency is not required then the service can return
$v_0$ to every request. A weakened definition of consistency can be provided
(web caches).

## Partially synchronous

Most networks are not purely asynchronous, if each node has a clock, it is
possible to build a more powerful service. All clocks are assumed to increase
at the same rate and are not synchronized, they may display different values
at different times, act as timers. Local timers can be used to schedule
actions, and to assume a message is delivered within a given time $t*{msg}$ or
is lost. Every note processes a message within a given known time $t*{local}$
and local processing takes 0 time. Can be formalized as special case of general
timed automata.

**Theorem** _It is impossible in the partially synchronous network model to_
_implement a read/write data object that guarantees the following properties:_

- Availability
- Atomic consistency

_in all executions_.

Proof. Similar to earlier theorem, divide network into two components
${G_1, G_2}$ and construct an execution in which a write happens in one
component and a read in the other. Formally construct execution $a_1$ as in
earlier theorem and the second execution $a_2'$ is constructed with a long time
interval during which no requests occur, must be as long as the duration of
$a_1$. Then append $a_2'$ to the events of $a_2$ as defined in the earlier
theorem. Construct $a$ by superimposing the executions $a_1$ and $a_2'$. The
time interval ensures the write completes. The read request however returns the
initial value not the one sent by the write request.

The analogue of the earlier corollary does not hold. The Corollary depends on
nodes being unaware when a message is lost. Some algorithms will return atomic
data when all messages are delivered, and only will return inconsistent data
when messages are lost. In the centralized alg, read requests are sent and if
received, the node delivers the requested data or an ack. If no message is
received by $2 * t*{msg} + t*{local}$ then the node concludes the message was
lost. The client is sent a response : ack or data. Atomic consistency can be
violated.

### Weaker Consistency Conditions

Important to specify in executions when messages are lost. Consistency
guarantee will require availability and atomic consistency when no messages
are lost, thus impossible to guarantee in the async model as a result of the
earlier corollary. If messages are delivered, then some notion of atomicity is
restored eventually.

A partial order of the read/write ops are done and that if one op begins after
another, the former does not precede the former operation in the partial order.
We use delayed t consistency that defines a partial order on the operations
with the caveat that an operation does not precede another if there was an
interval between the operations that all messages were delivered.

We use the following definition of a delayed-t consistent read-write object
$\alpha$.

1.  $P$ is a partial order that orders all write operations and orders all 
    read operations with respect to the write operations.
2.  The value returned by every read operation is exactly the one
    written by the previous write operation in $P$  (or the initial value,
    if there is no such previous write operation $P$).
3.   The order in $P$ is consistent with the order of read and write 
      requests submitted at each node.
4.   (Atomicity) If all messages in the execution are delivered, and 
      an operation $\theta$ completes before an operation $\phi$ begins, then
      $\phi$ does not precede $\theta$ in the partial order $P$.
5.   (Weakly Consistent) Assume there exists an interval of time longer then $t$ in which no messages are lost. Further, assume an operation $\theta$, completes before the interval begins and another operation $\phi$, begins after the interval ends. Then $\phi$ does not precede $\theta$ in the partial order $P$.

This provides for a time limit on how long the inconsistency can continue, up to when the partition heals (ie all messages are delivered). A variant of the centralized node is delayed-t consistent. Consider a centralized node $C$ and a secondary node $A$. The algorithm works as follows:

* A read at node $A$:
  * $A$ sends a request to $C$ for the most recent value.
  * If $A$ receives a response from $C$, save the value and send it to the client.
  *  If $A$ concludes the message was lost (timeout) then return the value with the highest sequence number received from $C$, or the initial value if no other exists.
* A write at $A$:
  *  $A$ sends a message to $C$ with the new value.
  * If $A$ receives an acknowledgement from $C$, then $A$ sends and an acknowledgement to the client, and stops.
  * If $A$ concludes a message was lost (timeout), then $A$ sends an acknowledgement to the client. 
  * If $A$ has not received an acknowledgement from $C$, then $A$ sends a message to $C$ with the new value.
  * If $A$ concludes the message was lost (timeout), $A$ repeats step d within $t - 4 * t_{timeout}$ seconds.
* New value is received at $C$:
  * $C$ increments the sequence number by 1. 
  * $C$ sends out the new value and the sequence number to every node.
  * If $C$ concludes the message was lost (timeout) then $C$ resends the value and sequence number to the missing node within time $t - 2*t_{timeout}$ seconds. 
  * Repeat step c until every node has acknowledged the value.

We then have the proof that this algorithm is delayed-t consistent:

The ordering of the write operations is the order the centralized node uses to keep track of client writes, using a sequence number. Each read is sequenced after the write operation, whose value it returns. If all messages are delivered, then the  algorithm is atomic. The central node ensures the partial order is respected. If an operation has completed then the centralized node already has been notified and therefore no operation begins later will precede the completed operation in $P$.  If some messages are lost then the executions may not be atomic. Assuming a write occurs at some $A_w$ then assume an interval when all messages are delivered, then all nodes will have been notified of the updated value and any operation will not precede the write operation. Assume that a read operation occurs at $B_r$ after which a time interval passes where all messages are delivered. $B_r$ must have received its value from $C$ or from an earlier write operation at $B_r$. In the former case, by the end of the interval $C$ will have ensured that every node has received at least the latest value sent to $B_r$. In the latter case $B_r$ will have send the value to $C$ and $C$ will have forwarded it to every other node. Therefore no later operation after this interval will precede the read operation.