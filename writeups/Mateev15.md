# Read-Log-Update: A Lightweight Synchronization Mechanism for Concurrent Programming
## Alexander Matveev, Nir Shavit, Pascal Felber, Patrick Marlier

### Xinghao Pan
RLU is a synchronization mechanism that extends RCU, and allows multiple concurrent reads with multiple concurrent writes.
The key idea is to exploit timestamps -- both reads and writes have timestamps read from a global clock; writes increment the global clock, so that reads are partitioned into pre- and post-write, which enables readers to read the correct version of the object (either the global pre-write, or thread-local post-write version).
The gliobal version is updated only when all pre-write reads have completed.
Write-write synchronization is achieved by locking.

My takeaway from all this is that timestamps once again make everything simple to reason about.

Much of the work here reminds of the concurrency control papers that we read earlier.
In particular, Berstein & Goodman described mechanisms that use T/O for r/w synchronization, and blocking/locking for w/w synchronization, which appears to be exactly what RLU does.
Is there any reason to reinvent the wheel then?
Why do we not apply and evaluate standard concurrency control mechanisms in this setting?

In the same context, it may be possible to use weaker isolation levels (e.g. RAMP) for some applications.
Write-write conflicts may also be resolved if updates are monotone (CALM).
