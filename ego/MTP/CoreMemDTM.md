need to coordinate DTM of cores and memories to reduce overcompensation
and minimize large penalties by DTM.

stall-balanced core DVFS policy, distributed cooling

premise:
due to inter-dependence of core and memory, a DTM decision taken for one might alleviate temperature for other and reduce overheads.

latches are used in front of FU, so the input remains constant during computation.

DTM for processor slows down compute-intensive cores, while DTM for memory slows accesses to heated channels, which in turn stalls memory-intensive cores, further reducing processor temperature and causes overcompensation.

we balance the slow down for different cores, due to which the overall workload completes faster.
Stall balancing also leads to using a less aggressive DVFS,
improving performance.




