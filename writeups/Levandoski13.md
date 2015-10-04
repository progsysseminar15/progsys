# The Bw-Tree: A B-tree for New Hardware Platforms
## Levandoski, Lomet, Sengupta
## Microsoft

### Erik Krogen

This paper seems to take a very practical, modern-hardware oriented look at how to implement a very fast Bw-tree index to support a record store. Any sort of latch-free design naturally orients itself away from updating in place as much as possible, and they have come up with some very clever techniques on how to avoid in-place updates. The page mapping table is especially clever in its ability to swap in new pages with a single update, removing the necessity to modify nearby nodes. The split and merge techniques, through their specific delta record types, are also particularly well formed. I especially like the idea of a thread encountering a partially complete update having to finish the update itself - this is an excellent way to avoid any sort of blocking, and they have carefully crafted the data structure such that all the necessary information is already available. The cache performance is a great additional benefit of no in-place updates.

One question I am left with is how exactly garbage collection works - is it something triggered immediately at the end of each epoch? Is there a specific garbage collection daemon? The use of epochs to avoid collection of memory while another thread is using it is clever, and obviously the performance results indicate that this isn't an issue, but I am curious what they are using here. It does seem that each epoch is short enough that the garbage collection should be small on each round.


### Yifan Wu

The researchers zeroed in on a few hardware developments (since the original B-tree): multi-core,
large main memeories, and flash storage. To optimize for those specific trends, they set out to
design algorithms that are latch free, no in-place updates, and more sequential writes (as opposed
to expensive random writes).

For the main memeory layer, the key trick to enable latch-free outcome is a mapping table between
page ID and physical address (a layer of abstraction between logical and physical pointers), whereby they are able to
apply "delta" updates by swapping out the updated page (compare and swap is fast). They have
designed very detailed and specific update-contention protocals.

To reclaim space, they use "epochs" to coordinate, and only reclaims the space when all dependent
threads have exited.

It's very interseting that they make page sizes elastic (and work well with cache!) so that they
could choose a good time to split. This is
definitely an innovation from traditional B-tree designs. I'm not sure what would trigger a split in
this case... The two phase split is another instance of atomic operations to avoid locking.

On flash, they use log structured store, and update with the delta updates. I'm curious what their
recovery performance is.

Questions:
- Since the author claim that the benefit over latch-free skip list is CPU
cache performance, could a similar idea be applied to skip-lists to improve perforamnce. I know some
companies use skip-lists also for security purposes (so it's not as easy to just replace with
bw-tree).
- B-trees are not trivial to implement, this seems orders of magnitude harder, are there
  considerations for the cost of complexity (and ease to intruduce bugs)?

### Chenggang Wu
BW-Tree is a latch-free B-Tree designed for multi-core processing and flash storage. In a latch-free setting, multi-core CPUs won’t get blocked by one another and therefore can reach near 100% utilization. Furthermore, since BW-Tree uses a delta update mechanism instead of update-in-place, it manages to reduce CPU cache invalidation. It is worth noting that BW-Tree uses atomic CAS instruction for updating state changes, and the strategy is used for both data changes and management changes. Also, BW-Tree is designed for flash storage. The characteristics of flash storage is that although it’s fast in random read, sequential read, and sequential write, it can incur a latency penalty for random write. To solve this problem, BW-Tree does log structuring at its storage layer, and therefore manages to avoid frequent random writes.

One question that I have is that will there be an efficient way for the BW-Tree to resolve the conflict raised by two concurrent update operations for the same page. Currently, the paper assumes that the concurrency-control will make sure that no two operations will update the same object simultaneously. Without this guarantee, however, there may be multiple branches of delta updates for a given page. Is it possible to borrow some ideas from MVCC to resolve this issue?

### Gabe Fierro

The paper describes a lock-free and mostly non-blocking B-tree alternative to
be used for constructing databases. The key components are using a layer of
indirection to have the nodes in the tree point to virtual rather than physical
memory locations, and applying changes to the tree as a log of deltas, rather
than experiencing contention on editing a physical page directly. This means
that the bw tree can use atomic operations (CAS) and does not have to
coordinate access between multiple threads.  There are also parallels to be
drawn between the delta log which is regularly compressed and how Postgres
handles logs of changes. Also similar to LFS, the design explicitly assumes an
abundance of main memory -- fetching a page from main memory is one of the few
blocking operations in the design.

The data structure does assume the existance of a concurrenct transaction
manager, and it would be interesting to see how the introduction of that over
this data structure impacts the great performance claimed by this paper. While
fast and non-locking concurrent access is a goal  of the data structure,
updating the same record from multiple threads should probably provide some
sort of ordering or feedback on last-write-wins, which would have to be left to
a higher layer. Also in the case of sequential access, how does a bwtree with
all of its mechanisms compare to more traditional data structures?


### Xinghao Pan
Bw-Tree works by combining a number of techniques:
- delta updates
- log-structured store
- indirection between logical and physical pages

The most important technique is the use of delta updates, which allow for latch-free accesses via compare-and-swap operations.
Delta updates also have small memory footprints, which improves cache hit rates, allows more throughput, and reduces the space that needs to be garbage collected.
Using a log-structured store enables sequential writes to flash, instead of having to do random writes which are much slower.
Mapping of logical pages to physical pages enables changes to be made to the physical pages on every delta update without having to propogate changes through the Bw-tree.
This indirection also allows us to easily relocate a logical page's physical address after a delta update has been installed.
(Note -- the mapping here is similar to the inode map we saw in LFS.)

The paper claims that ACID semantics can be built on top of the Bw-tree, by a contract between the Transactional Component (TC) and the Data Component (DC).
The DC supports WAL of the TC by ensuring all LSNs less than RSSP are durable and consolidated, and all LSNs greater than ESL are not durable.
Recovery is achieved by reading the durable state on flash and replaying operations between RSSP and ESL.
(This seems to partly punt the problem of durability to the higher-level TC, which has to ensure the durability of operations between RSSP and ESL.)

One of the problems of logging that we had discussed in class last week was the difficulty of reclaiming space.
In this paper, this is done via page consolidation (compressing deltas) and garbage collection via epochs.
Garbage pages are only created during consolidation, and so can be easily identified by the thread performing the consolidation.
To ensure the old page is not cleared away prematurely, threads joins the current epoch at the start of its operation to protect all garbage that might be created during and after the epoch.
Garbage is only collected after all threads have left the epoch, which implies that no thread is still dependent on the garbage page.

The above description, however, does not appear to fully address the problem of space fragmentation.
Internal fragmentation is probably mitigated by buffering multiple deltas, but it is unclear from the paper how external fragmentation is managed.
(Is external fragmentation even a problem for flash storage?)

Bw-trees have many of the same innovations of POSTGRES and LFS, except that it explicitly takes advantage of flash storage, and implements latch-free algorithms for structure modification operations.
As the conclusion points out, it would be interesting to try to apply these techniques to other settings.
In particular, I would like to see how well an ACID database built on top on Bw-tree works, since the TC is responsible for maintaining the atomicity and durability of transactions.
Another interesting application would be to implement time-range queries that are supported by POSTGRES.
In the experiments, it was seen that performance degrades as the delta chain length increases to more than 2, so it is not ideal to maintain long delta chains.
On the other hand, it is necessary to keep all deltas around to perform time-range queries.
Achieving a good tradeoff between the two would be an interesting exercise.

### Johann Schleier-Smith

The Bw-tree is designed to take advantage of modern hardware, specifically multi-core processors and flash storage. The approach makes a clever use of both mutable an immutable state. The mapping table points to the most recent version of each page, and allows the Bw-tree to use the atomic compare and swap instructions found in modern processors to push coordination to the hardware layer; the software never waits on other threads. While other progressive data structures might have only one mutable element, say the root of state, introducing mutable indirection between a large number of immutable elements reduces contention, and is a pattern that might find broader applicability.

Bw-tree pages themselves generalize the notion traditionally found in databases and operating systems—rather than representing a contiguous region of memory of fixed size, pages instead represent a variable-size collection of logical records, promised to be located close together. Pages grow with prepended deltas, so pointers to older versions remain valid.

The Bw-tree has some limitations. For one, it is an “atomic record store,” meaning that transactions are limited to single records. Also, the use of mutable state limits the ability to access old versions of the data, so even though this is a progressive data structure, it does not provide snapshot views. One implication of this is that scans can return data from different points in time.

One of the things I like about this paper is the aggressive attempt to make the most hardware-provided concurrency mechanisms. I am curious to understand more about the needs and design considerations that shaped the concurrency primitives available on current CPUs, as well as what might be possible in the future. How could one modify a B+ tree to take advantage of transactional memory hardware? What concurrency primitives would be most helpful for efficient database indexes? I am also curious to understand how some of the limitations of the Bw-tree can be addressed. It seems that Hekaton provides MVCC on top of it, but I don't quite understand how this works. I'm curious how one might extend it to provide consistent views of data without sacrificing high throughput and low latency on updates. Lastly, I am wondering whether this is a brittle implementation. In particular, the Bw-tree claims to avoid cache invalidations by writing updates to new memory locations, but to ensure that other CPUs read the new values we need to be certain that they have not cached the old locations. Perhaps the Bw-tree makes use of memory fencing to achieve stronger guarantees, but as described this “not in cache” promise seems tricky to enforce. I would like to understand what tacit assumptions about the nature of the hardware and compiler the Bw-tree may rely upon.

### K. Shankari

This was an interesting paper that tied fairly low level concepts, like current
trends in Computer Architecture, to fairly high level systems design. As
somebody who has struggled with tree locking in a prior life, I thought that it
was very cool that they were able to construct a lock-free system that could
update trees efficiently.

Nit: As somebody who is not a database person, I found this paper a little hard
to read - it looked like it required a fair amount of database knowledge. In
particular, the use of latches to mean implicitly mean database indexes in a
hardware oriented paper sort of annoyed me. While designing an circuit, a latch
is a flipflop that is used to latch or persist the state of a transient input.

### Michael Andersen

I really enjoyed this paper. It focuses on a tree datastructure that allows large mutations using
only very small locks, so small that they are implemented in hardware as Compare And Swap
instructions. This is a cunning idea, and I think I will go back and look at latch-using data
structures and see if I can borrow some of these techniques.

I may have missed it, but I am not sure how the mapping table does not pose a problem for
large bw-trees. Presumably it is difficult to have a map for all of the underlying pages,
and keep it consistent without a latch? Unless the size of the map is known in advance, but
generally that is not the case for a database that is growing.
