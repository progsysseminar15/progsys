#A Comprehensive Study of Convergent and Commutative Replicated Data Types
## Marc Shapiro, Nuno Preguica, Carlos Baquero, Marek Zawirski

###Xinghao Pan
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
Safe storage reclamation is also similar to tombstoning or the 2-phase approach of CRDTs.
