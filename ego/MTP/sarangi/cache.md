*axioms of cache coherence*
- write serialization, all the writes to the same address are seen in the same order by all the threads.
- write propagation, a write is eventually seen by all the threads. (never lost)

a bus is a set of copper wires that can support only one sender at a time; however, we can have any number of receivers.

a cache needs to get control of the bus before sending request.
we assume that bus controller allocates the bus fairly.

in shared state, caches do not supply data memory does.
only M,O,E states share data. else need to select one of the shared state caches to send and prevent other from sending. overhead.

at most one cache is allowed to have the responsibility to supply data.

*write-update*

![[share_invalid.png|400]]

in write-update protocol, order of writes may not be preserved if two caches writes to the same block at the same time.
make write atomic to fix this.

![[invalid_modified.png|400]]

in write-update, there can be multiple caches that have the same block in modified state. they just broadcast when writing to this modified state.

- RdX read miss
- WrX write miss
- Broadcast

![[MSI_updates.png|500]]

drawbacks,

- for every write, we need to broadcast, this increases bus traffic and power utilization.
- on write broadcast, every cache needs to check whether it contains the cache block or not.

![[coherence_terms.png|500]]


*write-invalidate*

- single writer and no readers
- multiple readers and no writer

the logic for avoiding multiple responses is that once a response is sent, the rest of the sister caches that have the data discard their responses. since only one cache can use the bus at a time.

![[MSI_invalid.png|400]]

bus
![[MSI_invalidate_bus.png|400]]


the order of writes to a single location is the order in which we enter the M state across the caches.

write operation needs to be performed first before sending a copy to other cache 
###### MESI

![[MESI.png|500]]

when in shared state if its the only copy, no need to broadcast invalidation request to others.

usefull in codes where we access a lot of blocks that are not shared across the caches, coz we can silently write without sending invalidate msgs on the bus.

however, both MSI and MESI suffers from the problem of frequently writing to the lower level, when we have transition from $M$ to $S$ state. write-backs are required so that we can perform seamless evictions from the $S$ state.

*arbitration*
the process of choosing one entity among a plurality of interested entities. multiple sister caches compete among each other to send a response to the requesting cache. there is thus a need for arbitration.

current approach is that all of them create their responses, and the moment they see a response on the bus sent by a sister cache, they discard their responses. this is time consuming, and requires additional hardware support.
###### MOESI
if a cache contains a block in the owner state then it is by default responsible to forward the data. this ensures that caches do not compete among each other to supply data to the requesting cache and they don't have to prepare responses and discard them.
aim is to eliminate write-backs as far as possible.

![[MOESI_bus.png|500]]

![[MOESI.png|700]]

need two temporary states, in case the owner gets evicted. according to our rules only owner transfers blocks but there may be other caches that may have block in shared state. wait for owner, else search sister caches in shared states else fetch from lower level.


---

many i/o devices write directly to memory, in case of write-back we need to check each level if the block is dirty.

on DMA request.
invalidate cache lines if not dirty, if dirty, write-back and then invalidate

###### incl_vs_excl_cache
*inclusive cache*
if a cache line exists in L1/L2, it must also exist in L2/L3.
lower levels are superset of upper levels.

if L2 evicts A, then only L1 needs to evict A, L3 does not need to invalidate.

*exclusive cache*
a cache line exists in only one cache level.

*non-inclusive cache*
there is no rule requiring duplication or exclusion.
the same line may or may not exist at multiple levels.

the hardware decides dynamically based on replacement policies, coherence, and performance.

