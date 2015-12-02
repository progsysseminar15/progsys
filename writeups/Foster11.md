# Frenetic: A Network Programming Language
## Nate Foster, Rob Harrison, Michael J. Freedman, Christopher Monsanto, Jennifer Rexford, Alec Story, and David Walker

### Johann Schleier-Smith

Frenetic promises a huge improvement over the pre-existing alternative, the NOX controller platform most typically used with OpenFlow. While my familiarity with network programming is limited, the low-level nature of existing tools is evident to me, as is the need for something better.

A few design decisions in Frenetic are particularly interesting. The authors distinguish between the query language, which is a sort of streaming SQL (somewhat similar to CQL), and the policy management language, which is based directly on functional reactive programming (interestingly enough, drawing inspiration from a language developed for robotics). Wouldn't it be better to have just one language? I've convinced myself that the answer is no, and it's interesting to see how the two interact. The query language produces a stream of events as output, events that are time-indexed, just like the events of the policy language. This is a really nice connection, and suggests that the event paradigm can provide a rather generic interface between programming environments that are quite different.

Another interesting design decision is that of adopting a centralized view of the network. The authors emphasize how this contrasts with the design of the NDLog language, in which programs take on a distributed style. It seems natural to me that the centralized viewpoint makes life easier for the programmer, though I would like to better understand the trade-offs involved.

I was pleasantly surprised to see that Frenetic provides good performance, comparable to hand-written NOX, perhaps because the authors provided programmers with good ways of reasoning about it. It's also nice to see that the authors implemented a broad cross-section of useful network management functions, so demonstrating that Frenetic can meet practical demands.