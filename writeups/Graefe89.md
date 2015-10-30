# Encapsulation of Parallelism in the Volcano Query Processing System
## Goetz Graefe

### Johann Schleier-Smith

This paper gives a clear description of a clean and simple database implementation, yet a powerful one. Volcano distinguished between various sorts of parallelism: the inter-operator parallelism of pipelining or of tree-oriented bushy parallelism, as well as the intra-operator parallelism of partitioning. The author introduces one abstraction, the *exchange operator*, that can be used to express either. Importantly, the exchange operator creates a clean separation between the data-shuffling parallelism-providing operations, and the core logic. Separately encapsulating the query logic and the parallelism functionality as independent operators allows a great deal of flexibility, both for extending scalability and for extending functionality.

One of the things that I found interesting about the exchange operator is that it operates in a push-based manner, whereas database iterators are traditionally pull-based / lazy. This creates the need for a flow control mechanism, which Volcano provides rather elegantly through the use of semaphores. I also found a number of the technical details interesting, for example the note on spin-locks, which we today tend to take for granted. Most impressive, perhaps, is that the entire system is only 10,000 lines of code.

I found this paper rather straightforward and was left with few questions. The diagrams of Figure 1 left me rather confused, but I found the accompanying text clear. I continue to search for the most important takeaways from what strikes me as a classic paper, yet I keep coming back to the systematic description of parallelism, and the encapsulation of parallelism. 