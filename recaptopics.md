Progressive: what did we say we liked at the beginning?
* Elegant.  Shankari: "observation exists, analysis is implied, and can be wrong"
* Natural world inherently progressive (Johann channeling Lamport)
* CALM: can have distributed consistency without coordination if everything is monotonic (i.e. progressive)
* Progressive helps with debugging/proving correctness of things inherent to distributed: non-deterministic msg races, partial failure, fault tolerance. 
* Replay: consistent, reproducible experimentation, debugging, deployment (recall Antelope, see Shankari's quote above)
* History can be interesting in its own right (recall Postgres): point-in-time queries, trend analysis

Recurring Themes:
* Implementation often has non-progressive/mutable somewhere in the stack.
* Try to localize the non-monotonicity somewhere cheap.  Think of CAS in Bw-Tree
* Garbage collection and reorg
  * inherent with limited resources: LFS, Postgres Vacuum, Edelweiss...
  * reorg even in the most recent work (e.g. Bw-tree)
* Storage organization traditionally presents inherent tradeoffs:
  * Write-optimized vs. Read-optimized?  (Think LFS, Postgres) Or something at a higher level of semantics? (Think about querying trends over time vs. point-in-time, checkpoints vs. replays, etc.)
  * Have things changed?  Big RAMs, Flash, make random access less painful?
  * Is it about ratios that enable background reorg? (size of random access "buffer" vs. size of block-oriented mag disks)?
  * Is it about the cost of coordination, and if so why has that become dominant?  Comm latency vs. compute?
  * Or are we just getting more clever (e.g. bW-tree, RLU)
* Deltas are natural in progressive.  Bw-tree, Diff dataflow, semi-naive Datalog evaluation
* In a progressive system, when are things "done"?
  * When is it safe to "Reveal" (in Bloom^L terms)
  * When is the "eventual" in "Eventually consistent"?
  * One answer: threshholds.  LVars, Bloom^L.
  * Another: Fail if you "witness" something frozen that turns out to change (LVish)


Similarities between
* progressive event handling (Lamport, SEDA, Elm)
* progressive dataflow (Naiad, Diff Dataflow, Volcano, Flux), and
* progressive data update (MVCC, LFS, Postgres, Bw-Tree, Monsoon, Tardis, LVars).
* Bloom et al an effort to unify events+dataflow, less focus on data update.  Opportunity?
* Related topic: control-flow vs. data-flow
  * "Threads"/Actors get played with regularly in the systems.  Lots of "piggybacking", "worker pools" etc.  
  * Can separate API and Architecture.
    * Exchange operator (Volcano): takes one relationship between control & data flow (the tight coupling of the 2 by iterators) and make it the API for a different relationship (the async push/pull/queue combos of shuffling data across a network)

Recuring Desire: Taxonomy/Map of "progressive" design patterns
  * Catalog ways to maximize progressiveness, and make non-progressive stuff free/fast/local. 
  * Maybe even teach a compiler to choose these things!
  * Protocol synthesis.
  * What level of history are we reasoning about?
    * I/Os?
    * Lattice operations?
    * Logic?
    * Application semantics?  etc.
