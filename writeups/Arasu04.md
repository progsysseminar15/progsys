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

<<<<<<< HEAD
### Chenggang Wu

This paper first provides an exact semantics for general-purpose declarative continuous queries over streams and relations. Specifically, the semantics is based on formal definitions for stream and relations, mappings among them, and relational query language. Then, it proposes CQL, a language that instantiates the black box in the abstract semantics using SQL as the relational query language and window specifications derived from SQL-99. The paper also compares CQL with other languages for continuous queries over streams and relations, and shows that CQL is more expressive than other languages.

I am interested in knowing more about how performant CQL is. For now, it seems that the query plan generator is fairly simple and uses hard-coded heuristics to generate query plans. It would be interesting to investigate possible performance optimization opportunities under this abstraction.


### Gabe Fierro

This paper defines the CQL language for defining continuous queries over streams and relations. The paper makes a point
to define the semantics of the language, and show that they have a complete language that covers the semantics
of other streaming query languages. The language formally defines both streams (a bag of elements with a tuple
and a timestamp) and relations (mapping from the set of timestamps to some set of tuples that match a relation)
and operators that go between these: stream-to-relation, relation-to-stream and relation-to-relation. 

It seems to me that CQL is a superset of many of the other systems and data-concerned flows we've looked at. Queries
that operate on data "up to" a certan time T almost feel ike freezes, and windows feel like chunked operators
that can help synchronize multiple streams of data. Of course, these can operate over logical timestamps rather
than wall time: the only requirement is that T is discrete and ordered.

The system makes use of monotonicity: the examples given in the paper make the assumption that all data essentially just "grows",
with the exception of the user membership query which requires negation. The system does not have a notion of
time being overwritten (though multiple values can have the same timestamp), which lets the system materialize
streams as a set of "diffs" over time: this is the Istream operator. Because the relational operators and the language
are well-defined, it becomes possible to statically reason about what queries are monotonic and which aren't. The ability
to reason about how a system will behave is a feature shared by the Out of the Tar Pit system, which also uses the idea
that data is either derived or unchanging. Likewise, CQL has both base and derive streams and relations.


### Johann Schleier-Smith

TO me the highlight of CQL is it user-friendly approach. It stays close to familiar SQL, restricting stream-to-stream operations, which instead are expressed as a sequence of operations: stream-to-relation → relation-to-relation → relation-to-stream. This not only allows the implementation to take advantage of well established relational know-how, but also stays with concepts familiar to users. The authors do a good job of addressing potential objections, showing how carefully designed defaults allow concise expression of most common queries. I also found it interesting that relations and relational operators can be expressed as stream and stream operators, and I wonder whether the converse is true.

Windows provide the connection between streams and relations, and so are central to CQL. *Now* and *Unbounded* seem to be particularly common window specifications, though CQL also provides tuple-based and time-based ranges. I wonder, though, whether these window specifications are really the right way to get a handle on time. In particular, I feel that the windowing specification should be responsive to what is happening in the data, for example by recognizing user sessions.

This paper also provides a number of practical implementation notes. One design choice that I appreciated was the encapsulation of system functions and streaming functions, together with relational operators, in an extended operation set. This allows a uniform high-level description, a plan including materialization and network io alongside relational processing.

My sense is that CQL inherits many of the strengths and weaknesses of SQL. It successfully adapts the relational model to streaming data, but it doesn't go that far beyond it. For simple queries this is fine, but I continue to wish for greater code re-use and more manageable ways of expressing complex programs.