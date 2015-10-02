# Logic and Lattices for Distributed Programming

## Neil Conway, William Marczak, Peter Alvaro, Joseph M. Hellerstein, David Maier

### Johann Schleier-Smith

The authors seek a powerful and programmer-friendly approach to distributed programming, one that makes is easier to build software that is reliable, yet runs with concurrence. For me, one sentence summarizes what this paper is about: “database theory on monotonic logic provides a powerful lens for reasoning about distributed consistency.”

Alternative approaches force developers into ad-hoc solutions, application concurrency schemes based on communication and coordination primitives, overly restrictive models, such as database transactions, or limited scope of guarantees, as in CvRDTs. This work generalizes previous uses of monotonic logic in distributed systems, building upon the Bloom language and adding the notion of arbitrary lattices as generalization of monotonic set operations. The authors claim that this approach allows much stronger guarantees about global program behavior than what other techniques can provide. I found it particularly useful to review the relationship between morphisms and monotone functions, as well as how non-monotone functions can be integrated in a Bloom calculation (coordination required).

I am still interested in understanding in more details how lattice logic can help a computation achieve parallel speedup, as the examples in this paper seem to focus on replication. I am also curious about the requirement for an idempotent “least upper bound” operations—while this requirement follows from the definition, there are cases where it turns out to be cumbersome (say in *count* or *sum*), and I wonder whether the language could provide guarantees to make these non-morphism monotone functions easier to work with. I also have a question about the shopping cart example: I am interested in understanding how we can recover from a client crash, whether it is easy to express client knowledge as optional, so that we fall back on using coordination it is absent.

### Erik Krogen

Having no prior experience with Bloom, this paper was very informative in a number of ways. The concept of a monotonic lattice is fascinating, especially the notion of how to formulate a shopping cart as a semi-bounded lattice. The distinction of morphisms and monotones was especially interesting and subtle. One key takeaway was that, though most applications do not seem monotonic initially, they can be formulated in such ways that they do become monotonic - I think the shopping cart is an excellent and unexpected example of this. 

I am left with some questions about how exactly some of these things work, though that is likely from a lack of knowledge of Bloom. How is state replicated across systems? Do they trade entire maps and merge? In the KVS case study, it seemed that each key was stored on all systems; how easy would it be to also provide partitions? Are there fundamental limitations on what types of systems can be implemented using BloomL and if so, can we categorize them? Can we perhaps provide interfaces on top of BloomL that abstract away the necessity of working with monotonic lattices only while still providing the same guarantees -- basically, how general is this?
