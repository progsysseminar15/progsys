# Differential dataflow
## Frank McSherry

### Erik Krogen

To be honest, I didn't really understand a lot of what McSherry was saying here... The overall idea of having incremental computation defined e.g. over an (epoch, iteration) ordering with dependencies on both (i-1, j) and (i, j-1) makes sense, I got pretty thoroughly lost in his implementation details. He goes through the work of defining a difference as (collection - sum of previous differences) but I don't see why this is helpful - isn't the point to build up a full collection as a result of differences? It seems to be that he needs to calculate a new difference from a previous difference (to pass on to subsequent operators) - maybe this is what he's doing and I'm just missing it. Although I can follow his example, I don't really see what benefits I'm gaining from it--I actually believed in the idea of differential dataflow more *before* reading the example...

In any case, the idea is interesting, and hopefully after some discussion I'll be able to see its usefulness more. 

### Ethan J. Jackson
I can't say I understood much of this.  I suspect it was written for an
audience well versed in both rust and dataflow of which I have merely a cursory
knowledge.  The basic idea appears to be an extension to some of the ideas in
Naiad:  you have some sort of cyclic computation graph for which you're using a
logical clock to mediate recursion.  The difference here is that by storing
diffs at each node, through some magic, you can make incremental updates to the
input data without requiring full recomputation.  Again, this is just a guess
as I really have no idea what he was saying.  I'll refrain from saying anything
more to avoid the risk of sounding incoherent.

### Chenggang Wu
This paper presents a new computation model called differential dataflow. It generalizes both the incremental and prioritized computation models with the additional power of handling changing datasets. When the dataset is modified during the computation, traditional optimization strategy such as incremental computation has to completely restart from the beginning. Differential dataflow is able to reuse states corresponding to the parts of the graph that have not changed, and therefore manages to improve the performance by reducing the number of operations executed during the iterative computation.

I think I get the overall idea of this computation model, but there are some details that make me confused. For example, in section 3.3, their differential computation approach uses partial ordering to define the deltaA11 as A11-(deltaA00+deltaA01+deltaA10). However, using the traditional total ordering approach, deltaA11 is defined to be A11-A10. Correlating the above two definition yields A10=(deltaA00+deltaA01+deltaA10)=A00+A01-A00+A10-A00, which implies that deltaA01=0. Since this is clearly not true, does it mean that the two definition of deltaA11 are different?

### Xinghao Pan

Differential dataflow is a generalization of incremental dataflows to partial orders of updates, which are retained for potential future computations.
A use-case which they are particularly interested in is incremental updates to iterative computations.
This is expressed as a two-dimensional lattice corresponding to rounds of input and iterations of the computational loop.
Previous rounds and iterations can be re-used for computing new differences and state.

Both the blog post and the paper were not exactly clear on how this computational re-use can be done in general.
The computational advantage seems to be very dependent on whether there are computationally efficient ways to accumulate or compute differences.
Otherwise computations could become increasingly expensive, since each state requires summing and differencing larger sets of updates.

The paper appears to acknowledge this issue (Section 3.4), but argues that "many specific operators have more efficient implementations".
I would be intersted to figure out if there is a (algebraic) property that captures what differential dataflows can be efficiently compute.
At the very least, the 'sum' of deltas must be commutative, associative, and have an inverse.

The separation of progress into multiple dimensions is a simple but powerful extension of linear progress of incremental dataflows, and seems vaguely related to some of the papers we've read.

1. In Lamport clocks / vector clocks, we saw that time and causality are only partially ordered.
2. The CALM-Bloom / CRDT work has replicas making separate progress before merging into an eventually consistent system, similar to how differences are summed together in differential dataflows.

In general, I think we can think of progressive systems as differential dataflows, with facts / updates accumulated over (partially ordered) time.
Is this a useful representation? Does it buy us any expressive power or computational advantages?
