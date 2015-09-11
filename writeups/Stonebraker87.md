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
