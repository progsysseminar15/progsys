# Time, Clocks, and the Ordering of Events in a Distributed System
## Leslie Lamport, 1978

### Erik Krogen
First thought: I don't like his use of a skinny arrow and a thick arrow to mean two different things. On to more relevant thoughts: It makes sense that a clock system with only logical clocks may have strange anomolous behavior, and it was nice to see an example of this. I am impressed with how simple the resulting algorithm is, even in the case of physical clocks, though I cannot say the same for the proof of its correctness. There is, however, what seems to be a pretty high communication cost to maintain this ordering/synchronization, and I would be interested to hear more about how systems that actually implemented this performed. Very interesting from a theoretical perspective, but is it practical?

One question I am left with: If we only ever move clocks forward, won't we inevitably have clock drift to the future? It seems reasonable to assume that this drift will happen very slowly since they should generally remain in sync, but for a system with a very high uptime, won't the clocks eventually no longer agree with what they should be? Also, is it easy in practice to bump the system clock time forward slightly? 


### Xinghao Pan

The major insight of this paper appears to be that "time is relative", in the sense that each observer sees their own time, and only agree on a partial ordering based on messages received.
There is no absolute real-time -- any total ordering that respects the partial ordering is valid.
Furthermore, logical clocks are sufficient for determining time within a closed system.
(Stated differently, "physical" clocks are logical clocks within the closed physical universe.)
The point, though, is that processes need not adhere tightly to a true physical time in order to coordinate.
Since processes can only causally affect one other through messaging, it suffices to align clocks at these agreement points.

There are places in progressive systems where coordination based on a (total) time ordering is useful; for example, in distributed logs, concurrency control, and replication.
In fact, the separation of physical time and (partially ordered) logical time can be, and has been, taken further.
An MVCC system allows reads and writes back in logical time, and imposes ordering between transactions / versions based on causality, which is a finer granularity than processes or even messages.

The complexity of coordination in an MVCC database seems to be pushed from aligning clocks to achieving a distributed agreement on a partial ordering.
Lamport clocks provide a way to acheive tighter alignment of clocks, and could potentially improve efficiency.
Hypothetically, if every transaction occurred atomically and instantaneously with a true physical timestamp, it would be possible to avoid nearly all conflicts.
Of course, this is impossible in practice, but if we were able to declare read / write sets upfront, having tightly aligned clocks could help us achieve a blend of MVCC with conservative T/O concurrency control without ridiculously high latencies.

### Chenggang Wu

This paper discusses how to use time and clock to maintain the ordering of events in a distributed system. In a distributed system, it is difficult to keep track of the total ordering of events because each machine has limited information about the state of other machines, and the time it takes to transmit message from one machine to another is not negligible. In order-sensitive applications, without proper coordination, the system could perform operations out-of order and produce incorrect results. The paper introduces the notion of partial ordering to reason about the “happened before” relationship between two events, and uses partial ordering to construct a valid total ordering of events. Then, it invents a logical clock for each machine and a communication protocol for machines to synchronize their clocks and enforce the partial ordering. However, partial ordering reflected on the logical clock may not represent the real order in which the events occurs, so the paper finally discusses how to offer a strong ordering guarantee using a physical clock, and to which extent physical clocks can drift from each other.

My concern for the communication protocol used for logical clock is that it requires every process to keep broadcasting its activity to every other process, and if there is a network delay or node failure, then the whole system will halt for a very long time. So this solution is very hard to scale. Introducing physical clock has a similar scalability issue because as the number of nodes grows, it becomes more and more difficult to synchronize the clocks among all machines. Maybe for some operations, we can relax the ordering requirements as maintaining the order may not be as important.

### Gabe Fierro

This paper discusses issues of ordering of events in a distributed system,
based around a synchronization of logical clocks. At a high level, Lamport's
simple explanation of partial ordering of logical clocks and his derivation of
a total ordering by sharing communication between processes is a very elegant
idea. The paper does spend time on how to handle "anomalous events", but these
are consequential to the main point of the paper. If processes are going to be
communicating *anyway*, then they might as well communicate their notion of
partial ordering to all other processes. This complete sharing of information
means that there is essentially continual synchronization that guarantees that
all processes execute the same instructions and in the same order (to the
extent that its relevant, barring concurrent events).

The first thing that struck me about the paper is how Lamport's broadcasting of
logical timestamps to strictly increase the logical clocks of other processes
reminds me of the lattices we examined in the Bloom^L paper. This feels like a
very natural comparison to draw, because logical clocks really do implement
"merge" -- clocks can only "grow". Of course, this is drastically simplified in
a reliable network context with in-order delivery and a lack of failing nodes.
Still, sharing monotonic state seems to be another progressive systems pattern.
