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

