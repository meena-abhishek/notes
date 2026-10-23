**Thermal problems in HBM**: HBM is a stack of DRAM dies bonded vertically. Because it's 3D and densely packed, heat doesn't escape easily. Heavy memory traffic on one channel heats up not just that channel's banks, but _vertically and horizontally adjacent_ banks too (heat diffuses in 3D). When any bank crosses a temperature threshold, the memory controller punishes the whole channel by dropping it into a low-power state — which kills performance.


The core insight of this paper: **these two problems are entangled, but everyone in prior work treats them separately.**
If you move an application to a cluster with low cache contention (good for performance), you might route its memory traffic into an HBM channel that's already hot (bad for temperature) — and vice versa. Fixing one blindly can break the other.

A **manycore processor** has dozens to hundreds of cores on one die. Managing every core individually (per-core DVFS, scheduling) doesn't scale in hardware — the power delivery network and clock distribution become too complex. So chips are organized into **clusters**: groups of cores (here, 8 cores/cluster, 8 clusters = 64 cores) that:

- Share a single **Voltage-Frequency (VF) domain** — all cores in a cluster run at the same clock speed and voltage. You can't throttle one core in a cluster without throttling all of them.
- Share a **Last-Level Cache (LLC)** — e.g., an 8MB L3-like cache shared by all 8 cores in the cluster.
- Share one **memory controller**, which is wired to one HBM channel.

### Why per-core management doesn't scale

Suppose you have **128 cores**.

If every core had its own:

- Voltage regulator (VR)
- Clock generator/PLL
- Temperature sensor
- DVFS controller
- Power management logic

then you would need **128 copies** of all this hardware.

This creates several problems:

- **Area overhead:** Voltage regulators, PLLs, and controllers occupy silicon area.
- **Power overhead:** Control circuitry itself consumes power.
- **Cost:** More hardware means higher manufacturing cost.
- **Routing complexity:** Delivering many different voltages and clocks across the chip becomes extremely difficult.
- **Control complexity:** The OS or firmware must constantly decide the optimal frequency and voltage for every core independently.

For example, if every millisecond the system had to compute the best voltage/frequency for 128 cores, that's much more work than managing 8 groups of 16 cores.

Each core has private L1/L2 caches, but the LLC is shared. If Core A (running App X) and Core B (running App Y) are in the same cluster, their memory accesses both compete for LLC space and bandwidth. App X's frequently-used data might get evicted by App Y's accesses. This causes **extra cache misses**, which means more trips to main memory (HBM), which is slower — hence "slowdown" or "performance degradation" relative to running alone.

This is fundamentally an _interference_ problem — it can't be predicted by looking at either application in isolation; it depends on the _combination_.
### The general principle
This is really just an instance of a broader idea in phase-based systems management: **a resource manager's reaction time (its epoch/sampling interval) needs to be short relative to the phase durations of the thing it's managing, or its decisions are made on stale/blended data and its interventions cost more than they save.** MTCM was designed and validated against applications whose phase behavior is slow-changing relative to 10ms — so it transfers well to "coarse-grained" DNN structures (huge layers, big batches) but would need modification (e.g., decoupling decision-frequency from commit-timing, as I suggested) to handle workloads with many short-lived phases, like a network with lots of small layers.



*why maintaining temp of HBM is more critical*
- **TCC (Thermal Control Circuitry)**: manycore-side hardware that, on a temperature violation, throttles the offending cluster(s) to minimum VF level. Blunt and reactive.
- **DTM (Dynamic Thermal Management)** on HBM: when a bank/channel overheats, the channel transitions to a **low-power state**, essentially becoming unavailable for a period. This is even more brutal for performance because it doesn't just slow the channel, it stalls memory requests

The whole point of _proactive_ resource management (like MTCM) is to **never trigger these reactive mechanisms**, because once triggered, they overcorrect and destroy performance — exactly what the motivating example (Fig. 2) shows.

#### 2.8 Neural networks as fast "surrogate models"

This is a classic ML-for-systems pattern: **expensive physics/simulation model → generate training data → train a cheap neural net that approximates its input-output mapping → use the NN at runtime instead of the expensive model.** The NN sacrifices some small accuracy (measured via MAPE — Mean Absolute Percentage Error) for orders-of-magnitude speedup (milliseconds → microseconds). This is essential because a resource manager evaluating "what if I do X?" for several candidate migrations, every 10ms, simply cannot afford to run a full thermal simulator each time.


- **Sniper** [35]: a fast, interval-based multicore _performance_ simulator — models cache hierarchies, timing, IPC, etc., without doing full micro-architectural simulation of every instruction (much faster than cycle-accurate simulators like gem5 in detailed mode).
- **HotSpot** [15]: thermal simulator (see above).
- **CoMeT** [31]: an integrated toolchain (built by some of the same authors' group) that combines Sniper + HotSpot + power models for 2D/2.5D/3D processor-memory systems — this is the actual experimental platform used.
- **CACTI-3DD** [33]: a modeling tool that estimates energy-per-access and latency for 3D-stacked DRAM (HBM) based on its geometry/tech parameters — used to get the "9.51 nJ per access" energy figure, which converts memory-access counts into power for the thermal model.



- Setup: 64-core chip, 8 clusters, 8-channel HBM, cluster `n` → channel `n` mapping. Clusters 2,4,5,7,8 run static background compute jobs (no memory traffic, at min frequency) — essentially "noise" clusters that don't matter for the memory story. Clusters 1, 3, 6 run `cholesky`, `water.sp`, `radix` respectively, at a _fixed_ medium frequency (2.8GHz) so the manycore itself never violates its thermal limit — isolating the story to the **HBM only**.
- A 4th app, `lu.cont` (memory-heavy, from SPLASH-2), needs a 4th cluster's worth of cores, and must co-locate with one of the three existing apps.
- **Scenario 1** (`lu.cont` + `cholesky` on Cluster 1): both are memory-heavy → severe _cache contention_ (18.25% slowdown) AND thermal violation (85°C on channel 1, spreading to adjacent channels 2-4, because contention concentrates heavy traffic onto one channel and its neighbors).
- **Scenario 2** (`lu.cont` + `water.sp` on Cluster 3): `water.sp` is nearly memory-silent, so almost _no_ cache contention (–1.48%, i.e., basically no slowdown — this is the _best_ performance option). But `lu.cont`'s heavy traffic goes through channel 3, which is **vertically adjacent** to channel 1 (still hot from `cholesky`). Combined heat stacks up → still a thermal violation, even though contention was minimized!
- **Scenario 3** (`lu.cont` + `radix` on Cluster 6): moderate contention (8.61% slowdown, worse than Scenario 2) but channel 6 is physically/thermally _far_ from channel 1, so no violation occurs anywhere.


optimal doesn't always = deployable

### The full chain, every time a boost is considered

Every 1ms DVFS epoch, for a cluster that currently has thermal headroom (no violation, safe to consider boosting), MTCM does this **before** actually raising the frequency:

1. **Predict the new memory access rate at the higher frequency** — call `NNm` with the app(s) currently running on that cluster and the candidate higher `f`. This is necessary because running faster generally means the cores generate memory requests faster too (more work retired per second → more loads/stores issued per second), so you can't just assume last epoch's access count still applies.
2. **Convert that predicted access count into power** — multiply by the per-access energy figure from CACTI-3DD (9.51 nJ/access in their setup), added onto the relevant channel's banks in `Pbanks`. Core power at the new frequency is handled too (via the standard V-F-power relationship, not `NNm` — that part's more analytically tractable since it doesn't depend on contention/behavior).
3. **Feed the updated `Pcores` and `Pbanks` maps into `NNcores` and `NNhbm`** to get predicted steady-state temperatures for cores and HBM banks at this hypothetical higher-frequency state.
4. **Check both against `T_thresh`.** Only if _neither_ subsystem is predicted to violate does MTCM actually commit the frequency bump.


You don't need to check _whether lowering VF helps_ — lowering frequency always reduces power, which always tends to reduce temperature, that direction is not in doubt. What you need to check is **how much** it helped, i.e., **whether it helped _enough_** to actually clear the predicted violation. The check isn't "did this make things worse?" — it's "is this sufficient, or do I need to throttle further?"

---

Our novel contributions are as follows.
1) We train two lightweight NN models to accurately
predict the impact of task migration on the system
performance considering cache contention in clusters,
and the impact of changing the VF levels on the HBM
access behavior of applications.
2) We train two lightweight NN-based thermal models,
engineered to accurately predict the temperature of the
clustered manycore and the HBM within the stringent
time constraints at runtime.
3) Enabled by our fast and accurate NN models, we present
MTCM, the first resource management technique that
jointly mitigates cache contention while enforcing ther-
mal safety on the systems with an integrated clustered
manycore and HBM by means of the task migration and
DVFS.

Each cluster k accesses memory through a dedicated controller MCk configured to access a specific channel ch

![[mtcm_training_method.png|500]]

training:
- first run application alone on one cluster and extract performance monitoring counters.
- run with another application 4 threads each, one core one thread. see diff in PMCs.

At runtime, however, an application
could start its execution on one cluster, then be migrated to
a new one in the following migration epoch, since MTCM’s
migration policy will be invoked periodically every 10 ms.
Therefore, there is a need to predict the postmigration PMCs
of Appa over only 10 ms-long windows, not over the whole
shared execution of applications. To achieve this goal, we
perform time-based slicing of execution traces from the task
migration scenarios of the previous phase at a 1 ms granularity.

, we select the model topology
that balances prediction error and inference time.

By mixing power values from the
individual executions and distributing them randomly across a
larger set of cores and memory banks, we construct synthetic
multiapplication power maps.

3) Single-Label Model Optimization: Aligned with the goal
of ensuring thermal safety, essentially predicting potential
thermal violations based on the power traces, we adjust our
training data, so that the models only predict the maximum
temperature across either the cores or memory banks, instead
of all the temperatures.

find candidate application -> evaluate performance (>mape) -> enforce thermal safely

The current temperature of all the
cores and the postmigration Pcores are passed to our NNcores
model to predict the new steady-state temperatures of the
cores on the manycore.

all the experiments. MTCM’s DVFS policy takes 24.6 µs on
average to boost or throttle clusters, representing 2.46% of
the adopted 1 ms DVFS epoch on a single core. MTCM’s
migration policy takes 105.5 µs on average to find a migration
option and apply it, equivalent to 1.05% of the 10 ms migration
epoch on a single core. Therefore, the benefits of our proposed
MTCM can be obtained with a negligible overhead on our
target 64-core processor.