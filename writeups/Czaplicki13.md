# Asynchronous Functional Reactive Programming for GUIs
## Evan Czaplicki and Stephen Chong

### Johann Schleier-Smith

Reactive programming has in recent years captured a great deal of mindshare in industry. This work describes theoretical foundations supporting the movement, provides a clean illustration of Functional Reactive Programming (FRP) in a simple language, and highlights specific performance challenges.

The Elm core language is quite small, which goes to show that by choosing building blocks carefully we can achieve a great deal of expressiveness. While I don't have a proof that the declarative nature of the languages is the key to its simplicity and power, I would advocate that this a key underlying reason.

Elm has an interesting approach to history. While it allows the programmer to define state variables as folds over history, its type system disallows certain compositions of these types, ensuring that it never needs to store any history. I am curious to understand this better, especially from the progressive systems perspective. I wonder whether this limitation is, as the authors claim, not a problem in practice, or whether it is an obstacle in certain useful programs. I also wonder whether a runtime implementation that understands history might do without the async language features, whether it might on its own guarantee a sort of eventual consistency, extracting concurrency implicit in the program and reshuffling execution to optimize responsiveness. This seems like a fruitful area for further research.

### Xinghao Pan

Elm is an FRP language for programming GUIs, which provides a simpler abstraction through which programmers can model dependencies in a highly-declarative way.
The differences of Elm from previous FRPs are (1) Elm uses a push model, thus avoiding unnecessary recomputation; and (2) Elm allows asynchronous computation, thus preventing the GUI from hanging while waiting.

The authors argue that FRPs are a more natural way to express GUI event streams.
Without much GUI programming experience myself, I am unable to evaluate this claim.
However, ELM programs translate to essentially a dataflow / SEDA architecture, so I can imagine that it would be possibly easier to reason about than languages that have mutable state.
This begs the question of whether we could have used a dataflow language instead; what are the relative strengths of dataflow and FRP; are dataflow and FRP essentially equivalent?

I found the management of asynchronous signals to be a rather clever yet natural trick which could possibly be applied to other dataflow-like programs.
Adding an async node to the DAG breaks the DAG into subgraphs so that synchronization order is maintained within the subgraphs but not between them.
Each async node produces no-ops until the corresponding async event returns, upon which it generates a new signal. (See Figure 8.)

### Chenggang Wu

This paper discusses Elm, a FRP language that helps easily create responsive GUIs. The two major features of Elm are that it supports asynchronous FRP and purely functional graphical layout. The paper argues that asynchronous FRP allows the programmer to specify when the global ordering of event can be violated, and therefore enables efficient concurrent execution of FRP programs.

It seems to me that declaring a node as “async” only make sense if the destination node has multiple edges connecting to it. At a high level, declaring a node as “async” let the destination node retrieve the current (unchanged) value of the async node instead of waiting for the value that’s currently being computed. However, I think all of these async and ordering issues exist because the paper’s design decision that every source node produces one value for each event regardless of whether the event is relevant to the node. As a comparison, in rx.js if the event is not relevant to the node, no values are fired, and therefore rx does not seem to be bothered with async and ordering issue. To provide equivalent functionality, RX.CombineLatest does very similar job as declaring a node as “async” in Elm, and RX.Zip also does very similar job as NOT declaring a node as “async” in Elm. So I wonder if the async and ordering issues are fundamental for FRP.

