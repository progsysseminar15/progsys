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

