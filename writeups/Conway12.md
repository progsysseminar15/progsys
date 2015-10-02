# Logic and Lattices for Distributed Programming

## Neil Conway, William Marczak, Peter Alvaro, Joseph M. Hellerstein, David Maier

### Johann Schleier-Smith

The authors seek a powerful and programmer-friendly approach to distributed programming, one that makes is easier to build software that is reliable, yet runs with concurrence. For me, one sentence summarizes what this paper is about: “database theory on monotonic logic provides a powerful lens for reasoning about distributed consistency.”

Alternative approaches force developers into ad-hoc solutions, application concurrency schemes based on communication and coordination primitives, overly restrictive models, such as database transactions, or limited scope of guarantees, as in CvRDTs. This work generalizes previous uses of monotonic logic in distributed systems, building upon the Bloom language and adding the notion of arbitrary lattices as generalization of monotonic set operations. The authors claim that this approach allows much stronger guarantees about global program behavior than what other techniques can provide. I found it particularly useful to review the relationship between morphisms and monotone functions, as well as how non-monotone functions can be integrated in a Bloom calculation (coordination required).

I am still interested in understanding in more details how lattice logic can help a computation achieve parallel speedup, as the examples in this paper seem to focus on replication. I am also curious about the requirement for an idempotent “least upper bound” operations—while this requirement follows from the definition, there are cases where it turns out to be cumbersome (say in *count* or *sum*), and I wonder whether the language could provide guarantees to make these non-morphism monotone functions easier to work with. I also have a question about the shopping cart example: I am interested in understanding how we can recover from a client crash, whether it is easy to express client knowledge as optional, so that we fall back on using coordination it is absent.

### Erik Krogen

Having no prior experience with Bloom, this paper was very informative in a number of ways. The concept of a monotonic lattice is fascinating, especially the notion of how to formulate a shopping cart as a semi-bounded lattice. The distinction of morphisms and monotones was especially interesting and subtle. One key takeaway was that, though most applications do not seem monotonic initially, they can be formulated in such ways that they do become monotonic - I think the shopping cart is an excellent and unexpected example of this. 

I am left with some questions about how exactly some of these things work, though that is likely from a lack of knowledge of Bloom. How is state replicated across systems? Do they trade entire maps and merge? In the KVS case study, it seemed that each key was stored on all systems; how easy would it be to also provide partitions? Are there fundamental limitations on what types of systems can be implemented using BloomL and if so, can we categorize them? Can we perhaps provide interfaces on top of BloomL that abstract away the necessity of working with monotonic lattices only while still providing the same guarantees -- basically, how general is this?


### Xinghao Pan

The CALM theorem states, approximately, that all logically monotone programs are confluent (invariant to message reordering and retry) and hence eventually consistent.
"Monotonicity" here refers to the property that adding things to the input can only increase the ouput (see [http://www.bloom-lang.net/calm/](http://www.bloom-lang.net/calm/)).
In that sense, a monotone program is a progressive program that never has to retract statements made.
"Eventual consistency" refers to achieving a final state of the system without coordination and independent of the message ordering.
Thus, very vaguely and informally, progressive (monotone) systems do not require coordination to achieve (eventual) consistency.

Bloom and Bloom^L are languages that encourage developers to write monotone (sub-)program with minimal non-monotone coordination.
Bloom^L extends Bloom by extending the notion of sets with monotone union operations to join semi-lattices -- a class of objects that have a well-defined commutative, associative, idempotent merge functions.

The CALM / Bloom work appears to be very similar to the I-confluence work, with some subtle differences (see [Peter Bailis's comment](http://www.bailis.org/blog/when-does-consistency-require-coordination/)):
1. I-confluence consistency refers to the C of ACID, and is concerned with ensuring validity at every intermediate state
2. CALM / Bloom consistency refers to the final eventually consistent deterministic state
Interestingly, both make use of commutative, associative, and idempotent merge functions to reconcile divergent replicas, which seems to suggest the key to minimizing coordination is the ability to merge divergent states.
As noted in the I-confluence paper, it would be interesting to dig deeper into how the two are related and in what concrete ways they differ.

Another thing that I would be interested to examine in greater detail is the implementation of rule evaluations / fact derivations.
The implementation described in the paper works well if the set of rules is small, which allows for a fast application to the new facts derived in the last time step.
However, in some of my own work, the set of rules can be very large (in the order of the size of the data set), but generate few new facts at each time step.
In such scenerios, rules are redundantly evaluated in the paper's implementation.
An alternative implementation that triggers new rules to be evaluated could instead be much more efficient.
Also possible would be to prioritize rules and only evaluate a subset at each time step.

### Chenggang Wu

This paper presents Bloom^l, an extension to Bloom that supports lattices and enables CALM analysis to the program consisting of arbitrary lattices. Although CRDT is the first to propose a conflict-free data model, it has a scope dilemma: on one hand, a small module such as a set makes the lattice properties easy to analyze, its functionality is quite restrictive and provide little usage in practice. On the other hand, although a large CRDT provides rich functionalities and application guarantees, the burden of ensuring lattice properties are left entirely to the programmers. I think the key idea for Bloom^l is that it allows the programmers to compose safe lattices into a bigger and bigger monotonic program. This bottom-up approach makes constructing a larger confluent program significantly easier than before, and because programmers just need to keep track of whether each small lattice component satisfies the correctness properties, this approach is also more scalable.

One concern that occurs to me is that not all distributed programs can be written in a confluent fashion, and when we write a program, it is unclear whether this program is expressible as a composition of lattices. Therefore, it would be nice to have a mechanism that helps users decide which portion of the program is expressible as lattices and which is not. 

### Gabe Fierro

This paper presents an extension to the Bloom language that gets around some of
the previous limitations of the language in regards to composing distributed
systems that achieve consistency without coordination.  At the core of the
approach is CALM analysis which asserts that distributed inconsistency can only
occur at a point of order. The Bloom language previously only contained support
for monotonically increasing sets which make constructing these types of
systems easy. BloomL extends the data types to use other monotonic structures
(while giving users the ability to define their own) and also works through how
to compose confluent programs from smaller, more grokkable confluenct components.

I do not know much about the Bloom language, but I would be interested to see
how much of its abilities are facilitated by operating in a dynamic language
such as Ruby, and if confluent guarantees and CALM analysis could be offered by
static analysis tools in languages perhaps better suited to large scale web
services: Ruby is not bad, but there are compiled languages (e.g. Go and Scala)
that can achieve much better performance than a dynamic language. The examples
presented in this paper are also very limited, and I wonder how possible it is
to express larger systems (even involving trivial features such as user
accounts) that incorporate logic that is not necessarily confluent, or even
incorporating security features. How well does the Bloom approach scale when
applied to real-world systems?  Or is Bloom mainly intended as a prototyping
language?

### Ethan J. Jackson

The CALM theorem shows that any system which is "monotonic" (i.e. collects
information but never changes its mind about old results) is necessarily
eventually consistent.  The goal of this paper is to extend Bloom with the
ability to express programs which conform to CALM, and automatically verify
that they do, in fact, conform.  In doing so, one could create distributed
systems which are provably correct, without the need for expensive coordination
algorithms.

This work in some ways reminds me of the work done by the PL people w.r.t to
advanced languages which have strong type systems (ML, Haskell, and the like).
It's quite a useful exercise to consider what the theoretically "right way" to
build these systems is.  I suppose the biggest question that this paper raised
for me, is how to approach the problem from the other end of the spectrum.  How
do we bring some of the insights of CALM analysis to existing traditional
programming languages, much in the way some of the features of Haskell (type
inference, etc) made their way into traditional languages, scala being the best
example.   It's not entirely clear to me how to achieve this, but if it's
possible the impact would be dramatic.
