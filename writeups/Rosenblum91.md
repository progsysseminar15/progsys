# The Design and Implementation of a Log-Structured File System
## Mendel Rosenblum and John K. Ousterhout

### Johann Schleier-Smith
Perhaps the most interesting content in this work, and that which remains relevant today, involves the approach to reclaiming space. A log-structured filesystem writes to large contiguous regions, termed "segments," and in the presence of updates and deletes requires a cleaning policy in order to consolidate live data and create free segments. The authors use simple heuristics throughout but their first attempts perform surprisingly poorly. The critical insight is a cost-benefit calculation which pushes more recently modified segments to be reclaimed later. In the authors' words, "stability can be estimated by the age of the data," and they find that consolidating stable data produces full segments that are likely to remain stable. I enjoyed this paper's emphasis on simple modeling and straightforward rules in design of practical system.

One potential capability of log-structured filesystems that the authors do not explore is the potential to go back in time, perhaps to recover accidentally lost data, or for audit purposes. It this objective likely conflicts with the performance optimizations described in the paper, though another set of techniques, likely including compression, could be called upon. I am also interested in understanding more about how this work relates to garbage collectors for programming languages, especially because modern flash sits in-between disks and memory in regard to random access speed.

### Erik Krogen
A big highlight was the segment cleaning mechanism, not surprising since a very large part of the paper was devoted to this process. Coming into the paper I was expecting reading the data to be the hard part, but it became quickly clear that a bit of indexing solved this problem without any issues and fairly low complexity. I've learned in previous classes that file systems have very strong bimodal distributions in two areas: a ton of very small files and a few large files that end up taking up most of the space; and a few files that are accessed very frequently and lots of files that are almost never accessed. Their usage study perfectly mirrors this, and it's amazing how well their cost-benefit model manages to capture this - the bimodal segment distribution plot they show is inspiring in how perfect it is.

One big question that remains for me: why didn't this ever take off? I'm not familiar with any file system that uses this approach that is used today, although COW systems like ZFS have some of the same ideas, though AFAIK without the sequential write optimization (which won't make nearly as much of a difference now that most things are moving to SSD). I will say that I always view a system with a garbage collection-type daemon with some healthy skepticism, but the results they reported are very promising. I wonder if there's a way to do something like this without the requirement for a background daemon--we've already seen it in both POSTGRES and Sprite LFS, so it seems to be a common theme. Could a file system like this be really useful on HDFS data nodes? HDFS has an append-only model, so it seems that it would be a perfect fit.

### Ethan J. Jackson
This is a really cool paper.  They realized that traditional file systems were
optimizing for the wrong thing, simple addressing for the CPU, when really the
most expensive operations, and therefore most important to avoid, are disk
seeks.  As a result of this realization, the built a file system in which all
writes are appends to a giant circular log consuming the entire disk.  This
reduces the number of seeks per write to effectively 0, greatly increasing
performance, at the cost of marginally more complicated reads.

I suppose I have two major questions about this paper.  Given that it seems to
be an obvious win, why did it not, in fact, win?  My (quite limited)
understanding is that while modern file systems do take advantage of journals,
they don't take it to quite the extreme approach proposed in this paper.  My
second question is, does any of this matter on SSDs?  Seeks are free, so the
performance benefit isn't there.  It's not clear to me if this transaction log
is fundamentally better given this fact.

### Chenggang Wu
The paper presents log-structured file system where all the modifications are written sequentially onto the disk in a log-like structure. The authors explained how to write new files and change existing files without modifying existing blocks using inode and imap. More importantly, they proposed a storage reclamation mechanism that is somewhat similar to the main memory garbage collector. The major difference is that for disk storage, since we want to minimize the latency overhead produced by seeks, we need to copy sparse live blocks to a new region to reduce the fragmentation. This will make more continuous free blocks available for a sequential write.

In the future, however, the use of NVM will probably be more and more prevalent. NVM provides persistent storage as disk does, but it is also byte-addressable and supports random-access with DRAM-like performance. Therefore, sequential writes are no-longer necessary and we only need to find available memory space. Based on the same reason, the copying during the storage reclamation also seems unnecessary. It would be interesting to know what additional complexities may incur using NVM with this storage mechanism.

### Yifan Wu

LFS took advantage of bigger disk bandwidth and larger main memories (buffering updates), and deals
with FFS’ latency due to seeks and synchronous writes to metadata. LFS appends to a single log
(disk), that way avoiding seeking (since it’s updated to the end of the log). The major challenges
with this method is memory fragmentation and locating the most recent data. Finding data is solved
by inode maps and segment summary (yay levels of indirection!), which conveniently is cached in
memory.

One big problem is cleaning, which has a high overhead. Another reason for why journaling file
system may have been more popular could be the simplicity of JFS in general. Though LFS had a big
memory advantage which should more important back then.

In terms of the relevance for today, though the spread of NVM makes seek time more irrelevant, the
side benefit of appending to the end is helpful for coordination/concurrency control in a distributed setting.

As for my question, it would be great to talk about what people truly cared about when choosing FSs.

### Xinghao Pan

The Sprite LFS is a log-structured file system that places the log front and center as the truth of the system.
It was developed in a technological setting where
- Processors are getting faster at exponential rate,
- Disk is getting cheaper but speed is not increasing as quickly as processors,
- Main memory is getting larger, which helps to serve more read requests without disk access, and to buffer more writes before going to disk.
- Workload is dominated by many accesses to small files.

Under this setting, the authors claim that LFS has the following advantages:
- Write throughput is improved by sequential access, thus reducing seeks,
- Files are cached in memory and most reads are satisfied by the memory,
- Simpler recovery process via checkpoint and roll-forward.

There are a couple of difficulties in implementing a log-structured FS that the authors acknowledge.
Firstly, files need to be quickly located and read from disk without needing to scan the entire log.
This is largely dealt with by keeping the inode indexing of Unix FFS, and by keeping the inode map in memory so as to reduce seeks.
Secondly, fragmentation may become a problem as files are deleted and overwritten.
LFS handles this through a combination of threading, compacting, and segment cleaning.

The technological setting today has changed since the 90's, which should prompt a re-think of the log-structured FS approach.
Processor clock speeds are no longer increasing exponentially, but we are better able to take advantage of parallelism through multicore technology.
Both magnetic disks and SSDs are now cheaper and highly elastic.
Recent developments, e.g. Spark, have exploited main memory on distributed machines.
How do these developments change the way we would implement LFS today?
(Should we increase the write buffer size, and effectively serve all reads and write from the main memory, and archiving to HDFS infrequently?)
Are the advantages and disadvantages of LFS still valid today?

In particular, the paper does not address how LFS should manage multiple processors reading / writing.
Related to this issue is how we can support transactional semantics (ACID) on top of LFS

An advantage of LFS that still holds today, I think, is that recovery is simplified.
This is something that we see in Spark too -- as long as the lineage (log of actions) is retained, all 

Most importantly, I think we need to understand LFS in its historical context, including whether it was successful, and why?

### Gabe Fierro

One of the key takeaways of this paper is the authors' use of "append-only"
indirection methods to avoid synchronizing state between multiple parts of the
file system as well to minimize access between the multiple tables needed to
manage file metadata. There are several places where this concept turns up:
* using version numbers inside the inode map to avoid having to access an inode
  (or worse an indirection block) to determine whether or not a file block is
  live
* the use of an inode map as a layer of indirection to find inodes. This does
  away with the simple FFS method of having a static inode location -- which
  simplifies implementation but adds disk accesses -- and by being written to
  the log, it stays in line with the idea of sequential writes. It does not
  require the disk to "go back" and edit a single entry in a known table
  lcation, which increases the number of seeks

The authors use the benefit of large main memory to subsidize the cost of the
tables they have to get around the usual sequential seek that is necessary for
reading logs. While the authors offhandedly mention NVRAM as a way to provide
crash protection, the increasing ubiquity and decreasing cost of SSDs raises
the question of a "triple-tier" log-structured file system. Distributed file
systems such as Ceph and "fusion drives" (combinations of SSD and spinning
metal) are already used to keep hot data in fast read/write locations. I wonder
if the authors' insights regarding treating hot/cold segments differently could
be extended to put hot segments in SSDs.

### K. Shankari
#### Overview
This paper describes a file system structure that is optimized to use the disk
efficiently for small writes. It does so by always appending writes to the end
of a "log". It uses a combination of threading (index across segments of the
file) and compaction (make the segments contiguous) in order to deal with
segmented files. One would expect that this would work for large sequential
reads too, specially after the compaction has run, but there are already
techniques for sequential accesses to large files.

It would be interesting to explore what would happen if you never deleted data
from the disk. Would that eliminate the need for threading and compaction? It
does not appear to be the case because you might still want to append to
existing files, which would cause the file to be segmented and require
threading. It might even make it harder, because if there are no holes, we
cannot compact the data to be contiguous again.

A naive but wildly inefficient technique to overcome that would be to write a
new copy of the entire file every time that it was appended. But even with
assuming infinite, no-cost storage, that would force you to read the file
before appending to it, which will make appends to large files inefficient. The
other option is to have a very efficient index and just always thread the
files. This is the equivalent of logging by timestamp in a database. The index
would need to be distributed in order to support distributing the load
physically. That must be what the current sharded databases do. I wonder how
efficient they are with small read/writes versus large ones. Theoretically, the
SQL databases avoid this by segmenting the data even further so that all
accesses are small ones (one row at a time), so that the workload is more
predictable.

But that might not be true any more with support for CLOBs and with NoSQL
databases. It would be interesting to look up the performance metrics of those
databases.
