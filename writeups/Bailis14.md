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

### Gabe Fierro

This paper presents a new isolation model and algorithms for implementing Read
Atomic (RA) isolation over a distributed database, in which either all or none
of each transactions' updates are visible to other transactions. The RAMP
transaction algorithms presented guarantee gommit dispite partial failures and
minimized communication between servers. These two situations address the most
expensive part of computing in distribued systems -- synchronization -- and
explore the weaker guarantee of read atomicity and how it can be achieved with
minimum synchronization between copies of data value X across several servers.
The paper presents 3 methods involving multi-versioning to prevent writers
from blocking each other, and we see the traditional space/speed tradeoff
with the amount of metadata associated with a transaction increasing as the
number of round-trip synchronization messages decrease and vice versa.

I am unfamiliar with the family of atomicity guarantees and how they map onto
possible workloads. the paper mentions that atomic visibility is required for
secondary indexing, foreign key constraint enforcement and materialized view
maintenance, but does not mention to what degree these tasks are actually
limited by cross-machine synchronization. It may very well be that this is
common knowledge to the db community, but I wonder to what degree such
operations are the actual bottleneck in some db applications. I do like the
idea that there are simpler guarantees that involve less coordination that
produce the desired results. Given the pattern we have been seeing with other
distributed, append-only systems, I wonder what effect the proposed garbage
collection (as an optimization) would have on the implementation complexity and
performance of the system.


### Johann Schleier-Smith

The atomic visibility model of RAMP transactions is to me and appealing one. It speaks to the needs of many modern applications, and nicely classified the trade-offs faced by consistency models. There is a simple definition for atomicity: fractured reads cannot occur. This says nothing about out-of-order reads, and it makes no provision for synchronization between operations. The guarantees weaker than that of snapshot isolation, or of serializability, but is allows RAMP to provide "synchronization independence," meaning that all transactions are guaranteed to succeed, absent hardware failures.

There are a number of interesting trade-offs discussed in the paper. The three transaction models trade-off between replicating metadata across partitions, which uses space, and additional rounds of communication, which increases latency on reads. The authors also choose a model in which readers never block on writers. It is probably possible to provide faster writes, as well as more up-to-date-reads  if readers wait until in-flight write transactions finish.

One of the proposed applications of the RAMP approach that I find particularly compelling is global secondary indexing. I understand the importance of atomicity across partitions, however, questions about the implementation remain in my mind. Indexes can involve complex data structures, and so maintaining versions with different timestamps and providing garbage collection over these structures could pose some challenges. I also am curious how RAMP might be used, or extended, to allow further progressive systems capabilities. For example, how might one incorporate support for merge operators to allow scalable implementation of a range of monotonic programs.

### Erik Krogen

Previous to reading this, I wasn’t really aware of the relaxed consistency models available, so seeing some presented was pretty interesting. The techniques employed here seem to be very clever. I especially appreciate the development of three different techniques, all of which have merit in different ways, allowing the tuning of a database based on workload (e.g. a workload where transactions are always short may want to use RAMP-F). One thing I noted is that RAMP-H, if used with decent-sized

One question raised here is what use cases RA semantics are sufficient for. The paper mentions a few, and they do seem to be pretty broad, but I am not familiar enough with workloads to know if this is really something that can be widely applied - though certainly it’s an improvement over not having RA semantics. One thing I found interesting is that, under the assumption of very short transactions, RAMP-F actually holds less metadata than RAMP-H. I wonder if RAMP-H Bloom filter size could be dynamic based on the size of transactions?

### Michael Andersen

The isolation model presented in this paper is an interesting one. I think a stronger case for its relevance is made in the various slides that exist for presentations of the paper. It is clear that it does not solve all problems, but there are certainly many problems that it does solve. The issue of inconsistent secondary indices is a good one, as it is easy to follow.

The mechanism takes a few minutes to fully understand, but once it is understood the simplicity of it is very appealing. Something that looks a little bit like 2PC ensures that all servers have the data that will be comitted before any of that data becomes visible. If any query accidentally sees some data that has become partially visible, it can use another RTT per server to get atomic transaction level consistency by asking for the version of the data that is not yet visible, but is guaranteed to exist. The Small, Fast and Hybrid algorithms are not particularly significant, they are just optimisations in case people are upset about having state proportional to the commit size.

Overall I really enjoyed this paper. Seems like a great idea.
