# The Bw-Tree: A B-tree for New Hardware Platforms
## Levandoski, Lomet, Sengupta
## Microsoft

### Erik Krogen

This paper seems to take a very practical, modern-hardware oriented look at how to implement a very fast Bw-tree index to support a record store. Any sort of latch-free design naturally orients itself away from updating in place as much as possible, and they have come up with some very clever techniques on how to avoid in-place updates. The page mapping table is especially clever in its ability to swap in new pages with a single update, removing the necessity to modify nearby nodes. The split and merge techniques, through their specific delta record types, are also particularly well formed. I especially like the idea of a thread encountering a partially complete update having to finish the update itself - this is an excellent way to avoid any sort of blocking, and they have carefully crafted the data structure such that all the necessary information is already available. The cache performance is a great additional benefit of no in-place updates.

One question I am left with is how exactly garbage collection works - is it something triggered immediately at the end of each epoch? Is there a specific garbage collection daemon? The use of epochs to avoid collection of memory while another thread is using it is clever, and obviously the performance results indicate that this isn't an issue, but I am curious what they are using here. It does seem that each epoch is short enough that the garbage collection should be small on each round. 
