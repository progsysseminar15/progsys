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
