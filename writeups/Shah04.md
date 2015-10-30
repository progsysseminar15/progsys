# Highly Available, Fault-Tolerant, Parallel Dataflows
## Mehul A. Shah, Joseph M. Hellerstein, Eric Brewer

### Johann Schleier-Smith

Parallel dataflows are perhaps more in need of high availability measures, simply because the failure modes are greater in number. The authors derive their approach from the “process pairs” technique, an approach to distributed computation in which two replicas with fail-over capability provide for high availability. They first show how this technique can be adapted to pipelined flows, then further extend it to data-parallel processing. The system they developed, named *Flux*, does this by extending the “exchange operator” that is the established building block of parallel dataflows, allowing for fault tolerance at the granularity of individual blocks within the pipeline. A naive solution, in contrast, might apply the process pairs analogy at each partition separately, leading to a quadratic decrease in availability with increasing system size.

My view is that this paper identifies the right place to put fault tolerance: at the interface between components. I am less sure that other aspects of this work represent the right choice. The process pairs approach can in principle allow for very high availability, as the backup system is ready to go. The experiments support this result, with a failure showing a slowdown lasting only about a second (though I have to wonder whether a system that detects failure so quickly can be stable). Replication is expensive, especially for large systems, and I must wonder whether an N+M solution of some sort would be more appropriate, or whether greater availability might be possible with a more flexible approach to fault tolerance.

The Flux approach is fundamentally log-oriented, and I wonder whether the power of the log abstraction could be better leveraged.
There is a lot of complex synchronization (acks, state transfer, pause, and so forth) going on at the boundaries between components, and it seems a simpler solution should be possible, and I wonder whether Chandy-Lamport might have been a better starting point. I also wonder about how to provide recovery when the whole system goes down.
