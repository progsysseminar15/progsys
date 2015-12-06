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

### Erik Krogen

I'm not very familiar with FRP or web GUI creation in general, so it's hard for me to say how natural Elm feels as a way to program. Composing graphical elements through their functional language seemed unintuitive to me, and although it was concise for their examples it seems that creating a complex GUI in that manner would be difficult - but this may just be because I'm not very comfortable with functional languages. Reacting to events instead of constantly recomputing seems obvious to me (especially since JavaScript is inherently an event driven language), and I'm kind of surprised that this was a new development over previous approaches. I feel the same way about the asynchronous computations - JavaScript is normally an asynchronous language, so it seems reasonable that you would be able to specify asychronous computation. It may just be because FRP is relatively young (I think?) so things are still sort of figuring themselves out - the async nodes certainly do add another level of complexity.

While they argue that Elm's GUI creation "can represent quite complex interactions with a small amount of code", I am not entirely sure that I buy their claim. I do like how concisely you can represent some things that are nontrivial in plain JavaScript (e.g. all of Figure 13). I am curious to know if any full web applications have been written in Elm, and would be interested to see how the code looks for a full production web app. I do think that the GUI creation and event handling system seem to integrate very nicely, which is definitely a plus. All in all the FRP paradigm seems to be an effective layer on top of JavaScript and Elm seems to provide a model that is more expressive and capable than what else is currently available. 

### Chenggang Wu

This paper discusses Elm, a FRP language that helps easily create responsive GUIs. The two major features of Elm are that it supports asynchronous FRP and purely functional graphical layout. The paper argues that asynchronous FRP allows the programmer to specify when the global ordering of event can be violated, and therefore enables efficient concurrent execution of FRP programs.

It seems to me that declaring a node as “async” only make sense if the destination node has multiple edges connecting to it. At a high level, declaring a node as “async” let the destination node retrieve the current (unchanged) value of the async node instead of waiting for the value that’s currently being computed. However, I think all of these async and ordering issues exist because the paper’s design decision that every source node produces one value for each event regardless of whether the event is relevant to the node. As a comparison, in rx.js if the event is not relevant to the node, no values are fired, and therefore rx does not seem to be bothered with async and ordering issue. To provide equivalent functionality, RX.CombineLatest does very similar job as declaring a node as “async” in Elm, and RX.Zip also does very similar job as NOT declaring a node as “async” in Elm. So I wonder if the async and ordering issues are fundamental for FRP.

### Gabe Fierro

Elm is an FRP language with asynchronous support that aims to be well-suited for developing GUIs: this includes the graphical layout
of an application as well as the logic behind the UI. One of the core principles in ELM is the notion of a "signal",
which is a value that changes over time. Instead of using wall-time ticks, Elm's notion of time comes from the progression
of events that occur when program input changes. This is very closely tied to the FRP notion that new "state" only comes
from the outside. Signals, which are represented as nodes in a DAG, are connected by means of function definitions, which are
just a thinly hiddden `map` and `reduce`.

At first, Elm seemed limited in its abilities; I wasn't sure that the relational model was able to capture all of the
intricacies and positioning usually required in modern web applications, but then it occured to me that much of modern
web programming can be thought of as a dataflow. The problem that Elm purports to solve -- that of asynchronously combining
multiple long running actions and using them as a trigger for other actions -- is addressed at least in part by Promises,
which were released in 2014, after the ELM paper. Events occur in the real world, which then trigger actions in a UI, and
that user-input state propoagates through the GUI. FRP seems like a nice fit for this.

I liked the trick with making asynchronous actions "synchronous" by having them emit dummy signal states when events
actually occur. This preserves the pipelined semantics of the DAG. The paper does not go into much detail on the runtime
for enabling this operation though. I believe they mention that their initial implementation was slow, but recent
benchmarks have shown elm to be much faster than similar JS frameworks such as React and Angular. I wonder what
they did in their compiler/runtime to make that work, and how proressive it looks.
