# Monsoon: an Explicit Token-Store Architecture
## Gregory Papadopoulos (MIT), David Culler (Berkeley)

### Erik Krogen
This paper was pretty exciting for me, as it explained something very close to hardware, but was simple enough and well enough explained that despite this I feel I was able to get a good grasp on its workings. It seems like one of the biggest flaws is the pipeline bubble resulting from the first portion of a two-token operation, but I wonder if this is fundamental - it seems that today's processors do significantly more complex things than dealing with a problem like this. 

Questions I am left with: What prevented this from working? Has anyone ever built practical systems with dataflow hardware, if not why and if so why isn't it more prevalent? What types of programs would be difficult to write on this type of processor, and is programming on this system generally natural? How well would e.g. Bloom / Datalog work if implemented on this type of hardware instead of on traditional processors? With improvements in hardware speed and design knowledge, could this work better today?

### Chenggang Wu

The paper presents explicit token store (ETS), a simplified approach to dataflow execution. The motivation is that traditional dataflow architectures tolerate communication delays and support generation and coordination of parallel activities directly in hardware. However, the complexity of these mechanisms is high and may further complicate the program mapping. On the other hand, expecting to achieve a perfect mapping (and thereby high performance) is non-realistic for many applications. So the authors strive to find a design of processors that sustain high performance even when the matching is imperfect. The main idea for ETS is that it dynamically allocates storage for token in sizable blocks. The architecture can directly execute dataflow graphs, and is simple to build and manages to make dataflow execution easier to understand.

Since the addressing capability is restrictive, I wonder how this would affect the architecture. The paper claims that this is fine as there is exactly one instruction executed for each action on the dataflow graph, but will there be other potential side-effects? Also, I am curious if there are systems today that inherit the idea behind ETS.

