# Hekaton: SQL Server's Memory-Optimized OLTP Engine
## Diaconu, Freedman, Ismert, Larson, Mittal Stonecipher, Verma, Zwilling
## Microsoft

### Erik Krogen

Hekaton introduces a number of new ideas: a mixture of in-memory and on-disk tables that can be accessed in the same manner, versioning with parallel garbage collection, native code compilation, commit dependencies, and continuous checkpointing. All of these are very cool features that seem to solve their intended problems very well. Commit dependencies make a lot of sense - transactions abort a very small portion of the time, so continuing optimistically under the assumption that a transaction will succeed seems reasonable. The continuous checkpointing also seems very intelligent - the main problem (that I can see) of checkpointing is the necessity to do some expensive operation all at once, but their clever data/delta stream system solves this issue.

This system reminds me a lot of Postgres - versioned records, garbage/archive collection, etc., except that there are a number of features here that should allow it to be more successful, like garbage collection happening continuously and in parallel, as well as the fact that it's working in memory so things should be fast. So this begs the question - if you wanted a fully historical database like the original imagination of Postgres, could this system be easily and modified to achieve that while still being efficient? It seems that it lends itself to this. 
