# Encapsulation of Parallelism in the Volcano Query Processing System
## Goetz Graefe

### Johann Schleier-Smith

This paper gives a clear description of a clean and simple database implementation, yet a powerful one. Volcano distinguished between various sorts of parallelism: the inter-operator parallelism of pipelining or of tree-oriented bushy parallelism, as well as the intra-operator parallelism of partitioning. The author introduces one abstraction, the *exchange operator*, that can be used to express either. Importantly, the exchange operator creates a clean separation between the data-shuffling parallelism-providing operations, and the core logic. Separately encapsulating the query logic and the parallelism functionality as independent operators allows a great deal of flexibility, both for extending scalability and for extending functionality.

One of the things that I found interesting about the exchange operator is that it operates in a push-based manner, whereas database iterators are traditionally pull-based / lazy. This creates the need for a flow control mechanism, which Volcano provides rather elegantly through the use of semaphores. I also found a number of the technical details interesting, for example the note on spin-locks, which we today tend to take for granted. Most impressive, perhaps, is that the entire system is only 10,000 lines of code.

I found this paper rather straightforward and was left with few questions. The diagrams of Figure 1 left me rather confused, but I found the accompanying text clear. I continue to search for the most important takeaways from what strikes me as a classic paper, yet I keep coming back to the systematic description of parallelism, and the encapsulation of parallelism.

### Yifan Wu

"Exchange" as a new operator seemed very clever, creating a clean abstraction for parallelism. The
authors did comment on the tradeoffs between parallelism and locking. Additionally query planning
would seem a lot harder as well. The differentiation of demand-driven data-flow and data-driven
data-flow is an interesting way to look at the design decisions.

"exchange" performed significantly better with larger packet size. Additionally cache locality might
be a factor also; it doesn't seem to me that the system/query planner is able to capture this since
the abstraction is so clean. I also wonder what the interaction of "exchange" and I/O partitioning scheme is.
In general it would be great to go over the "design space" of data-flows. For example, I found it hard to compare
Volcano with Naiad.

### Chenggang Wu

This paper introduces Volcano as well as the design and implementation to parallelize Volcano. In general, Volcano uses a conventional file system that includes a module for device, buffer, file, and B+-tree management. Within a process, it uses iterator that support open-next-close protocol to implement demand-driven dataflow. In multi-process scenario, the exchange operator is responsible for parallel execution and synchronization. Vertical parallelism pipelines the query evaluation between processes. Horizontal parallelism has two forms: bushy parallelism and intra-operator parallelism. In the first case, different CPU execute different sub-trees of a query tree, whereas in the second case, CPUs perform the same operation on different subsets of the data. The paper also discusses the change required in the file system to support the parallelization, comparison with GAMMA, and performance of Volcano under the multi-processer setting.

The paper has lots of implementation details and is pretty dry. There doesn’t seem to be many resources on the internet either. Their only examples shown in Figure 1 helps a bit in terms of understanding, but I still prefer a more concrete query example and a visualization of the query-tree as well as processes that are responsible for the different portions of the query-tree.

### Gabe Fierro

This paper looks at how to parallelize a tree-based query plan for dataflow,
using a system called Volcano. There is a lot of time spent on establishing the
notion of a node, which is the unit of abstraction for operators. An iterator
model connects each of the nodes. The exchange operator in the Volcano system
allows us to parallelize these operators without having to change those
operators to be aware of parallelism. The system supports three types of parallelism:
vertical parallelism (pipelining between processes), horizontal parallelism (bushy: two
subtrees are handled by different processes), horizontal parallelism (intra-operator:
partition the operator across multiple processes).

There's a bunch of systems stuff in this paper on how to actuallly do this in
a real system, which was nice to read. The iterator interface feels like
a progressive system, but its unclear how much other coordination is needed.
Intra-operator parallelism is what the Flux paper does, so I hope the discussion
delves into what aspects of Flux and volcano are similar in solving similar
problems. Flux does use a variation of the exchange operator, but introduces a global
coordinator to coordinate all the highly-available, fault-tolerant stuff. I wonder
if the larger Volcano system accounts for similar features.
