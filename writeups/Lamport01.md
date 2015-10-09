# Paxos Made Simple
## Leslie Lamport, 2001

### Erik Krogen
One of the biggest takeaways for me was the "pipelining" of an indefinite number of phase 1s. My initial thought when reading the paper was that Paxos must be very expensive, but this large optimization allows for what seems to be a very efficient consensus mechanism. I find it very interesting that, in a way, the burden of synchronization was pushed onto the proposers rather than the acceptors (by stating that a proposer must never send a higher-numbered message with the wrong value), which is not at all what I would have expected. Another big takeaway for me was the provability of the algorithm in the face of any number of proposers, but the optimization that only one will be proposing at any given time to prevent conflicts. This gives a good balance of fast execution in the normal case, but resiliency to conflict in all cases.

One question I am left with: Lamport states that Paxos is optimal when doing phase 2 only. Is this really optimal? Has there been any improvements since 2001? Can we not do any better? Are all of our further improvements only to come from relaxed consistency requirements?
