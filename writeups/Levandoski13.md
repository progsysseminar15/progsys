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
