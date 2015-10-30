# Highly Available, Fault-Tolerant, Parallel Dataflows
## Mehul A. Shah, Joseph M. Hellerstein, Eric Brewer

### Johann Schleier-Smith

Parallel dataflows are perhaps more in need of high availability measures, simply because the failure modes are greater in number. The authors derive their approach from the “process pairs” technique, an approach to distributed computation in which two replicas with fail-over capability provide for high availability. They first show how this technique can be adapted to pipelined flows, then further extend it to data-parallel processing. The system they developed, named *Flux*, does this by extending the “exchange operator” that is the established building block of parallel dataflows, allowing for fault tolerance at the granularity of individual blocks within the pipeline. A naive solution, in contrast, might apply the process pairs analogy at each partition separately, leading to a quadratic decrease in availability with increasing system size.

My view is that this paper identifies the right place to put fault tolerance: at the interface between components. I am less sure that other aspects of this work represent the right choice. The process pairs approach can in principle allow for very high availability, as the backup system is ready to go. The experiments support this result, with a failure showing a slowdown lasting only about a second (though I have to wonder whether a system that detects failure so quickly can be stable). Replication is expensive, especially for large systems, and I must wonder whether an N+M solution of some sort would be more appropriate, or whether greater availability might be possible with a more flexible approach to fault tolerance.

The Flux approach is fundamentally log-oriented, and I wonder whether the power of the log abstraction could be better leveraged.
There is a lot of complex synchronization (acks, state transfer, pause, and so forth) going on at the boundaries between components, and it seems a simpler solution should be possible, and I wonder whether Chandy-Lamport might have been a better starting point. I also wonder about how to provide recovery when the whole system goes down.

### Yifan Wu
Aside: I've never read about continuous query systems and it seems to be closer to the recent "big data
processing systems" like MR and Spark (as opposed to traditional DBMSs). I also thought systems like
Naiad were innovative for treating processes like DAGs but it appears that DBs have been treating
operators as DAGs for a while in query planning.

The paper mentions that parallel CQ systems don't provide great guarantees for fault tolerance and
availability. The traditional checking-pointing and logging incurs a high cost during recovery since
the buffer need to be loaded from disk. By having "process pairs", the system have redundant
computation that's fast for fail-over. (Aside: incidentally I first heard idea of the like at a talk
on MR by Jeff Dean and thought it was such a clever idea, but I see that it's not new...). The
system guarantees exactly-one input (no loss no duplication) and at least one dataflow. The key
challenge then is how to design the pairs. Cluster pairs introduce single points of failure (with
increased probability of second failure). I didn't quite get all the details of the Flux protocol,
but the constraint that primary and secondary copy cannot be on the same machine makes sense. The
producer consumer signaling process upon a takeover (after failure) seemed compex and it wodul be
great to flesh out the details in class.

Questions
- The system does rely on a global controller for state management --- could this be a scaling
bottleneck?
- Again I'm curious how much overlap existing popular systems have with the old system --- the
terminology is different and I cannot seem to wrap my head around the underlying design.
