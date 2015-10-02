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

