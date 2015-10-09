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

### Chenggang Wu

This paper presents a consensus algorithm that ensures all servers in a distributed system to agree upon an operation to perform. Using the implementation idea given in section 3, this enforces all servers in the system to perform the same sequence of operations, which provides a fairly strong consistency guarantee. During the normal operation, the consensus is actually pretty easy to reach because the system elects a single distinguished proposer to propose an operation rather than letting everyone propose different operations. Moreover, the additional latency incurred by the coordination seems relatively small because Phase I of the consensus algorithm can be performed prior to the reception of client-specified commands.

One question is that although the consensus algorithm guarantees all servers in the system to perform the same sequence of operations, it doesn’t seem to imply that the states between any two servers would be the same at every timestep because servers may execute commands at different speed rates. Also, it doesn’t seem that the servers in the section 3’s example are coordinating with each other to synchronize their progress. So does it imply that Paxos consensus algorithm only guarantees eventual consistency (especially for the example in section 3)?


### Gabe Fierro

Lamport states that the Paxos algorithm is quite simple and straightforward,
and falls necessarily out of the set of constraints that we would like to
express for a distributed system agreeing on some value. These constraints
maintain that: an acceptor can accept a proposal numbered $n$ if and only if it
has not responded to a prepare request having a number greater than n (P1a in
the paper) and if a proposal with value $v$ is chosen, then every
higher-numbered proposal that is chosen has value $v$. While in "plain
English", these are really ways to guarantee some sort of ordering in the
system. These constraints basically state that the value $v$ needs to be
consistent for proposals in any majority set, and acceptors can only consider
proposals newer than the last agreed-upon value. Then the paper discusses how
to learn these new values.

As with the clock paper, there is a notion that information can only grow,
which reduces the amount of coordination required by the algorithm because the
limit of what can be considered by an acceptor is constrained by the last
accepted value. I accept that the simpler definition of Paxos does not want
to consider Byzantine failures; I think it is telling that Paxos is used
nearly always for a distributed server consensus, and almost never between
client and server (from the persepctive of an end user). My question is if there
are additional constraints we could place on the learning of a value such that
we could reduce the amount of coordination in Paxos, and use that primitive
to build more complex logic (albeit not in the most straightforward way).

### Ethan J. Jackson
Again, the paper is somewhat fundamental so it's hard to look at it
objectively.  In this review I'm going to discuss the Paxos algorithm
generally, rather than the specific treatment of it in the Paxos made simple
paper.  Personally, my first interaction with Paxos was through Zookeeper and
the like -- off the shelf implementations which constrain the hard consensus
algorithm within a very specific API for distributed locking and leader
election.  What I didn't realize until I read the paper was how general Paxos
is.  The fact that you can use it to build arbitrarily complex state machines
with interesting fault tolerance characteristics.  One could even imagine a
virtual machine built on top of the algorithm.  Of course performance would be
awful.  At any rate, this algorithm is clearly a fundamental contribution of
significant importance which will have impact into the indefinite future.
