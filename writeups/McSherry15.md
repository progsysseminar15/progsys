# Differential dataflow
## Frank McSherry

### Erik Krogen

To be honest, I didn't really understand a lot of what McSherry was saying here... The overall idea of having incremental computation defined e.g. over an (epoch, iteration) ordering with dependencies on both (i-1, j) and (i, j-1) makes sense, I got pretty thoroughly lost in his implementation details. He goes through the work of defining a difference as (collection - sum of previous differences) but I don't see why this is helpful - isn't the point to build up a full collection as a result of differences? It seems to be that he needs to calculate a new difference from a previous difference (to pass on to subsequent operators) - maybe this is what he's doing and I'm just missing it. Although I can follow his example, I don't really see what benefits I'm gaining from it--I actually believed in the idea of differential dataflow more *before* reading the example...

In any case, the idea is interesting, and hopefully after some discussion I'll be able to see its usefulness more. 
