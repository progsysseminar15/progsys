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

### Gabe Fierro

Naiad is a distributed system that implements a parallelized dataflow
computation model called "timely dataflow" which aims to support low-latency
streaming while minimizing the amount of coordination between nodes. There is
also a lot of focus on supporting cyclic operations; this, combined with the
partitioning function, makes it feel like an abstraction over map-reduce.  They
place special logical timestamps on the edges between vertices (for of course
this is all modeled as a graph because its dataflow) and ensure that data never
flows "backwards". They then leave it up to the computation runnning how much
coordination it requires, either processing more incoming values, or sending
output values out to other vertices. With the use of the timestamps and the
knowledge of the graph layout, they can avoid some unnecessary coordination
that occures when the agents need to find out what they need to coordinate.
Having time flow forward and being able to reason about flow with partitioning
and known graphs means that those types of coordinations don't need to be done
(or at least to a lesser degree).

Some of the progressive-system flavors are here again. Garbage collection I
thin kis in the form of the 'checkpoint' and 'restore' interfaces. On periodic
checkpoints, the system will flush queues, and the system can recover from a
failed process by restoring from a checkpoint. I understand that Naiad is
faster to a degree because it sidesteps coordination among a large number of
nodes and knows where things *should* be. In the batching cases that are highly
parallel (e.g. wordcount) and are traditional map-reduce, I wonder if Naiad
does produce a performance increase. Its not clear that their performance gains
there are due to the timely dataflow or rather due to a well-engineered system
that follows a normal map-reduce computation pattern.


### Xinghao


Naiad is a high throughput, low latency distributed system for iterative, incremental computations, and is supported by the "timely dataflow" computation model.
As I understand it, timely dataflow converts nested looping into dataflow structures, by essentially tagging messages with logical timestamps that indicate the stage of the computation.

A few thoughts and comments:

1. Timely dataflows makes more explicit that even feedback / looping are 'progressive', i.e. only move forward in time, and contrasts with previous systems that either do not support iterations or provide weak consistency guarantees.
In practice, I don't see how this is much different from batch iterative systems which unroll the nested loops.
If differential dataflows are taken out of Naiad, it is pretty similar to Spark / DryadLINQ with pipelining enabled.
  * On a related note, I suspect that much of Naiad's performance comes from differential dataflows, which interestingly were not given much attention in this paper.
2. The authors argue that OnNotify, which is a non-monotone computation that requires coordination, may be useful, for example, if a single value is sent.
  * To be fair, Edelweiss-Bloom introduced single-value punctuations, which can arguably be shown to be monotone.
3. The pointstamping system is a mechanism to simulate logical causal time by exploiting information in the computation graph structure.
In contrast, Lamport clocks / vector clocks track physical causal time.
  * The concept of a frontier brings out the need to wait on earlier pointstamps. This is manifested as higher latency in the presence of micro-stragglers.
  Is this fundamental to all non-monotone operations? Are there ways to minimize latency (for example, by detecting the frontier at the key level) without too much need for coordination and communication?


### Johann Schleier-Smith

There are a few things that I really like about this paper. Whereas previous scalable processing frameworks such as MapReduce or Dryad supported only acyclic graphs, Naiad's cyclic computation graph allows for a richer and more general processing model. How much more general? I'm not sure how to measure it. The authors show an implementation of iterative machine learning algorithms, and I'm still thinking of examples that would not be lend themselves well to efficient implementation on the Naiad platform.

The “pointstamp” concept introduced by Naiad is a novel notion of indexing a computation. To me it is rather appealing, as it is one answer to the question of how to define “spacetime” for computation. In Naiad's definition notions of space and time are both connected to the program specification. Time is measured according to an input epoch sequence, together with a vector of loop counters, that originate in the graph, whereas space is indexed along the nodes and edges of the computation graph. This seems to be a construction that is both clever and general. There are a number of rules that make this system work, but key among them is that nodes cannot send events backwards in time.

Naiad presents a powerful approach, but its API is not one that is practical to program against directly. While the authors suggest SQL, MapReduce, and graph processing abstractions, these seem to capture only a limited amount of Naiad's flexibility and capability. What is really the best way to program for timely dataflow?