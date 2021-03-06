# SEDA: An Architecture for Well-Conditioned, Scalable Internet Services
## Matt Welsh, David Culler and Eric Brewer

### Gabe Fierro
SEDA, or staged event-driven architecture, is a proposed design for concurrent Internet services, centered around the idea
of using explicit queues to connect stages of operation. The authors argue that with this explicit delineation of
stages and isolation of capabilities, it becomes much easier to reason about the interaction between these stages
and use that information to dynamically scale the system to changes in the load. SEDA uses message passing between
stages. Each stage is an event-driven execution container, and many of these can be spun up in parallel
using something more like a thread pool. Messages are placed into queues before each component, and becomes queues
can be bounded, SEDA can easily apply backpressure.

SEDA feels very much like some of the dataflow systems we've been looking at. Many of these continuous query systems
(like Flux) connect stages of execution. Flux, like SEDA, uses introspection into queues to handle fault tolerance
and consistency in a distributed setting. It is interesting to consider this work in face of recent popularity
of event-driven servers such as NodeJS, which use asynchronous APIs to adapt programs to an event-driven, "state-machine"
execution model. In my mind, this paper starts to bridge the gap between dataflow/CQ systems and actual infrastructure
composition. I wonder how universal the staged-pipeline model is for designing systems, and if there are certain
systems that can't be built using such a model. Staged-pipelines sidestep the issue of mutable state, but it feels
like one could have immutable data being passed through a SEDA system.

### Yifan WU

Really out of my depth here with web servers, so mostly questions:
Questions
- There appears to be a dicotomy of concurrency: one based on thread and one on events, and SEDA
innovates by combinging the best of both worlds: tHe system automatically adjusts the thread pool
size based on queued events, and this is achieved by "staging". Welsh mentioned in his [retrospective]
(http://matt-welsh.blogspot.com/2010/07/retrospective-on-seda.html) that if it were designed today,
they would further decouple stages from the queues and thread pools. I'm a bit confused about what
the stages help achieve --- why does this have to be batched?
- I'm not sure how this relates to other "scheduling" processes such as distributed systems scheduler
(e.g. Omega, Sparrow) -- I imgaine they are vastly different, but are there shared ideas?
- One idea from the paper that relates well to the line of papers we have been reading at is exposing
the bottlenecks more transparently, and in SEDA's case by looking at the queues. In Bloom we see the
drive to expose challenging concurrency issues to the application developer.
- It seems that Apache Webserver is still the most popular (50%+), and it's mostly multiple
thread/process without notions of events (I might be wrong!), and I'm wondering why SEDA hasn't
"won" given all the benefits it seems to have.


### Xinghao Pan
SEDA is a staged event driven architecture that attempts to deliver the higher performance of event-driven designs while providing a easy programming and debugging interface.
A SEDA design is composed of event stages, each with its own event queue and controllers.
Event stages explicitly separate core application logic from thread management and scheduling.
Resource controllers manage resources of each stage, shielding programmer from the complexity of performance turning (reminds me of FRP model from Out of the Tar Pit).
There is an explicit control boundary between stages, constraining threads to a given stage

It was unclear what the difference is between SEDA and structured event queues of section 2.4, or whether SEDA is a general implementation of structured event queues. Hopefully the discussion will make this clearer to me.

The authors made an interesting comment in the conclusion that "in traditional event-driven design it is much more difficult to trace the flow of events through the system", whereas in SEDA message-oriented communication between stages establishes explicit orderings.
The ease of programming is attributable to the *explicit ordering*; this, however, may be achieved without SEDA, by having events events carry around timestamped traces of their passage through the system. SEDA encodes the ordering naturally, but is not the only means to capture ordering.

SEDA also looks very much like some of the later data flow papers that we have read, if we think of events as data flowing through the system.
I wonder if the converse is true, that is, are all data flows also representable as event models?
If so, could we implement data flows in SEDA, and what performance advantages might we get?
If not, what types of data flows are not event models, and what makes them special?

###  Ethan J. Jackson
This paper reminds me a lot of click in a lot of ways.  They're both
essentially stream processing systems, one operating on packets and one
operating on requests.  Also, both use a processing graph of independent stages
to increase code reuse and modularity.  While this is a fairly common point in
the design space, its efficacy depends a lot on the details of the system
you're running.  For packet processing, as data rates increased, the cost of
CPU cache misses between threads, and synchronization overhead on the queues
has largely made the approach unworkable.  That said, the NFV movement has
forgotten this lesson and is soldiering on with a similar architecture, though
with a VM at each stage this time.

All and all, I think this architecture didn't win as it sits in a midpoint
between two more useful extremes.  Most web servers don't actually need to
perform very well as they sit behind many layers of caches and CDNs.  For
those, the pre-forked apache architecture is the easiest for developers to
maintain and reason about.  On the other end, for high performance HTTP proxies
and various other middleboxes, the performance hit of the architecture is too
great making it not viable.


### Johann Schleier-Smith

SEDA describes relatively early work toward providing good performance under load, focusing on the challenges of scalable internet services. This is a challenge that I can appreciate, and the point is probably best made by comparing the throughput curve in Figure 2 to that in Figure 4. We see that even as concurrency goes up, system throughput stays flat. Latency must go up,  as we reach the system capacity constraint, but we the system continues to do useful work at a constant rate. This is an important operational characteristic, for when throughput degrades under load a service becomes difficult to manage reliably.

This architecture described in this paper consists of a series of processing stages linked by queues. Where possible, it avoids thread pools, instead operating in an event loop style. This is a pretty reasonable approach, though I would have liked to see a deeper discussion of the tradeoffs between threading and event processing (memory requirements, overhead involved in switching tasks, relation to CPU architecture). Also, I have seen both threaded and event loop systems exhibit degraded throughput under load, and I would have liked to see more on how to configure queues so to avoid such degradation. One strategy that works is to give higher priority to tasks deeper in the pipeline, and to reject requests at the source when we know that the system will be unable to complete them within acceptable time bounds.

Today, many most production systems continue rely heavily on threaded models. With appropriate admissions policies (connection limits) one can keep them in the regime of good throughput. Asynchronous event loops are important for connection multiplexing ([C10K problem](http://www.kegel.com/c10k.html)), but I don't see them broadly displacing threaded execution models, which provide both greater ease of programming, and often better performance as well.

### Chenggang Wu

This paper presents SEDA, an architecture for supporting well-conditioned services that are capable of serving concurrent requests. The motivation is that neither the thread-based model nor the event-driven architecture is capable of providing robust services under overloaded conditions. The former would incur overhead caused by having too many threads (lock contention), and the latter would be difficult to guarantee the main thread to be non-blocking (due to interrupt, page fault, etc). Therefore, this paper combines the two approaches and introduces the concept of staged event-driven architecture. One major benefit of SEDA is that it offers robust performance and fairness under heavy-load. The other benefit is that it makes the load and resource bottlenecks explicit in the application programming model. By doing so, it allows the application to inspect and tune the load based on the concurrency demand.

The paper was written fifteen years ago, when multi-core was rare and Linux thread scalability was poor. As of today, these are no-longer true, and therefore we may need to re-evaluate some of the design decisions made back then. Another concern is that SEDA is not well-optimized for normal load conditions. In fact, Apache and Flash could outperform SEDA under normal load. So it would be interesting to investigate further the performance bottleneck of SEDA under light/ normal loaded conditions.

