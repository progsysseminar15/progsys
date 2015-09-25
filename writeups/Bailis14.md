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

### Chenggang Wu

This paper presents a new isolation model: Read Atomic isolation. This model ensures that all of none of a transaction’s updates are observed by other transactions. Note that this is a relaxed guarantee compared to the ‘atomicity’ in ACID where it requires all of none of a transaction’s updates are performed. By guaranteeing atomic read, this isolation model is well suited for workloads that require presenting read snapshots that reflect a state between two transaction boundaries, but not within the middle of a transaction (examples include foreign key constraints, secondary indexes, and materialized view maintenance). The model does not provide serializability guarantee, and therefore cannot prevent inconsistency resulted from concurrent updates. The core of RAMP algorithm is that the write transaction contains a timestamp that identifies the transaction and a list of data written in the transaction. In the PREPARE phase, the client sends the meta information to partitions, which then store the data versions. On COMMIT phase, the lastcommit version is updated and the new version becomes visible to read operations. If a read operation interleaves a write transaction and read different versions, it will issue a second round request to get a ‘matched’ version.

One thing to note is that different from the traditional timestamp based CC, the timestamp for RAMP is used to distinguish between different transactions but impose no ordering guarantees to these transactions. This also implies that the model does not guarantee serializability. A clever point in the design is that the two-round write protocol guarantees that all writes in the transaction are present on their partitions, so the read does not have to stall and wait. It would be interesting to see how this new isolation model is adoptable by commercial DBMSs.

### Yifan Wu

RAMP formalizes and improves what a lot of existing large systems practices regarding consistency.
I don't think I fully understand the theory behind this paper so mostly questions.

I was most surprised by how RA is sufficient for most workflows. It would be great to explain how
and why... Especially in the context of the CAP theorem. Why is this versioning so much more
powerful than existing MVCC systems?

I hope that we could dive into the three respective RAMP algoriths and compare them against existing
algorithms mentioned in the survey. In particular the survey mentioned that evaluating these
algorithms are very difficult, and it would be nice to get a sense of progress made in terms of
understanding the bahavior/trade-offs.

I'm also not fully convinced that this is completely linear:
- wouldn't consistency deteriorate (in terms of time/transactions affected)
- # of replicas?
