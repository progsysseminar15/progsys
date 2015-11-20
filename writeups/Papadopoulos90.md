# Monsoon: an Explicit Token-Store Architecture
## Gregory Papadopoulos (MIT), David Culler (Berkeley)

### Erik Krogen
This paper was pretty exciting for me, as it explained something very close to hardware, but was simple enough and well enough explained that despite this I feel I was able to get a good grasp on its workings. It seems like one of the biggest flaws is the pipeline bubble resulting from the first portion of a two-token operation, but I wonder if this is fundamental - it seems that today's processors do significantly more complex things than dealing with a problem like this. 

Questions I am left with: What prevented this from working? Has anyone ever built practical systems with dataflow hardware, if not why and if so why isn't it more prevalent? What types of programs would be difficult to write on this type of processor, and is programming on this system generally natural? How well would e.g. Bloom / Datalog work if implemented on this type of hardware instead of on traditional processors? With improvements in hardware speed and design knowledge, could this work better today?
