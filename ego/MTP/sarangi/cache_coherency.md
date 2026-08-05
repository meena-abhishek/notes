*coherence*
is about ordering of operations from different processors to the same memory location

*consistency*
is about ordering of all memory operations from different processors to different memory locations


shared cache becomes bandwidth bottleneck (contention)

write-update protocol, push an update to all copies
write-invalidate protocol, ensure there is only one copy

interconnect is important, how do we decide who is closest neighbour with valid block to get from. need a protocol for this, else flooding bus, increase latency.

on a read,
- if local copy is invalid, put out request
- if another node has a copy, it returns it, otherwise memory does.

on a write,
- read block into cache as before
- updates caches or invalidate caches

snoopy bus,
single point of serialization for all memory access

directory,
- single point of serialization per block
- directory tracks which caches have each block
- directory coordinates invalidation and updates

![[directory_protocol.png|500]]

if zero cores have block, then we don't need to send broadcast to all other cores for invalidation

![[directory_ex1.png|500]]
![[directory_ex2.png|500]]

every access needs to go through directory

invariant, only one processor can have the exclusive access

----

shared data provides communication among the processors through read and writes of the shared data.

NUMA,
non-uniform memory access
physically distributed memory but logically acts as shared memory 

if local memory doesn't have data, then other cores memory controller sends data via interconnect.
![[distributed_memory.png|500]]

---

on a read miss, cache need to get control of the bus.

