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
