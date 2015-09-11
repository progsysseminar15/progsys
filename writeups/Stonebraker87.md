# The Design of the Postgres Storage System
## Michael Stonebraker

### Johann Schleier-Smith

Reading this paper really feels like time travel&mdash;the introduction references to WORM media and makes the assumption that non-volatile main memory will be common, while the SQL dialect bears unfamiliar syntax and offers a little window into how things might have turned out differently. Of course, Postgres has become a highly successful technology, and key trends identified by this early paper regarding the performance of CPUs, the development of multi-processor systems, and the gating nature of disk io all ring true today. This paper contains quite a lot of implementation specifics, discussing in depth the bits used to encode the row format, calculating the space used by index structures, and describing the use of R-trees to make temporal queries efficient. These tidbits not only help make the work concrete, but could be just as useful in an implementation today. For me the main takeaways are in the details, rather than in the theme of the paper.

While Postgres is used widely today, the temporal query capabilities that so influenced the design of its storage system have faded to obscurity. I am curious to better understand why that is. Were efficient implementations actually developed? Was storage too expensive to permit retaining full history? Did developers prefer to express history explicitly in the schema? Did the query language provide a natural way to ask the questions that users cared about (say analyzing changes through time rather than analyzing snapshots)? Was the approach compatible with the separation between the operational data store and the data mart, and with the ETL process linking the two? Did reorganizing the data to star schema or other formats provide better flexibility and performance?

The Postgres design paper contains a very compelling notion: that time should be integral to our idea of a database. Understanding how this motivation played out in practice is of great interest to me.

### Erik Krogen
One of the biggest points made is simply that we can provide the same consistency guarantees as WAL systems with enormously less complexity (compare to ARIES). However, we provide this at the tradeoff of increased storage cost, and a vacuum daemon overhead (which they argue is negligible, but I am not entirely convinced). The overall concept of keeping all data is fascinating, and something we discussed at length during our last meeting, so it was interesting to see it presented in a more formal manner. One thing we hadn't considered was the possibility of moving to archive, since storage these days is so many cheaper/larger than what they were dealing with at the time the paper was written.

I think an open question is whether a system like this (which didn't really take off at the time, and even the authors admitted may not have been the best idea) could work better in today's environment of extremely cheap storage. HDFS is perfectly suited for append-only and has hugely scalable storage. It could be interesting to use local SSDs for fast storage, then HDFS as an archive - HDFS is essentially WORM so many of the techniques discussed here could possibly be useful.

### Ethan J. Jackson
This paper is an excellent example of a system which trades efficiency for
simplicity.  I don't have a deep knowledge (or any knowledge for that matter)
of typical database systems, but even for a novice like me, the Postgres
transaction log makes immediate intuitive sense.  Building the system around
this log, allows them to simplify many parts of the system from
concurrency management, archiving, and crash recovery code. Having chosen the
right abstraction, the rest of the system falls out elegantly.

This paper is so old, I do wonder if many of it's assumptions still hold in the
age of cheap SSDs.  Also they seem to punt on the question of fault tolerance
(understandably as it was likely less of a concern at the time).  That said,
a transaction log doesn't seem particularly difficult to deal with in an
active/backup configuration -- I'm sure the problem has been solved many times
over since the writing of the original paper.

### Chenggang Wu
This paper presents the design of Postgres, a no-overwrite storage system. Perhaps the biggest feature that distinguishes Postgres from other storage system is that the records are append-only and monotonically accumulated over time. Because of this, Postgres is able to store the current state as well as the historical states at any point of time, and there is no need to use write ahead log. This feature makes the recovery almost instantaneous since the system does not have to roll back to retrieve the historical state from the current state. Moreover, incorporating time as an extra field allows users to query certain historical state (or a sequence of states). However, since the amount of data being stored is constantly increasing, sequential scan becomes too costly and therefore Postgres requires clever indexing mechanisms. Due to the hardware constraints 30 years ago, Postgres uses ordinary disks to store the current state and optical disks to store the historical state. The paper introduces in detail the mechanism for vacuuming the magnetic disks, shipping data to the archival disks, and low-level implementation of R-tree indexes for archived records.

Our storage devices, however, have changed dramatically over the 30 years. For example, 3dxpoint allows persistent data storage while providing comparable access speed to DRAM. As these devices become cheaper, we can probably make assumption that all data can fit into NVM in the near future. With that in mind, what would be the changes in system’s architecture and indexing structures for the next-generation no-overwrite DBMS? Given the progressive nature of the system, how can the performance of the system be improved (without sacrificing the integrity) if implemented in monotonic programming languages such as Bloom?


### Yifan Wu
Postgres was a revolutionary DB system that helped popularize SQL, and overcame a lot of the
complexities of achieving the theoretical benefits. The paper comprehensively covered the goals,
designs (archive and index), and performance. In the context of the class, the paper offers
historical perspective on what has changed and what has stayed in the DB evolution in the past
couple decades.

First, it’s clearly a progressive system in the sense that no data is every overwritten. This key
notion makes the recovery mechanism appears simpler than ARIES. Today we see a lot of DB’s
(especially the NoSQL suite) like this to avoid complex concurrency issues. More broadly, this
progressive notion seem to be very versatile, serving different purposes with different use
patterns.

Although archival system is logically very clean and have good crash behavior, but seemed a costly
process. This would not scale with today’s replicas and it wasn’t clear to me if the paper addressed
this issue (distributed DB system and replicas were mentioned on page 4).
Besides the notion of logging history, there are a few other interesting comparisons with today’s DB
world:

- The hardware has clearly evolved, we now used non-volatile memory extensively, and CPU
instructions are becoming more of a resource (not just I/O).

- Its concurrency control back then seemed simple (and don’t need atomic clocks plus Paxos), and
it’s unclear if the Tmin/Tmax is robust against different machine times.

- The use of UNIX’s file system instead of directly managing its own buffer pools, which is in
contrast to the recent research trend to reduce the OS overhead for DB performances.

- Neat tricks like bloom filter is also very effective to deal with memory constraints, which
  still is a relevant technique today. This is because despite bigger and faster memory, there
  is still a memory hierarchy.



### Xinghao Pan

POSTGRES is an early database system that is built on the idea that no data is ever overwritten, by simply treating all updates as insertions.
Concurrency control is achieved by a simple two-phase locking protocol.
Durability / recovery is provided by replicated mirrors of the database, thereby obviating the need for complicated recovery code.
In addition, since every version of every record is never discarded, it is possible to perform historical queries, something that is typically not supported in modern databases.

POSTGRES also has a hierarchical storage system, where older versions of records are ``vacuumed'' onto write-once-read-many storage stystems.
It is important to note that this is possible to implement and perform cheaply because of the no-overwrite nature of the database.
I also find it interesting that the hierarchical storage fits naturally into the modern day model of storage (main memory, SSDs, magnetic disks on HDFS).

The description of concurrency control and recovery seem to be overly simple.
For example, it is not clear how asychronous processes (potentially on different machines) can consistently agree on timestamps, or on transaction IDs without it becoming a bottleneck.
I also wonder if other concurrency control mechanisms, such as MVCC, could be a more natural fit and perform better.
Achieving consistency across database replicas is also a non-trivial problem, since the execution needs to be not only serializable but also deterministic, which I feel was not sufficiently addressed in the paper.
(Perhaps the Calvin work would address this?)
Having multiple mirrors also increases the storage cost which is already significant in a never-overwrite system.

It is also interesting to contrast POSTGRES with Sprite LFS.
In POSTGRES, the logging approach was a solution to simplifying recovery, concurrency control, and archiving;
LFS uses logging to improve write throughput.
POSTGRES is strictly no-overwrite; LFS cleans up overwritten and deleted files.
Both POSTGRES and LFS have some form of hierarchical storage, be it between magnetic disk and optical medium (POSTGRES) or main memory and disk (LFS).
Both also address the issue of fast retrieval of information through some index.

### Gabe Fierro

What was most interesting to me in reading this paper were the assumptions made
about the host operating systems for Postgres; the paper explicitly mentions
that the operations around time management are CPU intensive and are presumed
to be handled by increasing speed of CPUs. Also, Postgres assumes (like Sprite
LFS) that a surplus of main memory will assist in reducing the number of costly
I/O operations for accessing transactions within the log.  Postgres also
amortizes expensive operations by turning them into writes against the log for a record
whenever possible. This means that optimizing the "hot path" around the records benefits
the wide array of features offered by Postgres and ultimately simplifies its design

This construction of a no-overwrite database that provides queries over the
history of transactions for a record requires concurrency control and timestamp
management, which Postgres is able to manage by only updating the time of the
latest transaction when the actual update logically takes place. It isn't clear
reading the paper how this mechanism would scale to a larger, distributed
system. Postgres uses a two-phase locking policy in main memory, which seems to
imply that there would need to be either a distributed lock or a distributed
timestamp ordering mechanism to maintain consistency in the order of
transactions.  I would like to see how other this and other log-based database
systems have handled the question of ordering in a log when distributing a
database. 

### K. Shankari
#### Overview
This paper describes a database system that attempts to solve the problem of
crash recovery management by eliminating it. It does so by never overwriting
data - all updates are turned into inserts. It deals with segmentation by
utilizing the database index to support some form of threading, and also
compacts while writing historical records to offline storage.

Transactions are initialized by advancing a counter to create the XID. If this
needs to be strictly monotonically increasing, it would imply a synchronization
point. Need to modify transaction state (e.g. from "in progress" to
"committed"). Even with storing everything, might want to compress for
performance, e.g. bloom filter. Requires user to specify access patterns -
similar to "hot" and "cold" files, relations need to specify how they will
access archival data.

A key notion is that of validity - each version of a record is valid from the
time that it is written to the time that the next version is written. Should
versioning be exposed or should it be under the covers?

Comparing the compaction strategy with LFS, the primary difference is really in
the B-tree index. 
