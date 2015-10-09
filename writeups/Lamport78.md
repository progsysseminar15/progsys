# Time, Clocks, and the Ordering of Events in a Distributed System
## Leslie Lamport, 1978

### Erik Krogen
First thought: I don't like his use of a skinny arrow and a thick arrow to mean two different things. On to more relevant thoughts: It makes sense that a clock system with only logical clocks may have strange anomolous behavior, and it was nice to see an example of this. I am impressed with how simple the resulting algorithm is, even in the case of physical clocks, though I cannot say the same for the proof of its correctness. There is, however, what seems to be a pretty high communication cost to maintain this ordering/synchronization, and I would be interested to hear more about how systems that actually implemented this performed. Very interesting from a theoretical perspective, but is it practical?

One question I am left with: If we only ever move clocks forward, won't we inevitably have clock drift to the future? It seems reasonable to assume that this drift will happen very slowly since they should generally remain in sync, but for a system with a very high uptime, won't the clocks eventually no longer agree with what they should be? Also, is it easy in practice to bump the system clock time forward slightly? 
