#CQL: A Language for Continuous Queries over Streams and Relations
##Arvind Arasu, Shivnath Babu, and Jennifer Widom

###Xinghao Pan
In this paper, the authors identify a lack of a precise language for expressing continuous queries.
Their goal is thus to define precisely the abstract semantics of CQs, and to develop a concrete language that instantiates the abstract semantics.
Streams and relations are defined as data types describing relational data evolving (via inserts and deletes) over time.
(Note that streams and relations are equivalent in the definition.)
Operators convert between the two types (stream-to-relation and relation-to-stream) or are n-ary functions from relations to relations.
A crucial fact that makes everything work is that queries are also defined by time, which makes the queries sensible and well-defined.

In the concrete lanugage CQL, conversion of streams to relations are essentially windowing methods, and relations are converted to streams by using the Rstream operator (which subsumes Istream and Dstream).
Relations-to-relation operators are defined using SQL.

It seemed almost obvious to me (in the context of this class) that the solution to CQs would be exactly to timestamp everything.
I think, however, that this highlights the power of the progressive system abstraction.
Reasoning about any constantly evolving system (which is basically all systems) is hard in general,
but once we recognize the progressive nature, that time only flows forward, and annotate everything with time, all moving parts are pinned down and can be easily reasoned about.

Another thing that occurred to me is that all the (non-monotone) progressive systems we have seen need some way to track the passage of time.
In LVars we had freezing, in Naiad we had epochs and iterations, and in CQL there are windows and heartbeats.
With freezing, epochs, and heartbeats, there was a need to stop or quiesce the system prior to issuing the punctuation.
However, there is an alternative that we saw in the Bloom^L shopping cart -- under the right conditions, it is possible to allow a punctuation to be received prior to some inputs.
Understanding and quantifying the pros and cons of each approach could lead us to developing better ways to mark progress in a progressive system.

### Chenggang Wu

This paper first provides an exact semantics for general-purpose declarative continuous queries over streams and relations. Specifically, the semantics is based on formal definitions for stream and relations, mappings among them, and relational query language. Then, it proposes CQL, a language that instantiates the black box in the abstract semantics using SQL as the relational query language and window specifications derived from SQL-99. The paper also compares CQL with other languages for continuous queries over streams and relations, and shows that CQL is more expressive than other languages.

I am interested in knowing more about how performant CQL is. For now, it seems that the query plan generator is fairly simple and uses hard-coded heuristics to generate query plans. It would be interesting to investigate possible performance optimization opportunities under this abstraction.

