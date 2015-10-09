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

