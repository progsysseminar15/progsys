# Frenetic: A Network Programming Language
## Nate Foster, Rob Harrison, Michael J. Freedman, Christopher Monsanto, Jennifer Rexford, Alec Story, and David Walker

### Johann Schleier-Smith

Frenetic promises a huge improvement over the pre-existing alternative, the NOX controller platform most typically used with OpenFlow. While my familiarity with network programming is limited, the low-level nature of existing tools is evident to me, as is the need for something better.

A few design decisions in Frenetic are particularly interesting. The authors distinguish between the query language, which is a sort of streaming SQL (somewhat similar to CQL), and the policy management language, which is based directly on functional reactive programming (interestingly enough, drawing inspiration from a language developed for robotics). Wouldn't it be better to have just one language? I've convinced myself that the answer is no, and it's interesting to see how the two interact. The query language produces a stream of events as output, events that are time-indexed, just like the events of the policy language. This is a really nice connection, and suggests that the event paradigm can provide a rather generic interface between programming environments that are quite different.

Another interesting design decision is that of adopting a centralized view of the network. The authors emphasize how this contrasts with the design of the NDLog language, in which programs take on a distributed style. It seems natural to me that the centralized viewpoint makes life easier for the programmer, though I would like to better understand the trade-offs involved.

I was pleasantly surprised to see that Frenetic provides good performance, comparable to hand-written NOX, perhaps because the authors provided programmers with good ways of reasoning about it. It's also nice to see that the authors implemented a broad cross-section of useful network management functions, so demonstrating that Frenetic can meet practical demands.

### Erik Krogen

This seems to be an excellent example of a situation where a declarative language makes a ton of sense for expressing your program. The example programs look very intuitive, and are able to very concisely represent complex interactions. I am not familiar with network programming, but their description of the normal pitfalls of attempting to compose different rules on plain NOX sounded absolutely terrible - Frenetic seems to be an enormous improvement over this. The idea of Frenetic reacting to changes in the network instead of setting timeouts on rules to just let them expire when they're no longer needed is great as well, and seems to be much more intuitive as well as more performant in some cases. 

The only concern I had when reading the paper was one of performance, so I was pleasantly surprised and impressed to see their performance results being very competitive with hand-tuned NOX programs. I think they did an excellent job of the trade-off between expressiveness and ease of understanding the cost of your programs; I was thinking initially that it would probably be very easy to accidentally write a program that is very prohibitively expensive, but it seems that due to the design of the language this scenario should be readily apparent to the programmer.

Overall I am very impressed with this and it seems they have done an excellent job of providing a clean abstraction on top of a difficult-to-use low level system, but it will be interesting to hear a perspective from someone who actually does network programming.

### Xinghao Pan

There are a number of things going on with Frenetic as an improvement over NOX.
Firstly, the query language is decoupled from policy management, which allows it be to expressed in a declarative language without any state modification.
Secondly, the policy management is written in an FRP language, so policies can be composed together easily in a pipeline, instead of having to manually factor out the potentially large number of disjoint cases that are managed via different priorities.
Third, both the query and policy management produce event streams.
Since there were a few ideas thrown into the paper, it became difficult for me to keep track of what were the key concepts or takeaways that I should have.
Was it the separation between the query and policy management, or the event stream (immutability?) idea, or the declarative / functional interface, which contributed to the success of Frenetic?

The separation of query / policy management / implementation reminded me of the functional relational programming separation we saw in the out-of-the-tarpit paper.
The general principle seems to be that one should pick a different (restricted) language for each task, carefully design the system so that the core parts (essential state / policy management) are not affected by other parts (accidental state or essential logic / query), and leave the implementation / infrastructure as a problem to be solved separately.

I also found it interesting that Frenetic was implemented via NOX, so one could argue that it is nothing more than syntatic sugaring, although that would miss the point that NOX is obnoxious to program in whereas Frenetic is much more natural.
This leads back to one of our ongoing discussions on layering progressive systems on mutable state and vice versa.
There seems to be a strong argument for picking the correct abstraction (progressive systems, FRP, functional declarative, etc.) solely for the sake of reducing programmer complexity, and worry about achieving performance secondly.
