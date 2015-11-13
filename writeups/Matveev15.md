# Read-Log-Update: A Lightweight Synchronization Mechanism for concurrent Programming
## Alexander Matveev, Nir Shavit, Pascal Felber, Patrick Marlier

### Gabe Fierro
RLU is an updated take on the read-copy-update mechanism (essentially copy-on-write with compare-and-swap, if I understand)
for synchronizting maniuplation of a data structure across multiple writers and readers. RCU has the limitation that it requires
that writers operate on some external serialization order, probably imposed by locks. RLU can allow concurrent writers.
The basic process of RLU involves a global logical clock that serves as a "version" marker for copies of the data structure.
When a write occurs, all current readers are divided into two groups: those that started before the write and those that started
after. All "before" readers just read the earlier version of the data structure, and all "later" writers read the updated one.
Multiple writers can be supported because the global clock serves as a log ordering. Various optimizations such as
fine-grained locking on data structure components or "batching" writes can help speed stuff up.

The idea here feels similar to the Time-Ordered log-based multiversion concurrency control, in that the explicit
ordering of readers and writers granted by a global clock (which can just be the order of arrival) gives a nice
way to handle RW synchronization. We see some of the same progressive system patterns here: logs require more
space and therefore require garbage collection (here it seems in-line w/ the data structure usage, rather than
a separate process). What is interesting is that they do discover that making copies of their data structure
is the slow part and limits the efficacy of the data structure in certain situations. I'm curious about the effect
of data structure size on the RW throughput, and also how well this would perform as an on-disk data structure
(or perhaps as an interface to one).

### Yifan Wu

RLU addresses key issues of RCU by making it possible to have reads with multple writes going on,
and semi-automating the process. It avoid RCU's limit of only one update at a time with its
per-thread object-level write-log. The update is amortized by lazy evaluation of conflict.

Questions:
- Would be great to give on overview of the "design space" of concurrency on the thread level; it seems related to
the database world of concurrency (MVCC) but with a completely new set of terms... It's unclear to me what
tradeoff is faced by RLU.
- I'm not sure how RCUs are being used by programmers --- why does it have to be complicated?
- The performance gain seems very marginal (though the code does look simpler) --- is this a
  fundamental improvement or just a library that makes writing better code easier?
- I don't see how this paper is related to SEDA


### Johann Schleier-Smith

The well-established RCU mechanism provides a concurrency primitive allowing non-blocking reads on diverse data structures. It accomplished this at some cost to writes, which must wait until previously started readers have finished before completing their work. Despite its ubiquity, RCU can pose challenges for programmers, requiring them to create copies of modified objects and to manage detailed pointer updates. RLU relieves the programmer of these burdens by implementing something akin to software transactional memory.

I haven't yet fully understood how updates are reflected in the write log, how the *get_actual* and *get_copy* operations are implemented, but I understand that they encapsulate the logic for maintaining different versions of the state. The *rlu_synchronize* operation then finds a safe time to apply these changes, which is after any previously started reads have completed. RLU uses a global clock, incremented when entering a lock, to give order to operations.

RLU seems to be strictly better than RCU. Perhaps it is more complicated, but it makes life easier for programmers and exhibits excellent performance. I am curious to understand its limitations. It provides limited support for managing conflicts between writers, and I wonder how this shortcoming might be addressed, whether for example one could RLU to allow concurrent modifications to different parts of a tree data structure. Such extensions could allow the approach to apply to more problems, or potentially even to distributed systems where latency becomes an important factor.


### Erik Krogen

I've heard of the ubiquity of RCU, especially within the Linux kernel, and was somewhat surprised to hear that it's essentially only been used for singly-linked lists (though now that I understand the algorithm more, I can see why that's the case). Truly the beauty of RCU is in it's simplicity; once I saw it explained it was fairly obvious. RLU seems to lose a little bit of that, but seems to make sense as an evolution (and one that's been a long time coming!). The ability to do multiple different updates and have them appear simultaneously is an especially important contribution, in my opinion. The ease of use also seems to be very important - if something is too hard to use (like RCU), it will only appear in very specific environments (i.e., the Linux kernel), whereas RLU may be much more available to the general application developer.

One question - I'm not really sure why RCU is so hard to use. I buy that it is, but it seems relatively straightforward, so I'm not sure I understand the difficulty of programming using it. Perhaps it's just because the guarantees given are relatively weak, and all you can use is a singly-linked list (which is relatively restrictive)? 
