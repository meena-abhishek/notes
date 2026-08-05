applications fighting over shared cache - cache contention
cores overheating - thermal throttling

these two problems degrade performance of multi-threaded applications on manycore chip organized into clusters (groups of core sharing cache)
fixing one problem in isolation can make the other worse.

a single monolithic shared cache across hundreds of cores doesn't scale (wire delay, coherence traffic)

clustering creates a resource management problem: the OS/runtime must decide which applications go on which clusters, and this affects both cache contention and heat generation.

in a clustered many-core processor like amd zen 3,
cache contention does not directly generate much additional heat inside the cache. instead, it increases execution time and memory activity, which raises the total energy consumed and creates thermal hotspots around the contending cores and memory subsystem.

Multiple applications
        │
        ▼
Shared LLC contention
        │
        ├── More LLC conflicts/evictions
        ├── More cache misses
        └── More DRAM accesses
                │
                ▼
Longer execution time
Higher dynamic power
Higher memory-controller activity
                │
                ▼
Higher chip temperature
                │
                ▼
Thermal limit exceeded (e.g., 80°C)
                │
                ▼
DVFS / Thermal throttling
                │
                ▼
Lower frequency → Lower performance



If the combined working sets exceed the cache capacity or repeatedly evict one another's data, cache contention occurs.

inclusive vs non-inclusive cache [[cache#incl_vs_excl_cache]]

if we migrate applications purely to balance cache contention, ignoring temperature. this can push chip temperature above safe limit, triggering TCC and erasing the gains.

smartCM - a runtime control loop

NN predicts the performance gain and a thermal model predicts migration safety.

a second control loop does cluster-level DVFS to squeeze out any remaining thermal headroom.

cache contention,
when two applications run on cores in the same cluster, they share the LLC. If both have large working sets, they evict each other's cached data repeatedly, causing more cache misses -> more trips to DRAM -> slower execution. this is contention. it is hard to predict analytically because it depends on the fine-grained temporal interleaving of two programs memory access patterns.
not just how much cache they want, but when they want.

cluster-level DVFS is preferred,

cores inside a cluster shares L3 cache, NoC router, power delivery network, clock network.

chaning frequency independently for each core creates problems:
clock syncronization, voltage island complexity, inter-core communication latency.

designers often create multiple DVFS domains. each cluster can run at different frequency.

migration changes contention and temperature distribution.

**Runtime (every 10 ms, per Algorithm 1):**

1. **Find migration candidates:** any application AoI (Application of Interest) that's sharing a cluster with ≥1 other app is a candidate (it _might_ benefit from moving away from contention).
2. **For each candidate, check every other cluster as a destination:** does it have enough free cores?
3. **Predict post-migration performance** using the NN (called three times — for AoI, for the apps left behind (γ), and for the apps already on the destination (ω) — see below).
4. **Compute ∆Perf** = predicted total IPS after / current total IPS before − 1. Only keep this migration candidate if ∆Perf exceeds a threshold `I` (a safety margin against model noise — you don't act on a predicted 0.5% gain because that's within the model's own error band).
5. **Thermal safety check:** using the _actual_ physics-based RC thermal model (not the NN) — estimate temperature if this migration were executed; discard if it violates T_crit.
6. **Pick the single best surviving migration** (highest ∆Perf) and execute it.

But Section VI's actual experiments are explicitly an **open system**: 40 applications arriving at 5 different Poisson rates specifically to sweep "various system utilization values." At low arrival rates, the chip is mostly idle, so response time ≈ execution time (queueing delay ≈ 0). At high arrival rates, cores become scarce, applications may have to wait for a cluster to free up before they can even start, and **that waiting time is exactly the kind of system-level inefficiency that cache contention and thermal throttling make worse** (throttled cores finish slower → hold onto their cores longer → new arrivals wait longer). So response time is the more complete "how good is this system under realistic load" metric, and it's _why_ the x-axis of Fig. 5/6 is arrival rate — the whole point is to show how the two policies diverge as congestion increases.

---

technique that mitigates cache contention under thermal constraints.

#cache_partitioning
**Cache partitioning** is an architecture-level technique for controlling cache contention by giving each application (or core, or thread) a _dedicated, isolated portion_ of a shared cache, instead of letting all applications freely compete for the same cache space.

In a normal shared cache (like the LLC in this paper's clusters), all co-running applications insert and evict lines from the _same_ pool of cache "ways" using a shared replacement po.licy (e.g., LRU). If one application has a large, cache-hungry working set, it can evict another application's data even if that other application only needed a small amount of cache.
Allocation can be **static** (fixed at boot/config time) or **dynamic** (re-adjusted at runtime based on which app needs more cache,

cache partitioning is architecture-level, migration/mapping is system level.

runtime application mapping through migration to balance/mitigate cache contention throughout execution time.

only using LLC/DRAM access are not sufficient to estimate CC.

cache contention on a cluster can result in different slowdowns in the performance of different applications. This investigation shows that the relation between cache contention and application performance is complex.

###### 1. Access count measures volume, not destructive interference
Contention happens when two applications' cache lines **map to the same sets** and evict each other. An application can generate a huge number of LLC accesses while mostly **hitting on its own already-resident data** (high hit rate, high reuse) — those accesses don't touch new lines, don't evict anyone else's data, and don't cause misses. So "high access count" can just mean "this app talks to the cache a lot," not "this app is aggressively stealing cache space from its neighbor."

application migration can improve the overall system performance if contention on the destination cluster is less than contention on the source cluster, and if this migration is thermally safe.
Hence, there is a need to predict migration impact
on the chip temperature and on the overall system performance
considering cache contention on both source and destination clusters.

we develop an
NN model to predict the impact of an application migration on the
performance of all applications running on source and destination
clusters, by taking as input the most relevant performance counters
that indicate cache contention.
This model enables us to build a
smart cache contention mitigation technique, called SmartCM, that
periodically invokes the model to evaluate the impact of potential
migrations on the overall system performance. The impact of these
migrations on the chip temperature will be also checked, so that the
migration expected to lead to the highest overall performance without
violating Tcrit is performed. 
Additionally, DVFS is employed on the
clusters to exploit available thermal margins and to avoid potential
thermal violations that may occur due to changes in the execution
phases of applications.


**IPS = Instructions Per Second** — literally, how many instructions application `k` retires (completes) per unit time while it's running.

Why call it a **proxy**? Because "performance" is an abstract goal (how fast/well is this application doing?), and IPS is a _measurable stand-in_ for it. The logic is:

- If an application is running at high IPS, it's making rapid progress through its program → it's executing fast.
- If cache contention causes stalls (the core sits idle waiting for data from a slow LLC/DRAM access instead of retiring instructions), IPS **drops** — the same application, same amount of code to run, but fewer instructions completed per second.

So IPS directly captures the _symptom_ of contention: contention doesn't change _how many_ instructions an app needs to execute, but it changes _how fast_ they get executed, and that shows up immediately as a lower IPS. This is why it's the natural signal to feed into the NN (Section IV) — it's a clean, directly-measurable readout of "how much is this app suffering right now."

_"We're going to use IPS as our measurable handle on performance, use it to drive contention-reducing migrations, and report success in terms of response time — but every migration must also pass a thermal safety check, because ignoring that would be self-defeating."_

Since AoI, γ and ω could be executing at different
V/f levels on their corresponding clusters s and d, frequencies fs and
fd are also included in the feature set.

This logic runs independently for **every** eligible AoI in the system (every application sharing a cluster with at least one other app).


feature tuple $(PC_{AoI},\ PC_γ,\ PC_ω,\  f_s,\  f_d)$ and the label, $IPS_{AoI}$′.

![[smartCM.png]]

Cache contention isn't constant throughout a program's run — it changes as the program moves through different phases (e.g., a compute-heavy loop vs. a memory-heavy loop). If you trained on one big average number per application, the model would lose all of this phase-dependent behavior. So instead, they chop every trace into fixed **10ms windows** (matching the actual runtime migration epoch — so the training data granularity matches how the model will actually be queried at runtime) and treat **each window as one independent training row**.

Here's the subtle part: instead of just taking "the next 10ms" of the AoI+ω trace, they find the window in the AoI+ω trace where AoI executes **the exact same instruction range** (`ins_AoI_1` to `ins_AoI_2`) as it did in slice `S_t`.


 
 *The training-time meaning of each slot*

Recall from Section IV-A, the training feature tuple was always:
```
(PC_AoI, PC_γ, PC_ω, fs, fd) → label IPS_AoI'
```
During training, this tuple always meant something very specific:

- **Slot "self"** = the application whose _future_ IPS we're predicting (AoI)
- **Slot "partner-before"** = whoever `self` was co-running with **before** migration, on the **source** cluster (γ)
- **Slot "partner-after"** = whoever `self` will be co-running with **after** migration, on the **destination** cluster (ω)
- **f-before, f-after** = frequency of the source cluster, frequency of the destination cluster

So the network never learned "co-runner A" and "co-runner B" as interchangeable, symmetric slots — it learned one specific asymmetric relationship: **"partner I'm leaving behind" vs. "partner I'm arriving to."**

 Now check the three runtime calls against that exact template
```
IPS'_AoI ← PM(fs, fd, PC_AoI, PC_γ,  PC_ω)
IPS'_γ   ← PM(fs, fs, PC_γ,  PC_AoI, 0)
IPS'_ω   ← PM(fd, fd, PC_ω,  0,      PC_AoI)
```

