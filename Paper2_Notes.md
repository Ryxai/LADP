# Notes

CAP Twelve Years Later: How the Rules Have Changed

The theorem states that any networked shared-data system can at most have two of the following 3 properties: 

* Consistency (up-to-date)
* High-availability 
* Tolerance of network partitions

Opened the mind of designers to trade-offs and wider range of systems. The 2-of-3 model oversimplifies the tensions among properties however. Incredible range of flexibility for handling partitions and recovering from them. Modern CAP should maximize the combinations of consistency and availability tailored to application.

## Why 2 of 3 is misleading

The easiest way to envision CAP is with two nodes on opposite sides of a partition.  Updating of state will cause the other node to become inconsistent. If not then updating can  not be done.  Only when communication occurs can consistency and availability be preserved, forfeiting partition tolerance.  NOSQL focuses on availability first, consistency second.  ACID (atomicity, consistency, isolation and durability) databases focus on consistency. 

The tension in CAP first became apparent in BASE v ACID.  2-o-3 is misleading due to:

1. Partitions are rare, there is little reason to forfeit A or C. 
2. The choice between C and A can occur many times at a very fine granularity and different subsystems can make different choices, choices can change depending on the operation/data involved. 
3. All three properties are more continuous then binary. 

CAP should allow for A and C most of the time when partitions are not occurring, detecting those partitions and explicitly accounting for them is important. 

1. Detect partitions
2. Enter an explicit partition mode, restricting some operations
3. Initiate a recovery process to restore consistency and compensate

## ACID BASE and CAP

ACID is primarily focused on consistency. BASE was created in 1990s to capture trend of high availability.  Make the choice of availability and spectrum explicit: Basically available, soft state, eventual consistency. 

Acid properties when highlighting availability are minorly affected:

* Atomicity: During a partition both sides should be atomic. High level atomic operations actually simplify recovery.
* Consistency: Usually means the transaction preserves all database rules (unique keys).  CAP consistency is a strict subset of ACID consistency, and neither can be maintained across partitions. Recovery is required, maintaining invariants during partitions may be impossible, careful thought required on disallowing/recovery.
* Isolation: If ACID isolation required, it can only operate on one side of the partition. Serializability requires communication in general, fails across partitions. Weaker definitions of correctness are viable across partitions by compensating during recovery.
* Durability: No necessity to lose durability, can avoid it by choosing soft state. During partition recovery is possible to reverse durable operations that violated invariants. 

## Cap-latency Connection

CAP ignores latency in the traditional variation. CAP's major use takes place during a timeout when the program must make a decision concerning the partition: to cancel the operation and decrease availability or proceed with operation and risk inconsistency.  Practically a partition is a time-bound on communication. 

There is no global notion of a partition. If nodes can detect a partition mode. Bounds can be set for timeouts. May make sense to forfeit strong consistency to maintain low latency. 

## Cap Confusion

When systems are entirely unavailable having an offline mode may be important, however it requires recovering from long partitions. Scope of consistency is concerned with the integrity of  data up to a boundary but does not make guarantees outside that boundary i.e.  Paxos and atomic multicast.  Independent self-consistent subsets can make forward progress, may  not be possible to ensure global invariants. 

When considering the possibility of avoiding partition tolerance, (the choice is for consistency and availability) then need to ensure the probability of a partition is lower then all other failure modes, as then choosing consistency or availability is required.  Choosing CA is difficult due to the practicalities, due to latency issues it is also common for forfeit perfect consistency for better performance (across large networks). Maintaining consistency requires a knowledge of the system invariants.  Choosing invariants that need to be restored is prone to error, is the same concurrent updates issue that makes multithreading more difficult then sequential programming.

## Managing Partitions

Challenge is minimizing effect of partition on availability and consistency. Managing partitions explicitly enables this. Includes:

* Detection
* Recovery process
* Plan for all the invariants potentially violated

Has three major steps:

1. Detect partition
2. Enter partition mode limiting operations
3. Initiate partition recovery when restored communication

Quoruming systems are a method of one-sided partition (quorum side proceeds, other does not).  During partition mode there are two options:

1. Limit available operations, reducing availability
2.  Recording extra information about operations to enhance recovery.

Continuing to attempt communication will enable detection of the end of the partition.

## Which Operations Should Proceed?

Limiting operations depends on system invariants. For invariants that need to be maintained during a partition, limiting available operations. Externalized events can be put-off, forfeiting availability in a way the user does not see.  Partitions create difficulty in presenting user interfaces for running but not complete tasks. 

Use of atomic operations allows for vastly easier analysis to high level operations on invariants. Version vectors can be used to track history of operations and capture casual dependencies.  If ordering vectors cannot be done, then operations are concurrent and possibly inconsistent.  Causal consistency is the best outcome if the designer focuses on availability.

## Partition Recovery

During the recovery two major operations need to be completed:

* Making the state on both sides of the partition consistent
* Compensation for mistakes made during partition mode

Simple way is to roll back the system to a consistent state and then play the operations forward in a consistent deterministic order. Sometimes, manual intervention is required for these conflicts however. Limiting operations during partitions to certain operations that can automatically be merged during recovery is one method.  Delaying risky operations is a common methodology.  Using commutative operations is the closest approach to a generalized framework. 

Commutative replicated data types are a class of data structures that provably converge after a partition.  They only allow locally verifiable invariants (makes compensation unnecessary). Using CRDTs for state convergence allows for temporary violation of global invariants, converge after the partition and then execute any compensations.

Compensating transactions is a way of dealing with long-lived transactions. Long-lived transactions deal with a variation of the partition decision: hold locks to keep consistency or expose uncommitted higher concurrency. Compensating transactions break a large transaction into a set of smaller sub-transactions which are committed along the way. If the transaction needs to be undone new orders are issued to remove the sub-transactions. 

Avoiding aborting transactions that use incorrectly committed data. Depends on the net effect of the transaction sequence on the states and output: after compensations does the database end up in the same place it had been if the sub-transactions had not been executed and must include externalized actions. 

Careful management of invariants is an alternative to the tradeoff of consistency and availability. Specifics depend on the service's invariants and operations. 