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

