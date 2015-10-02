# Edelweiss: Automatic storage reclamation for distributed programming

## Neil Conway, Peter Alvaro, Emily Andrews, Joseph M. Hellerstein

### Johann Schleier-Smith

The way that Edelweiss builds upon bloom is interesting. First it removes features, the <- deletion operator is not supported, and channels must be derived from persistent collections. The latter demand leads to distributed applications that follow what the authors call the *Event Log Exchange* (ELE) pattern. While I have not found ELE defined or described as a pattern elsewhere in the literature, it embodies an intuitive notion for distributed systems. All state can be constructed, or reconstructed if necessary, by replay of communication between nodes.

While one might expect that a more restrictive programming model leads to lengthier programs, the Edelweiss programs developed in this work are smaller than their Bloom counterparts. The improved inference abilities for Edelweiss programs allow for automated garbage collection, enabling efficient execution of what appear to be very inefficient programs. I still wonder whether the language provides programmers with enough intuition about when automatic garbage collection will be efficient, but the demonstration remains compelling nonetheless. A broader theme that may be interesting to learn more about is that of designing languages for garbage collection.

I continue to remain curious about is the value of providing exactly-once semantics as a distributed systems primitive. In Bloom and Edelweiss, the 
<~ operator allows messages to be lost, reordered, or delayed, but it appears to avoid duplicate messages. This is perhaps the simplest implementation of a “fire and forget” communication scheme, one that mirrors many physical implementations (though not IP networks, where duplication is possible). What might a language built upon more reliable messaging (streams, or perhaps simply guaranteed delivery) look like?

### Erik Krogen
The ability to reason about storage reclamation in isolated, preprogrammed scenarios and then apply these to larger applications is very interesting. I don't like the comparison made in the paper about a causally consistent KV store taking 13,000 lines of C++ as opposed to 19 in Edelweiss--this seems like a very unfair comparison, as the C++ store likely had a more rich feature set, was more configurable, etc. I don't doubt that even the core functionality would take many LOC than Edelweiss, but this still seems like a silly comparison. 

It would be very interesting to see if these ideas could be extended to more traditional programming models. My biggest concern about Edelweiss--it seems that you need to have a very powerful understanding of what is happening under the hood to be able to get Edelweiss to cooperate correctly with you. Programs need to be written with a solid understanding of the cases that Edelweiss knows how to deal with; hopefully this happens almost automatically, but I am skeptical that will always be the case. Some of the reclamation cases discussed in the paper seem to be motivated by specific use cases that were discovered as applications were being written (e.g. "Edelweiss supports several special cases that avoid the need for user-supplied punctuations for this program")--I wonder just how general the rules implemented are. 

### Xinghao Pan

Edelweiss is a sublanguage of Bloom that ensure nodes accumulate knowledge, and that no message sent across nodes is ever retracted or forgotten.
It is catered to Event Log Exchanges where nodes accummulate and exchange immutable logs of events.
The key novelty claim is that Edelweiss automatically reclaims space without programmer assistance.
This is achieved by a collection of mechanisms, of which the DR+ technique appears to be the most important.

The DR+ rule looks for persistent collections that are involved in a set difference pattern: X.notin(Y).
If both X and Y are persistent and acummulative, it is easy to see that any element of Y will be effectively be deleted from X, and thus can be reclaimed safely from X.
Other techniques then help to reclaim storage from the subtrahend Y.
The bulk of the paper then demonstrates how we can build increasingly complicated applications in Edelweiss, including supporting causal consistency and snapshot reads (which is basically what RAMP requires).

Interestingly, as the paper also notes, the garbage collection of Edelweiss actually runs counter to the CALM / Bloom goal of monotonic programs.
Since storage reclamation is really only possible via set difference, we have to introduce non-monotone logic for Edelweiss' rule rewritting to work.

On the other hand, I think it is possible to extend the automatic garbage collection (particularly DR+ rule) to other ELE, and more generally progressive, systems.
All we require is that all collections only accumulate knowledge and that there is some way to indicate facts that are no longer valid / have been dominated.
The first requirement is easily obtained almost by definition of progressive systems.
Figuring out dominated facts may be harder, but could possibly be done by having a conservative estimate of the minimum progress across all nodes.
(Think about conservative T/O in Berstein & Goodman.)
