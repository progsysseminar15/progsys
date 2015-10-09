# Paxos Made Simple
## Leslie Lamport, 2001

### Erik Krogen
One of the biggest takeaways for me was the "pipelining" of an indefinite number of phase 1s. My initial thought when reading the paper was that Paxos must be very expensive, but this large optimization allows for what seems to be a very efficient consensus mechanism. I find it very interesting that, in a way, the burden of synchronization was pushed onto the proposers rather than the acceptors (by stating that a proposer must never send a higher-numbered message with the wrong value), which is not at all what I would have expected. Another big takeaway for me was the provability of the algorithm in the face of any number of proposers, but the optimization that only one will be proposing at any given time to prevent conflicts. This gives a good balance of fast execution in the normal case, but resiliency to conflict in all cases.

One question I am left with: Lamport states that Paxos is optimal when doing phase 2 only. Is this really optimal? Has there been any improvements since 2001? Can we not do any better? Are all of our further improvements only to come from relaxed consistency requirements?

### Xinghao Pan

My understanding of the Paxos algorithm from this paper boils down to the following implication chain:
1. an object is installed as version $n$ with value $v$ only if
2. a majority of acceptors accept version $n$ with value $v$, only if
3. a majority of acceptors agree that no lower-numbered version will be accepted AND the proposal is for version $n$ with value $v$ AND any highest-numbered version less than $n$ (if it exists) has value $v$.
The algorithm does not appear straightforward to prove formally, but Lamport introduces a number of invariants that the algorithm maintains, which he then argues are sufficient for correctness.

I have some doubts about the use of a distinguished proposer and learner.
This seems to reduce the problem to that of a centralized server -- the distinguished proposer can bypass the acceptors and attempt to install a value by informing the distinguished learner about it; the installation is completed once the distinguished learner receives said message (and propagates it for replication).
Furthermore, the election of a leader is itself a consensus problem!

The discussion on replicated state machines is almost orthogonal to Paxos.
We could use any consensus algorithm to ensure that the servers always agree on the command and their sequence, and therefore operate as replicated state machines.

With regards to our class, there are some ideas that do appear related to progressive systems.
Paxos uses immutable versions of objects, where versions grow monotonically.
The "promise" provided by acceptors also grows montonically.

Another thing that initially stuck me was that the complexity of consensus was pushed to the proposers (and even to choosing a distinguished proposer), which have to serve as the coordinators between acceptors.
On second thought, however, this is not that uncommon; in particular, transactional databases depend on the transaction manager to perform safe operations while the data managers simply serve reads and writes and maintain locks or versions.
Taken further, the shopping cart example we saw in Bloom pushes the complexity to the client.
This begs the question of what the trade-offs are for placing the coordination at the different agents in the distributed system.
Is there a way to understanding the complexity of coordination at each level?
