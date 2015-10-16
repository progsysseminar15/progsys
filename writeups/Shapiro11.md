# A Comprehensive Study of Convergent and Commutative Replicated Data Types
## Marc Shapiro, Nuno Preguica, Carlos Baquero, Marek Zawirski

### Xinghao Pan
The largest section (Section 3) was dedicated towards showing how simple CRDTs can be made complex and composed together for complicated operations.
This begs the question of how far we can push the limits of CRDTs.
What operations allow us to build new lattices from existing ones?
What objects cannot be represented using CRDTs?
What programs can / cannot be written using monotone logic?
As we have seen in both the CRDT and CALM / Bloom / Bloom^L work, the class of monotone programs may be much larger than we initially believed.
In fact, no-abort concurrency control protocols may be monotone too -- think of conservative T/O for example; the low watermark of read / writes increases monotonically, and as long as all reads and writes are sent to all replicas, we will reach an eventually consistent state.

A question that I kept coming back to while reading this paper was: How are CRDTs and Bloom^L different?
CRDT differentiate themselves from Bloom by criticizing Bloom for only supporting simple grow-only sets, but Bloom^L goes some way towards answering this criticism.
Bloom^L claims that (a) CRDTs require programmers to ensure monotonicity, and (b) CRDTs do not guarantee monotone application logic in general.
However, Bloom^L also supports user-defined lattices which require programmrs to provide proper merge functions.
Using operation-replicated CRDTs also helps ensure the overall application logic is monotone.
I suppose one point of difference might be that CRDTs might `silently' fail if a programmer assumes a non-monotone operation is in fact monotone, but in Bloom^L non-monotone operations are made explicit and forced to coordinate.

This paper only briefly discusses garbage collection, and requires some strong conditions for garbage collection to take place.
Edelweiss uses program analysis to identify when a tuple is long longer visible or have further effects downstream, which is only possible due to the Bloom^L language support.
The Edelweiss paper also reads more like a bag of tricks rather than a principled approach towards monotone garbage collection.
It would be interesting to try to generalize Edelweiss to general ELEs / CRDTs.
As a naive example, reference counting can be thought of as an increment/decrement counter, which the CRDT paper shows is monotone.


### Erik Krogen
A great deal of information was presented in this paper. One of the main takeaways for me was that, to implement things in a CRDT manner, you have to think about them in a very unconventional manner. Yet, by building the right abstraction, you can hide the implementation oddities behind a reasonable interface (though interesting restrictions sometimes bleed through this abstraction). I was surprised with some of the structures they were able to build as a CRDT, e.g. the graph.

As I wondered after reading the Bloom paper, I am left with two questions: how easy would it be to build on top of these, and are there fundamental restrictions on what can be accomplished using CRDTs? They are able to present some structures as CRDTs that I would not have expected; can this be done for everything? When does it make more sense to synchronize and use more conventional structures than to use these more limited, more complex CRDTs, and vice-versa?

###Johann Schleier-Smith

The authors present both an accessible review of the CRDTs and a library of examples. Their approach is refreshing, as each individual example is readily digestible, and relates to a real-world example. The final, example, co-operative text editing, demonstrates clearly with two implementations that CRDTs can be used in applications that are both useful and non-trivial.

I also found the relationship between state-based and operation-based CRDTs to be an interesting one. The two formulations are equivalent, but have rather different requirements. Op-based replication requires reliable broadcast, whereas state-based replication does not. State-based replication, on the other hand, requires greater bandwidth. I wonder whether the Specification 4 (State-based emulation of operation-based object) might inspire a more general approach to dealing with a lossy channel. I also wonder whether there these are the only two formulations of CRDTs, or whether there is some other formulation that is also equivalent but presents yet another alternative model for programming and perhaps different system expectations.

I am still reflecting on whether I like CRDTs, or eventual consistency in general. Under what circumstances should we be exposing data that has not converged? Editing a local document copy even as another person is working on a remote one seems reasonable. It is possible for nonsense to emerge, but there should be opportunities for marking this. But is it ok for items removed from a shopping cart to reappear, just so long as they will eventually again disappear? There are some cases where it seems better to hide the internal work of the distributed system until convergence has occurred. I have not seen, however, how to tell that a CRDT has converged.

I also wonder whether the problem solved by CRDTs is too narrow. Replication is important—it allows for fault tolerance, reduces latencies in a distributed system, even allows for disconnected operation. However, it speaks only to a limited extent to parallel speedup, an important promise of distributed systems. I am left wondering how broad the class of problems that benefit from this approach is.

### Chenggang Wu

The motivation of this work is that in distributed systems, maintaining a global total order of operations is very costly, and can negatively affect scalability and performance. For some applications, maybe we do not need such a strong consistency guarantee in the first place. Therefore, a more practical and less expensive approach is to use mechanisms that only guarantee eventual consistency: A replica can perform an update without reaching a consensus with other replicas, and later on, the replicas talk to each other in the background and reconcile conflicting updates. By getting rid of online coordination overhead, we can achieve better performance, scalability, and availability. The paper uses convergent replicated data types (CRDT) for guaranteeing eventual consistency. These data types have mathematical properties to ensure that diverged versions of replicas can be safely reconciled.

One of the key contributions is that the paper shows how to build complex CRDTs (graphs, monotonic DAGs, and sequence) from simple ones (counters and registers). While in theory, using these sophisticated CRDTs can let us build a larger class of monotone programs, I am wondering if there are more concrete examples of applying these data types to distributed systems and some experimental evaluation of their usefulness.


### Yifan Wu

Questions and comments:
- There are two types of CRDTs based on their convergence model, one is state-based (CvRDT) and one is
operation-based (CmRDT). State-based appraoch has been around for a while, in systems like NFS (and
recently Dynamo). CmRDTs I don't understand very well --- what is the "source pre-condition"? How
does its characteristics map to real-life system issues?
- this *does* address the issue of phantom deletes (deleted in one replica but shows up later again,
  per Dynamo's cart anomoly)
  right? So it's more than just eventual consistency? Would like to understand what happens in this
  example a bit more.
- similar to those raised above: does Bloom^L's lattice and morphims have the same powers as the
  CRDTs listed? The paper commended that the CALM appraoch does not support remove without
  synchronization but I thought that was addressed in Bloom^L. I do think that Bloom's lattices and morphims
  as foundations to reason about concurrency-friendly data-structures to be cleaner than a
  "portofolio" though.
- how to reason about whether to encapulate monotonicity into datastructures or the programming language?
- is there more meansures to reason about eventual-consistent systems? I feel like there is a wide
  spectrum of them and there should be more vocabularies to describe their performance
  characteristics.
- Op-based seem to rely on reliable network/broadcast channel --- what is the damage with less
  reliable network?
