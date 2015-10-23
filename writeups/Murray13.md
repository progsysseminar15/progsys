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
