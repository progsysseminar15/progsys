# Naiad: A Timely Dataflow System
## Derek G. Murray Frank McSherry Rebecca Isaacs
## Michael Isard Paul Barham Martin Abadi

### Ethan J. Jackson
The Naiad system, not unlike Dryad, is trying to be somewhat of a general data
processing framework on top of which simpler systems can be built.  Again, like
Dryad, Naiad allows programs to specify a computation graph made up of
vertices which do the actual computations, and edges which shuffle data
between them.  Unlike Dryad, Naiad allows cycles in these graphs which, while
somewhat constrained, allow the development of iterative algorithms.

The key idea here is controlling computational cycles through timestamping.
Each message in the system has a "logical" timestamp associating it with a
particular epoch.  This allows the system to notify vertices when they've
received all the data associated with a particular epoch and act accordingly.
It's an intellectually satisfying idea, though I'm not totally sure it's the
right abstraction.

I suppose what I'm struggling with here, is whether or not time should be
explicitly baked into the programming model, or left implicit like it is in
most systems.  Clearly, they've built a working system which has some
impressive numbers, but it's not clear to me whether their timestamping
algorithm is the key enabler of their results, or an incidental detail.  It
certainly seems to make the system somewhat more difficult to reason about.


### Erik Krogen
Naiad has some interesting ideas about combining a batch and streaming approach via the use of alerts that mark when all data for a certain period has been receieved, allowing coordination/finalization of data processing. Yet by allowing vertices to continue working on other data and not requiring communication in the general case, they maintain a low latency route. The concept of pointstamps (timestamp + location) is used to determine whether or not a particular datum can possibly flow to a given vertex through some clever use of data sharing, though this does still require broadcasted communication across the system.

They say that 'stateful dataflow vertices' are 'needed to execute iterative and incremental computations with low latency'. Is this fundamental? Spark achieves pretty low latency in a non-stateful manner. Stateless dataflow vertices are generally much easier to reason about (at least from a system point of view) and easier to provide fault tolerance for; is there a fundamental performance tradeoff with this? 

### Chenggang Wu
This paper presents Naiad, a distributed system for parallel data processing. The goal of Naiad is to simultaneously deliver high throughput for synchronous batch processing, low latency for asynchronous stream processing, and the ability to perform cyclic workload. Integrating these features into a single system makes Naiad a more efficient, succinct, and maintainable platform. Naiad provides low latency for streaming workload by asynchronously processing messages that don’t require coordination. For operations that requires batch processing (eg: aggregation), Naiad improves the throughput by enabling parallel batch processing within a dataflow (and even within a cyclic component). One of the main challenges for Naiad is to decide when a vertex within a cyclic component can perform a batch operation. The solution is to use a vector timestamp with two components: one representing the time epoch and the other representing the loop count. By formulating a timestamp ordering, it is possible to create a sequence of unprocessed operations in which one can result in another. Keeping track of the frontier of the sequence can give a clue to which batch operation can be safely executed. The experimental analysis section shows that Naiad can perform as effectively as other specialized systems on the same workload.

Actually I am not sure if I fully understand how the vector timestamp works to determine whether a batch operation is safe to execute. Hopefully, after tomorrow’s discussion it will become clearer.

