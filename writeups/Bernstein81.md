# Concurrency Control in Distributed Database Systems
## Philip A. Bernstein and Nathan Goodman
## Computer Corporation of America

### Gabe Fierro

I do not have much experience in databases, so a comprehensive overview of the
various techniques and how they fall into a set of patterns was very
informative. While the paper focuses on the structure and correctness of
several concurrency control algorithms, the algorithms are evaluated in part by
the amount of coordination they require to accomplish a serialization order.
There are some methods such as Multiversion T/O or using Basic 2PL for rw sync
and TWR for ww sync that minimize the amount of coordination. Multiversion T/O
in particular demonstrates a log-based approach that has optional garbage
collection that seems like other progressive systems we've seen such as
Postgres In the context of progressive systems, there is a collection of
problems we see being solved over and over again, such as garbage collection.
I wonder how informative a similar survey of progressive, append-only systems
would be.

The paper fully admits that it is focused on "the structure and correctness of
synchronization techniques and concurrency control algorithms", and leaves a
couple of dimensions open. Firstly, the network among all of the distributed
TMs and DMs and clients is assumed to be perfectly reliable, with in-order
delivery and without errors on any messages. While implementing these features
over a network is usually fairly straightforward (sequence numbers, etc), I am
curious as to how much complication this adds to the client and server
implementations, which might require additional coordination to provide a
perfect network. This is especially interesting in the context of
synchronization-free timestamp ordering. The paper also states that throughput
and transaction response time need to be evaluated across several dimensions.
Given the context of this class, the dimension of intersite communication is
especially interesting and would provide an interesting avenue of later work to
go through. There are some combinations of 2PC and T/O interfaces for rw/ww
sync that have properties such as writelocks never conflicting with writelocks
that could be leveraged for constructing a progressive system.

### Chenggang Wu

This paper surveys traditional concurrency control methods in both centralized and distributed database management systems. The focus is on the structures and correctness rather than performance optimizations. The ultimate goal of concurrency control is to achieve serializability. Although there are tons of proposed methods, they can be characterized into two basic techniques: two-phase locking and timestamp ordering. The paper offers a pretty good review of basic 2PL, but goes deeper by providing detailed implementation for DDBMSs using ‘prewrite’. Additionally, it introduces some optimizations (primary copy 2PL and voting 2PL) that exploit data redundancy, and methods for deadlock prevention and detection. For timestamp based CC, the paper introduces the implementation in DDBMSs using ‘buffering’ for reads and writes. It also presents multiversion T/O which progressively logs all the read/write timestamps.

Although the algorithms mentioned in the paper guarantee serializability, the cost of both 2PL and T/O is pretty high. In reality, few commercial systems provide such guarantee, and the majority of systems choose to sacrifice consistency for performance. As long as the trade-off between consistency guarantees and performance exists, there is no ‘one-size-fit-all’ solution to all applications. It would be interesting to see how future systems can balance the trade-offs for some targeted application workloads.

### Xinghao Pan

Berstein and Goodman provide a comprehensive review of the contemporary concurrency control methods, which are categorized into two classes of two phase locking (2PL) and timestamp ordering (T/O).
2PL uses mutual exclusion to prevent conflicts from happening, whereas T/O methods restarts transactions that violate the timestamp ordering.
The goal of concurrency control methods is to provide isolation and serializability.
In particular, the T/O methods (especially MVCC and conservative T/O) are most like progressive systems.

One of the key insights (claimed by the authors) is that rw synchronization and ww synchronization can be attain separately by different methods, and then joined together by ensuring a common serialization ordering.
This allows them to mix and match various 2PL and T/O methods together into 47 integrated concurrency control methods, some of which they claim are better than using the same method for both rw and ww synchronization.

In the nearly 35 years since, there is a recognition that serializability is often too expensive to achieve in a distributed environment, and also that many applications do not actually require such strong guarantees.
Some of the excellent work done here may thus no longer be relevant in today's context.

Nevertheless, I think there are a couple of interesting ideas that are worth thinking about.
Firstly, this paper shows that there are different ways that we can implement progressive systems (while providing transactional serializability).
Different strategies may be used to manage conflicts -- restarts and buffering both work by separating the notion of real-time with system history, and we can even integrate mutual exclusion mechanisms.
Secondly, the separation of rw and ww synchronization suggests that we can deal with different types of conflicts in different ways, as long as they are tied together in a coherent manner.
I would speculate that this could also be true at weaker isolation levels.


### Yifan Wu

This paper survey and consolidate what was apparently a myriad collection of algorithms on
concurrency control over DDBMS, and present that to the core the algoirthms are just concerned with
two types of consistencies (rw and ww), with two basic methods: 2PL and timestamps. It also poses the
challenge of evaluating these algorithms as open questions.

It would be valuable to discuss what ideas we could re-use for weaker consistency gurantees, i.e.
the "fundamental trade-offs". For instance, it seems that the write set idea in RAMP might be
related to the buffering ideas.

I also wonder if there are any new hardware developments that make any of the algorithms more
feasible. I might be completely wrong but F1's almost-global atomic clock might be an example where
hardware improved T/O style algorithms performance.


### Johann Schleier-Smith

Before reading this paper I had not realized that the theory of distributed databases had been so well developed as early as 1981. While some of the techniques described in this paper might be considered ad-hoc, they are accompanied by others rooted in considerable theory. The idea of decomposing serializability into rw- and ww-conflicts was new to me, though the notion of using timestamps to aid in serialization was not.

Today's distributed database implementations seem to have changed the role of the transaction manager somewhat. As described in the paper, "TMs issue commands to DMs specifying stored data items to be read or written." When I think of a distributed database today, I think of linked transactional systems. As described here, they use two-phase commit, but instructions received from the TM are not expressed in terms of reads or writes, but rather in terms of higher-level operations. Perhaps there are applications where more of the approach described in this paper is evident, perhaps in distributed filesystems. I would be curious to learn about that. Another thing that I wonder about is whether other decompositions might be possible, or useful, e.g., suppose we look at partitions of the data, then separately resolve serializability on the graphs of each partition, while still agreeing on a total ordering. How would one best choose the partitions so as to support this? I also have the sense that the world has largely given up on serializability, accepting weaker notions of consistency for most applications, sometimes accepting incorrectness, at other times adding compensating engineering. Still, given the strong theoretical motivations for serializability I would like to understand whether it is provably inefficient, whether efficient implementations might be possible, perhaps ones based on complete analysis of a stored program, possibly in conjunction with the data it operates on, as well as what other models can provide useful guarantees to programmers.