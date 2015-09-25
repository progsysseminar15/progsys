#Scalable Atoimic Visibility with RAMP Transactions
##Peter Bailis, Alan Fekete, Ali Ghodsi, Joseph M. Hellerstein, Ion Stoica
##UC Berkeley and University of Sydney

###Xinghao Pan
This paper identifies a new isolation level, Read Atomic (RA), which the authors claim matches the requirements of many real-world systems.
RA ensures that either all or none of a transactions' writes are observed by other transactions.
A few algorithms (RAMP-F, RAMP-S, RAMP-H) are presented that achieve RA, while guaranteeing synchronization independence and partition independence.
Synchronization independence implies that no transaction can cause other transactions to block or restart.
Partition independence means that clients never need to contact other partitions that their transaction do not directly reference.
The two properties ensure that RAMP algorithms scale well.

The thing that struck me most was the identification of a ``good enough'' level of isolation that can also be implemented scalably.
In contrast, stronger isolation levels (serializable, snapshot isolation, etc.) require coordination, while weak isolation levels (eventual consistency, read uncommitted, etc.) are insufficient for real-world uses.

The key to doing so, as the authors suggest, seems to be in separating the task of providing atomic visibility from that of mutual exclusion.
This seems to be a particularly relevant observation to this course -- if we have a progressive, multi-version system that keeps all operations (reads, prepares / prewrites, writes, commits), we would be able to separate the problem of mutual exclusion from that of transactional isoltion.
In theory, I think it would be possible to provide any isolation level with the correct type and amount of coordination.

A claim of this paper is that RA is sufficient for many systems.
This is, however, not properly substantiated, and one could imagine other systems that require stronger or weaker isolation levels, or even user-defined conflict rules.
Ideally, we would want to build a system that could implement and support any isolation level; one possible way to achieve this might be to build on a progressive system, and determine the necessary coordination required at database initialization.
