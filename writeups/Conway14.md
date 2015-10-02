# Edelweiss: Automatic storage reclamation for distributed programming

## Neil Conway, Peter Alvaro, Emily Andrews, Joseph M. Hellerstein

### Johann Schleier-Smith

The way that Edelweiss builds upon bloom is interesting. First it removes features, the <- deletion operator is not supported, and channels must be derived from persistent collections. The latter demand leads to distributed applications that follow what the authors call the *Event Log Exchange* (ELE) pattern. While I have not found ELE defined or described as a pattern elsewhere in the literature, it embodies an intuitive notion for distributed systems. All state can be constructed, or reconstructed if necessary, by replay of communication between nodes.

While one might expect that a more restrictive programming model leads to lengthier programs, the Edelweiss programs developed in this work are smaller than their Bloom counterparts. The improved inference abilities for Edelweiss programs allow for automated garbage collection, enabling efficient execution of what appear to be very inefficient programs. I still wonder whether the language provides programmers with enough intuition about when automatic garbage collection will be efficient, but the demonstration remains compelling nonetheless. A broader theme that may be interesting to learn more about is that of designing languages for garbage collection.

I continue to remain curious about is the value of providing exactly-once semantics as a distributed systems primitive. In Bloom and Edelweiss, the 
<~ operator allows messages to be lost, reordered, or delayed, but it appears to avoid duplicate messages. This is perhaps the simplest implementation of a “fire and forget” communication scheme, one that mirrors many physical implementations (though not IP networks, where duplication is possible). What might a language built upon more reliable messaging (streams, or perhaps simply guaranteed delivery) look like?