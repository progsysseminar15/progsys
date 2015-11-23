# Monsoon: an Explicit Token-Store Architecture
## Gregory Papadopoulos (MIT), David Culler (Berkeley)

### Erik Krogen
This paper was pretty exciting for me, as it explained something very close to hardware, but was simple enough and well enough explained that despite this I feel I was able to get a good grasp on its workings. It seems like one of the biggest flaws is the pipeline bubble resulting from the first portion of a two-token operation, but I wonder if this is fundamental - it seems that today's processors do significantly more complex things than dealing with a problem like this. 

Questions I am left with: What prevented this from working? Has anyone ever built practical systems with dataflow hardware, if not why and if so why isn't it more prevalent? What types of programs would be difficult to write on this type of processor, and is programming on this system generally natural? How well would e.g. Bloom / Datalog work if implemented on this type of hardware instead of on traditional processors? With improvements in hardware speed and design knowledge, could this work better today?

### Chenggang Wu

The paper presents explicit token store (ETS), a simplified approach to dataflow execution. The motivation is that traditional dataflow architectures tolerate communication delays and support generation and coordination of parallel activities directly in hardware. However, the complexity of these mechanisms is high and may further complicate the program mapping. On the other hand, expecting to achieve a perfect mapping (and thereby high performance) is non-realistic for many applications. So the authors strive to find a design of processors that sustain high performance even when the matching is imperfect. The main idea for ETS is that it dynamically allocates storage for token in sizable blocks. The architecture can directly execute dataflow graphs, and is simple to build and manages to make dataflow execution easier to understand.

Since the addressing capability is restrictive, I wonder how this would affect the architecture. The paper claims that this is fine as there is exactly one instruction executed for each action on the dataflow graph, but will there be other potential side-effects? Also, I am curious if there are systems today that inherit the idea behind ETS.

### Xinghao Pan

(The following review was written with little understanding of hardware / processors or historical context.
In particular, I found the introduction difficult to understand, and it was only later in the paper that I started to understand and gain some bits of appreciation of the problem.) 

This paper addresses the complexity faced by dataflow architectures, which the authors argue are due to implicit token store and matching.
The proposed alternative is to make the token store explicit:
upon invocation of a function, an activation frame is explicitly allocated.
The activation frame is explicitly referenced by each token, so that mapping is immediate.
Monsoon is a multiprocessor that incorporates an ETS.

The description of Monsoon's parallelism sounds similar to that of Volcano's horizontal and vertical parallelism.
Non-interfering operators can be assigned to different processing units (horizontal) and pipelining is used within each processing unit (vertical).

Another passing comment in the paper that struck me was: "... by using dataflow graphs as a programming methodology, we are guaranteed that all execution schedules produce the same result", which sounds like CALM monotonicity / determinism yet again, in a different context.
My understanding of the statement is that, as long as things flow forward (progressive), we can achieve deterministic outcomes without coordination.

Dataflow processors seemed like a good idea. Were there any reasons that they did not work out? If so, are there lessons we should learn from when developing our own progressive systems?


### Johann Schleier-Smith

The promise of dataflow architectures is a compelling one. The model for the computer, rather than being limited to the step-wise operations of the Von Neumann architecture, allows for parallel execution whenever the inputs of an operator are ready. Monsoon differs from previous dataflow architectures, which also serve as targets for the *Id* programming language, is that the compiler makes explicit the intermediate states that the processor must maintain during program operation. The work of defining these *activation frames*, which provide storage for matching in-flight operations, those for which input operands remain outstanding, is taken on by the compiler, ahead-of-time, rather than in hardware during execution. Instead of hash lookups, the hardware needs only deal with linear offsets. This makes sense to me, as does any change that helps keep the lower layers simple.

I found elements of the discussion of the explicit token store (ETS) architecture at the end of Section 2 particularly interesting. The authors note that by ignoring presents bits, the execution model reduces to that of multiple imperative control threads. At the same time, by adjusting the state transitions, they implement a variety of dataflow primitives. To me, this indicates that the hardware described offers a great deal of flexibility, that it presents a powerful element for managing concurrency.

One thing I continue to wonder about is whether some of the principles of Monsoon could be applied to dataflow processing at a higher level. The activation frame and explicit token store architecture that the authors describe in a hardware realization might also be attractive for a software dataflow system. Perhaps operations and inputs would be at a coarser level of granularity, but the notion of triggering an execution when all of the bits are ready is a compelling one. In practice, such a system would also require attention to further distributed systems concerns, such as failure handling, but I am curious to explore whether the core ideas might translate.
