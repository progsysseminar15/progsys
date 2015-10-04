# Hekaton: SQL Server's Memory-Optimized OLTP Engine
## Diaconu, Freedman, Ismert, Larson, Mittal Stonecipher, Verma, Zwilling
## Microsoft

### Erik Krogen

Hekaton introduces a number of new ideas: a mixture of in-memory and on-disk tables that can be accessed in the same manner, versioning with parallel garbage collection, native code compilation, commit dependencies, and continuous checkpointing. All of these are very cool features that seem to solve their intended problems very well. Commit dependencies make a lot of sense - transactions abort a very small portion of the time, so continuing optimistically under the assumption that a transaction will succeed seems reasonable. The continuous checkpointing also seems very intelligent - the main problem (that I can see) of checkpointing is the necessity to do some expensive operation all at once, but their clever data/delta stream system solves this issue.

This system reminds me a lot of Postgres - versioned records, garbage/archive collection, etc., except that there are a number of features here that should allow it to be more successful, like garbage collection happening continuously and in parallel, as well as the fact that it's working in memory so things should be fast. So this begs the question - if you wanted a fully historical database like the original imagination of Postgres, could this system be easily and modified to achieve that while still being efficient? It seems that it lends itself to this.

### Johann Schleier-Smith

This paper describes in some detail the considerations for a commercial implementation of an in-memory OLTP engine. There are lots of details, including those of integrating with a traditional SQL database, the approach to generating native code, and details of the timestamp-oriented multiversion concurrency control mechanisms. The authors share quite a lot of learnings, which though thoughtful are largely incremental. In some cases, say with respect to code generation, I was surprised to learn that such techniques are not already commonplace.

Whereas other recent in-memory database designs partition the data so that each region is accessed by one CPU only, in Hekaton all CPUs access the entire database. The authors claim that this reduces the need for application partitioning. This is certainly desirable from a user perspective, as administering partitioning can be burdensome and can distort schema from that which represents the problem most naturally.

There are two themes that contribute to the dramatic performance improvements over previous database storage engines: focus on cutting CPU instructions, and focus on eliminating latching or locking. What isn't clear from the paper is whether these are of comparable importance under ordinary operating circumstances. I am left wondering how much one improvement or the other would separately contribute, e.g., suppose they had retained locks or latches but compiled to native code to reduce CPU use. With faster code latches are held for shorter periods of time, so contention goes down as well. How much improvement latch-free data structures add is unclear.

Drawing conclusions from the experimental results is a bit challenging, as usual, because details of hardware and workload make a tremendous impact. The reported throughput, which goes as high as 36,000 transactions per second, is not particularly impressive. VoltDB has published benchmarks approaching 1 million transactions per second. My suspicion is that the system is bottlenecked outside of the transactional engine, perhaps in legacy code for initiating queries or returning results.

### Chenggang Wu
Hekaton is a database engine that targets OLTP workloads. It is fully integrated into the SQL server, and therefore avoids unnecessary cost for the users to switch to a new DBMS. Hekaton indexes are designed specifically for memory-resident data, and it requires users to only put the most performance-critical tables in main memory. The main reason that makes Hekaton performant on querying main-memory data is that all its internal data structures are latch-free, and can therefore provide very high multi-core CPU utilization rate. Moreover, transactions that only access Hekaton tables can be compiled into machine code for further performance improvement. Hekaton uses an optimized version of MVCC for concurrency control and a corresponding garbage collection mechanism to clean the older versions and reclaim storage space.

As non-volatile storage becomes increasingly popular and affordable, it might be useful to think about what would be changed if we build the next version of Hekaton optimized for querying data stored in NVM. As the storage devices become cheaper, it might be feasible to preserve all the old versions of data without doing garbage collection. This may help if the user wants to retrieve a certain version from the past.

### Ethan J. Jackson

Hekaton is an enhancement of Microsoft's SQL server designed for OLTP
workloads.  The basic idea inverts the traditional architecture of databases.
Instead of keeping your database on disk, and using memory as a cache, we keep
our database in memory, and use the disk just for durability.

It was a cool paper which has obviously had quite a large impact in the real
world.  One thing I wondered about is other potential applications of their
transaction system.  One could imagine stripping away the SQL layer above it,
and the disk durability layer below it, and shipping it as a library which
could be built directly into applications.  You would end up with a lockless
datastructure which is fairly general and has a lot of sophisticated features
which have to be hand rolled  today.

### Yifan Wu

Hekaton combines a lot of existing algorithms and tricks to build a very targeted main memory DB
targeted at OLTP workload. It's so dense that there is not much point in summarizing...

So questions:

- It seems that batching multiple log records into one large I/O could interfere with
the granularity of transaction durability. Say if trasaction A, B, C, D were commited and something
went wrong between C and D, then A, B, and C are all lost as well.
- The speed improvements do come with some restrictions, for instance, the natively compiled procedures impose quite a few restrictions (e.g. user-defined
functions) which might be a backwards comptibility issue for existing systems to migrate over.

- I'm not sure about the argument against partitioning since data needs to be split somehow, even with
no itention of partitioning it cannot all live on the same machine.
- Not sure how checkpointing works... I don't see how splitting the updates/inserts and deletes help
  with processing.
- It would be great to discuss concerns in main memory databases in general, especially the
fundamental trade-offs that the systems different against. Prof Pavlo from CMU made an interseting
comment in his [blog
post](http://www.cs.cmu.edu/~pavlo/blog/2014/06/open-problems-in-transaction-processing-part-1.html) about in-memeory database being a "mostly a solved problem from a research
point-of-view".
- In general it would be nice if DB system papers pointed out limitations of the system... (why is
  that not a requirement).

### Xinghao Pan
Hekaton is memory-resident database engine optimized for OLTP workloads.
None of Hekaton's techniques are particularly novel; however, the combination of MVCC with the emphasis on main memory and compiled stored procedures appears to give Hekaton the big win over its competition.

A few takeaways:
- Hekaton is able to provide latch-free transaction isolation semantics by using optimistic MVCC. The only requirement is the ability to perform a CAS on the record End time.
- Also because transaction order is determined by transaction end time (instead of log serialization order) in MVCC, it is possible to have multiple log streams and avoid bottlenecks on the end-of-log access.
- As an unexpected (to me) benefit of residing in volatile main memory, Hekaton never writes dirty data to durable storage, and hence does not use write-ahead logging.
This makes it possible to avoid problems associated with WAL, and to generate only a log record at transaction commit time.
- Because the only data on durable storage is are checkpoints and minimal log data, and dirty data is never written to durable storage, fragmentation becomes less of a problem.
- Another surprising feature of Hekaton is that checkpoints are not snapshots of the database, but rather compressed logs describing insertions and deletions since the last epoch.
Hence, there is no need to pause the database, and the checkpointing process can run in the background in an essentially lock-free manner.


Questions:
- The description of MVCC seems incomplete, in that there appears to be a possibility for deadlocks, unless all reads are enforced before any write.
- Cascading aborts is a potential problem of Hekaton, but the experiments do not seem to stress the system under such scenarios. Would we see performance collapse under cascading aborts?
- One explicit architectural principle of Hekaton is to not partition the database, i.e. threads operate in a shared-memory environment. While this is reasonable for scaling-up, scaling-out in a distributed environment could present a different set of challenges. It would be interesting to see what and how design choices need to be changed to achieve high performance in the cloud.

### Gabe Fierro

Hekaton is an OLTP engine that operates in memory and uses disk for durability.
It is designed for highly concurrent access, avoiding locks and taking a
log-driven approach to applying changes and transactions. This paper also fills
the gap of transaction manager for the bw tree paper (in fact, this uses a bw
tree!). The primary goal of the paper is to improve the speed of their OLTP
engine by reducing the number of CPU instructions required to execute a query
and limiting bottlenecks. Tables contain record versions, which have a begin
and end time for their validity (these timestamps are applied by transactions),
which also helps in speeding up garbage collection, which is usually a source
of bottleneck in log-based systems. Hekaton takes a similar approach to bw
trees for decreasing locks: having processes complete partially finished
actions such as scanning for dead records.

It was not clear to me when reading the paper how checkpoints worked with the backing store.
Are multiple checkpoints generated and saved, and then during crash recovery the whole
system plays back the entire history of transactions? Effort is spent on parallelizing
playback of some time epoch within a checkpoint, but history must still be played back
sequentially unless steps are taken to remove unneeded history.  It would be nice
to see some statistics around crash recovery time, though because this is offline it
is not a huge issue. There is also a question around using the Hekaton approach for SSDs
and eliminating the durability features -- where are the current bottlenecks in Hekaton
and how do durability features factor into this?

### K. Shankari

This paper is a continuation of the earlier work on BW trees that indicates how
the index works in the context of a larger system. It also introduces several
other novel features of the system, such as a storage engine that is agnostic
to the records structure and performs callbacks to the table specific layer in
order to generate hashes and indexes. The paper also highlights seamless
interoperation with an existing SQL server installation, which did not appear
to introduce significant research problems, but which undoubtedly increased
both the engineering complexity and the impact of the work.

In relation to progressive systems, I noticed that they used timestamps for
synchronization, which was acceptable for them since they were not a
distributed system. I also noticed that, similar to the theme from last week,
while we can get away with immutable data, it looks like progressive systems do
need to mutate metadata so that the data can retrieved efficiently. Is this a
general requirement? Is there such a thing as non-mutable metadata?

I also noticed, that contrary to postgres, they did not have an explicit goal
to keep data forever. It would be interesting to see why that was the case, and
what it would take to add that support if they so wished.

### Michael Andersen

This paper presents an in-memory OLTP engine for SQL Server. It gains a bunch
of performance by using good datastructures (like bw-tree) and by compiling
parts of the queries into native code. In general I felt like the paper was
a collection of good ideas that had in part been presented elsewhere. For example,
after reading Postgres, and "Concurrency Control in Distributed Database Systems",
nothing here seemed particularly surprising.
