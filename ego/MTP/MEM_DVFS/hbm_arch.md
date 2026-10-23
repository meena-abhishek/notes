**HBM4** (newly ramping in 2026, JEDEC standard JESD270-4):

- Capacity: up to 64 GB per stack at the high end, though the first wave of 12-high HBM4 actually delivers 36 GB per stack (same as HBM3E) — the bandwidth jumps first, capacity increases come later with taller stacks [Kynix](https://www.kynix.com/Blog/hbm3e-vs-hbm4-2026-specs-performance--supply-guide.html)[Gpusmith](https://gpusmith.com/articles/en/pdfs/hbm3e-vs-hbm4-ai-gpus.pdf)
- Bandwidth: the JEDEC floor is 2 TB/s at 8 Gb/s per pin, but real shipping products are already exceeding that — Micron's HBM4 delivers over 2.8 TB/s per stack at speeds greater than 11 Gbps, and Samsung's HBM4 runs at 11.7 Gbps and can push to 13 Gbps, roughly 22–35% faster than HBM3E's ceiling [HBM4 vs HBM4E vs SPHBM4: Standards, Bandwidth, Capacity and Packaging Explained | AiCybr Blog +2](https://aicybr.com/blog/hbm4-vs-hbm4e-vs-sphbm4-memory-standards)
- Some vendor roadmaps target as high as 3.3 TB/s peak per stack [Kynix](https://www.kynix.com/Blog/hbm3e-vs-hbm4-2026-specs-performance--supply-guide.html)

**Capacity** is how much data can be _stored_ at once — the size of the warehouse.

**Bandwidth** is how fast data can be _read out and put back_ — the speed of the conveyor belt moving boxes in and out of that warehouse.

The key thing: reading data doesn't use it up. The same 36 GB or 64 GB sitting in memory can be read over and over again, many times per second. Bandwidth measures how many times per second you can move data across the connection, not how much total data exists.

So if you have 1 TB/s of bandwidth and only 64 GB of capacity, it just means:

**1,000 GB/s ÷ 64 GB ≈ 15.6** — you could theoretically read through the _entire_ contents of that memory over 15 times every single second.

That's actually exactly the situation GPUs are in. For AI workloads like running a large language model, the same weights (stored once) get read from memory repeatedly for every single token the model generates — thousands of times per second. High bandwidth is what lets the processor re-read that same data fast enough to keep up, without capacity needing to grow at all.


#why3d

Traditional/planar DRAM is called "2D" because each memory cell's transistor and capacitor are laid out **side-by-side on a single silicon plane** — the classic 1T1C (one-transistor-one-capacitor) cell, built using standard planar CMOS processes. All the cells in an array sit in the same X-Y plane on one die. Scaling to more capacity means shrinking that 2D layout and packing more cells into the same plane, or making a bigger die.

This is different from **3D NAND flash**, where "3D" means the memory cells themselves are stacked vertically _within a single die_ — layers of cells grown on top of each other in the Z-axis on the same silicon, dramatically increasing density per unit area without shrinking the cell.

- **3D NAND** — cells stacked vertically _within_ one die (true 3D cell architecture)
- **3D-stacked DRAM/HBM** — multiple _2D_ DRAM dies stacked and TSV-connected (die-level 3D, cell-level still 2D)

![[dram_vs_hbm.png]]

#memory_wall

Memory _capacity_ did scale nicely with Moore's Law — you can fit exponentially more bits on a chip because that's mostly a density/transistor-count problem. But _latency_ is governed by physics that don't shrink the same way: charging and discharging a DRAM cell's tiny capacitor, activating a row, driving a signal down a wire — these have hit diminishing returns. So the real tension is:

- **Compute speed** ↑↑↑ fast (historically)
- **Memory capacity** ↑↑↑ fast (also tracks Moore's Law)
- **Memory latency** ↓ very slowly (the actual bottleneck)

Bandwidth is a bit of a middle case — it's scaled reasonably well because engineers found ways to get more bits per second (wider buses, more channels, higher clock speeds, 3D stacking) even without fixing the underlying latency problem.
"The memory wall refers to the growing performance gap between processors and memory. Historically, CPU speeds scaled aggressively, but DRAM access latency — the time to actually fetch a piece of data — improved much more slowly, because it's bound by physical limits like capacitor charge time and signal propagation, not just transistor density. So even though memory _capacity_ and even _bandwidth_ have scaled well with Moore's Law, latency hasn't kept pace, and processors increasingly stall waiting on memory. That gap is the memory wall, and it's the reason the industry has invested in things like deep cache hierarchies, prefetching, wider memory interfaces, and — especially relevant to Micron — technologies like HBM, which attacks the problem by massively increasing bandwidth and moving memory physically closer to the compute die, even if per-bit latency itself doesn't fundamentally change."

A few things that make this land well in a Micron-specific interview:

- **Distinguish bandwidth from latency.** Interviewers at a memory company will notice if you conflate them. HBM primarily solves a _bandwidth_ wall, not a _latency_ wall — it doesn't make one random access faster, it lets you move much more data per second via wide parallel interfaces.
- **Name concrete mitigations**, since "strategies to bridge the gap" is literally what they'll want to hear about: cache hierarchies, prefetching, out-of-order execution, wider/multi-channel memory buses, 3D die stacking (TSVs) as in HBM, processing-in-memory (PIM) or near-memory compute, and higher-level techniques like batching/locality-aware software.
- **Tie it to why Micron's products exist.** Mentioning that DRAM roadmaps (DDR, LPDDR, HBM) are literally the industry's ongoing answer to the memory wall shows you understand the business relevance, not just the textbook definition.

----

Let's work through this systematically — there are several distinct questions bundled together, so I'll take them in a logical order.

## Why do embedded/real-time systems need AI models at all?

Modern automotive, avionics, and space systems increasingly need to do things that are extremely hard to specify with traditional rule-based logic: object detection and classification from cameras/LiDAR, sensor fusion across many noisy inputs, anomaly/fault detection, predictive maintenance, autonomous decision-making (e.g., collision avoidance). These tasks — recognizing a pedestrian, classifying terrain, detecting an anomalous vibration pattern — are fundamentally pattern-recognition problems where the "rules" aren't cleanly expressible by hand but _can_ be learned from data. That's exactly what neural networks are good at, which is why these domains have moved from purely deterministic control software toward embedded AI models for perception and decision layers.

## What do AI models actually require from memory?

Two distinct properties, and it's worth separating them clearly (we touched on this early in our conversation):

- **Capacity**: model weights (potentially millions to billions of parameters) plus intermediate activations need to be stored somewhere accessible to the compute engine.
- **Bandwidth**: this is the dominant constraint for most inference and training workloads. Every layer of a neural network requires reading large weight matrices and activation tensors from memory, doing the compute (mostly matrix multiplications), and writing results back — repeatedly, for every inference pass or training step. Modern AI accelerators (GPUs, NPUs) can perform far more arithmetic operations per second than they can be _fed_ data for — this is the well-known "memory wall" applied specifically to AI workloads, sometimes phrased as being "bandwidth-bound" rather than "compute-bound."

## Why do real-time/critical systems specifically need high bandwidth (not just capacity)?

A few compounding reasons specific to real-time systems:

- **Hard deadlines**: A real-time system (e.g., automotive perception feeding a braking decision) has a strict time budget — the entire sense→infer→act pipeline must complete within milliseconds. If memory bandwidth is the bottleneck, the neural network inference itself takes longer, directly eating into that deadline. Missing the deadline in a safety-critical context isn't just "slow," it can be a certification/safety failure.
- **High sensor data rates**: modern sensor suites (multiple high-resolution cameras, radar, LiDAR) generate continuous, large-volume data streams that must be ingested, fused, and processed continuously — this is a sustained high-throughput requirement, not a burst.
- **Multiple concurrent models/tasks**: real systems often run several AI models simultaneously (e.g., object detection + lane detection + driver monitoring), each competing for the same shared memory bandwidth, compounding the total bandwidth demand on the system.
- **Traditional DRAM (DDR/LPDDR) bandwidth ceiling**: as you found from the earlier numbers in our conversation, LPDDR5X tops out around a few hundred GB/s across a full multi-channel configuration — this is the "commonly used DRAM memories struggle to provide" line from your abstract. It's not that DDR/LPDDR can't work at all, it's that the ratio of achievable bandwidth to the AI compute engine's actual appetite has become badly mismatched, especially as models grow more sophisticated even at the edge.

## How does HBM specifically help here?

HBM directly attacks the bandwidth mismatch, using exactly the mechanisms we discussed earlier in this conversation:

- Very wide interface (1,024/2,048-bit) instead of DDR's narrow bus → dramatically more bits moved per cycle
- Stacked dies with TSVs placed right next to the compute die (2.5D/3D packaging) → shorter signal paths, lower energy per bit, higher achievable bandwidth without pushing per-pin frequency to extremes
- Result: HBM3E/HBM4 deliver roughly 1–3+ TB/s _per stack_, versus tens to a few hundred GB/s for LPDDR/DDR — directly closing the gap between what the AI accelerator can compute and what memory can feed it.

For real-time/embedded specifically, this translates to: the perception/decision pipeline can pull in and process sensor data and run inference fast enough to meet the hard deadline, and can do so even while running multiple concurrent models, because the bandwidth ceiling is no longer the limiting factor.

## Does HBM help inference, training, or both?

**Both**, but the emphasis and mechanism differ slightly:

**Training** (this is where HBM became essential first, e.g., NVIDIA H100/H200/B200, AMD MI300 series):

- Training involves processing huge batches of data through forward _and_ backward passes, repeatedly, across many layers, with large gradient and optimizer-state tensors also needing to be stored and moved
- This is extremely bandwidth-hungry and somewhat more tolerant of raw latency (training runs for hours/days, so a bit of extra per-access latency matters far less than sustained throughput over the whole run)
- HBM's primary value in training is **sustaining extremely high, continuous data movement** — keeping the massively parallel compute cores (thousands of GPU cores) fed so they're not stalled waiting for weights/activations/gradients

**Inference** (increasingly relevant for edge/embedded, and the context your abstract is really about):

- Inference is typically more latency-sensitive per individual request (you need _this_ frame's object detection result _now_, not just high average throughput), and often more memory-capacity-constrained per request depending on model size
- Bandwidth still matters a lot, especially for large models — every token/frame of inference requires re-reading the model's weights, and if those weights are large relative to how much useful work is done per read (this is the "arithmetic intensity" concept), bandwidth becomes the direct limiter of inference speed
- HBM helps inference primarily by **reducing the time spent re-reading weights/activations per inference pass**, which directly shortens end-to-end inference latency

## Does HBM actually reduce inference latency, and how?

Yes, but it's important to be precise about _the mechanism_, since HBM's core spec win is bandwidth, not access latency (recall from way earlier: HBM doesn't fundamentally lower the individual random-access latency of a DRAM cell — the physics of activate/sense/restore are the same).

Here's how higher bandwidth still translates into lower _effective_ inference latency:

- Most inference workloads (especially for large models) are **bandwidth-bound**, not latency-bound in the classical single-access sense — the dominant cost isn't "how long does one random access take," it's "how long does it take to stream all the weights needed for this layer through the compute engine." If bandwidth is higher, that streaming completes faster, directly shortening the layer's compute time, which shortens the whole inference pass.
- Since neural network inference involves reading _huge, mostly sequential/structured_ chunks of data (weight matrices, activation tensors) rather than scattered random single-word accesses, this is exactly the access pattern where bandwidth (not per-access latency) dominates total time — which is precisely the scenario HBM is optimized for.
- So: HBM reduces inference _latency_ as an end-to-end, wall-clock-time-per-inference measure, even though it doesn't reduce the fundamental _access latency_ of a single random DRAM read. This distinction — bandwidth-bound throughput improvement vs. reducing intrinsic access latency — is worth stating explicitly if this comes up in an interview, since conflating the two is a common mistake.

**One-line summary tying it all together:** "Embedded/real-time AI workloads need to move huge, structured chunks of weight and sensor data through the compute engine within hard deadlines, and traditional DRAM's bandwidth increasingly can't keep pace with what modern accelerators can compute — the 'memory wall' applied to AI. HBM addresses this directly through a very wide, short, stacked interface that delivers far higher sustained bandwidth than DDR/LPDDR, which helps both training (feeding massively parallel compute continuously over long runs) and inference (streaming weights/activations fast enough to meet per-request latency deadlines) — and even though HBM doesn't reduce the intrinsic access latency of a single DRAM cell, it reduces effective end-to-end inference latency because these workloads are dominated by bandwidth-bound, bulk data movement rather than scattered random accesses."


---

This turned out to be a genuinely current and fast-moving topic — the answer as of today (September 2026) is a bit different from what you'd have found even a year ago. Let me go question by question.

## Do modern HBM stacks use microbumps?

Yes — and this is actually a live, newsworthy point right now. Most HBM in 2026 AI packages still uses microbump/TSV stacking, sitting on a CoWoS-style interposer next to the GPU. This wasn't a foregone conclusion: a January 2026 JEDEC decision raising the HBM package height ceiling from 720 µm to 775 µm let 16-high HBM4 be assembled with microbumps after all, deferring hybrid bonding's memory debut to HBM4E and HBM5. So HBM4 — the current cutting edge — actually stayed on microbumps rather than moving to hybrid bonding, mainly for cost/economic reasons: with HBM4 pad pitch at 10 µm, moving to hybrid bonding at that pitch does not yet make economic sense. SK hynix in particular has planned accordingly, sticking with advanced mass-reflow molded underfill for 16-high HBM4 while keeping hybrid bonding as a backup.

## What are TSVs made of?

TSVs (Through-Silicon Vias) are copper-filled — the process involves etching a via hole through the silicon die and filling it with copper via electroplating (SK hynix's assembly flow is described as going through "silicon etch and TSV copper fill"). Copper is used for the same reasons it's used everywhere else in chip interconnects: high electrical conductivity and good thermal conductivity, both of which matter enormously for a vertical stack.

## Why can't TSVs "go out of dies and connect with other dies" directly?

I think what you're getting at is: why do you need a separate bonding step (microbumps or hybrid bonding) instead of the TSV just poking through and touching the next die's TSV directly? A few physical reasons:

- **Mechanical/manufacturing precision**: a TSV is etched and filled while the wafer is still thick and mechanically stable. Dies are then thinned dramatically (down to tens of microns) so a TSV can actually reach through. You can't reliably align and physically fuse thousands of microscopic copper columns from one wafer directly onto thousands from another wafer with atomic-level precision without an intermediate structure — you need some interface layer to absorb small misalignment, surface roughness, and thermal expansion differences.
- **Electrical/surface contact requirements**: raw copper left exposed at a via's end can oxidize and won't make a clean, low-resistance electrical joint if just pressed against another raw copper surface without proper surface preparation.
- **This is exactly why microbumps (or hybrid bonding) exist as the connecting layer**: a microbump is a small solder ball sitting on top of a TSV's exposed pad, specifically designed to be reflowed (melted and re-solidified) to form a reliable, tested, mechanically compliant joint to the corresponding pad on the neighboring die. It's the "connector," not a fundamental limitation on TSVs.

## Is Cu-to-Cu bonding used by the industry, and how does it work?

Yes, but with an important nuance: it's in high-volume production on logic chips already (Intel's Foveros Direct at 9 µm pitch in its Clearwater Forest Xeon 6+ server CPU, TSMC's SoIC at 6 µm), but for HBM specifically, it has just been postponed — HBM4 stayed on microbumps rather than adopting it, precisely because of that JEDEC height-limit change and current economics at HBM4's 10 µm pad pitch.

**How Cu-Cu hybrid bonding works, when it is used:**

- The top and bottom surfaces of adjacent dies are polished to atomic-level flatness using chemical mechanical planarization (CMP)
- Pick-and-place assembly happens at room temperature
- Then an anneal above 200°C forms both SiO2-to-SiO2 (oxide) and Cu-to-Cu (metal) bonds simultaneously — this is exactly why it's called "hybrid": you're bonding two different material interfaces (dielectric and metal) at once
- The result is a direct copper-to-copper connection with no solder, no bumps, and no gap-fill/underfill material needed — a genuinely gapless interconnect

**Why the industry wants this for HBM eventually**: it enables far tighter interconnect pitch (single-digit microns vs. ~20 µm for microbumps, vs. ~100 µm for older solder bumps), which means more connections per unit area, thinner stacks, and — relevant to your next question — dramatically better thermal conduction, since there's a continuous solid copper/oxide path instead of bumps sitting in an underfill polymer (a much worse thermal conductor than copper).

## Does heat get trapped between layers in modern HBM stacks — and if so, how?

Yes, and this is a well-documented, serious engineering problem. HBM's three-dimensional stack traps heat and amplifies thermo-mechanical stress, and internal thermal resistance rises sharply beyond 12-layer stacks.

**Why it happens:**

- Each DRAM die generates heat during operation, but only the die at the very bottom of the stack sits near the package substrate/heat spreader; dies stacked above it have to conduct their heat _down through_ every layer beneath them (or occasionally up through a heat spreader on top) before it can actually leave the package.
- The interconnect layer between dies (microbumps + underfill/mold compound, in today's mainstream approach) is a much worse thermal conductor than solid silicon or copper — underfill polymers in particular are thermal insulators compared to metal. So every microbump/underfill layer the heat has to cross acts like a partial thermal barrier, and with many layers stacked (12-high, 16-high), those small resistances add up substantially.
- This is worsened by the fact that HBM sits immediately next to a very hot compute die (GPU/accelerator) on the same interposer, so it's also picking up lateral heat from its neighbor, not just generating its own.

**How the industry is addressing it (multiple competing approaches right now):**

- Hybrid bonding, once adopted, directly helps here too — hybrid bonding in 16-Hi HBM structures achieves approximately 47% lower thermal resistance compared to microbumps, attributed to the continuous thermal conduction path enabled by Cu-Cu direct bonding, since you eliminate the poorly-conducting underfill/bump layers.
- Even without hybrid bonding, vendors are pursuing distinct interim fixes: SK hynix's approach embeds a high-thermal-conductivity, electrically insulating cooling component within the hot die-to-die PHY area to create a dedicated heat path, targeting a reduction in thermal resistance of over 30%; Samsung's approach moves DRAM next to the processor with a copper heat spreader; and Micron is refining the base-die circuit design for efficiency.

So to directly answer: yes, heat absolutely gets trapped between layers in stacked HBM, primarily because the interconnect material between dies conducts heat far worse than the silicon/copper it separates, and this compounds with every additional layer — which is exactly one of the key motivations pushing the industry toward hybrid bonding for future HBM generations, even though HBM4 itself just delayed that transition by a generation for cost/yield reasons.{}

---

Good news — this turned out to be a very well-covered topic with plenty of real, citable detail. Here's the elaboration, followed by direct links.

## Why the D2D PHY is the actual hotspot

Multiple sources converge on the same explanation: the Die-to-Die Physical Layer (D2D PHY) — the ultra-high-speed interface linking the processor and HBM stacks — continuously moves terabytes of data per second, and as thousands of signaling lanes and billions of transistors switch at extremely high frequencies, switching losses, leakage current, and electrical resistance generate substantial heat. This is compounded by the fact that the HBM sits immediately next to an already very hot GPU/accelerator die on the same interposer, so heat concentrates in a small area from two directions at once. When it gets bad enough, the system triggers thermal throttling — automatically reducing clock speeds and voltage — which directly eats into the performance gains HBM was supposed to deliver in the first place.

## SK hynix's iHBM (the one I mentioned)

- Rather than relying on the older approach of dissipating heat indirectly through the core die, iHBM integrates cooling elements (ICEs) — described as electrically non-conductive, thermally conductive silicon-based material — directly into the D2D PHY, right where the heat is concentrated, opening a separate, dedicated thermal path.
- Result: a claimed 30% reduction in thermal resistance.
- Practically, it's built to slot into SK hynix's existing manufacturing: the solution uses SK hynix's Wafer Level Packaging (WLP) process based on the already market-proven Mass Reflow Molded Underfill (MR-MUF) technology, meaning it needs only minimal design adjustments rather than a wholesale process overhaul.
- It's targeted at future generations — HBM5 and beyond — rather than being retrofitted into current HBM4.

## Samsung's HPB (Heat Path Block)

- Conceptually parallel to SK hynix's approach and targeting the same root cause: Samsung's HPB is reported to introduce an independent thermal path within the D2D PHY region, improving heat conduction and dissipation, lowering thermal resistance, and strengthening overall system stability.
- Samsung's CTO stated that HPB has already been implemented in HBM4E, with reliability and stability fully validated — so, notably, Samsung's version is a generation ahead of SK hynix's in actual deployment status.
- Samsung also frames this as an integration-level decision, not a bolt-on: the key lies in optimizing the placement of the base die and core dies, suggesting HPB is being designed into the full memory-stack architecture from the start.

## Why this competitive detail matters (and could be a good interview point)

Both companies independently identified the exact same physical bottleneck (D2D PHY heat concentration) and are pursuing structurally similar fixes (a dedicated internal thermal path, rather than only relying on external heat spreaders/liquid cooling) — which is a strong signal that this is a genuinely fundamental constraint on how far HBM stacking can scale, not a vendor-specific quirk. As stack heights push further (some reporting mentions layer counts potentially reaching around 20 for HBM4E/HBM5), thermal management is becoming an increasingly critical constraint for both performance and scalability alongside bandwidth — arguably as important a design axis going forward as bandwidth itself.

Here are direct links if you want the primary reporting:Those five should cover it well — the Tom's Hardware and Blocks & Files pieces are the best plain-English explainers of iHBM specifically, TrendForce gives you the clean Samsung-vs-SK hynix comparison, ServeTheHome has the conference-level technical detail including Micron's angle, and the MDPI paper is the peer-reviewed source if you want actual thermal-resistance numbers and modeling.

![[iHBM.webp]]

Good clarifying question — and there's a distinction worth making here, because "the wires" and "the PHY" are two different things.

**The interposer wires themselves aren't what's generating the heat**

The silicon interposer is largely a passive component — it contains dense metal redistribution layers (RDL), essentially very fine, densely-packed wiring that physically connects the GPU's pins to the HBM stack's pins. Passive copper wiring carrying a signal does generate some resistive (I²R) heating, but it's relatively minor compared to what's happening at the two ends of those wires.

**The D2D PHY is active circuitry, not the wires themselves**

The D2D PHY sits on the **base die** of the HBM stack (the bottom-most die in the stack, which acts as the interface/controller layer between the DRAM dies above it and the outside world) and on the corresponding interface on the GPU/accelerator side. This PHY contains active transistor circuits — things like:

- Output drivers pushing signals onto the interposer wires
- Input receivers and amplifiers picking up incoming signals
- Clock/timing circuitry, similar in spirit to the DLLs we discussed earlier, keeping thousands of signal lanes properly synchronized
- Equalization and signal-conditioning circuits to maintain signal integrity at very high per-pin data rates

These are the **billions of transistors switching at extremely high frequencies** that generate the real heat — through dynamic switching losses, leakage current, and resistance in the active circuitry — not the passive metal traces of the interposer itself.

**So to directly answer your question**

iHBM isn't cooling the interposer's wires as they run "through and under" the package. It's cooling the **active driver/receiver circuitry sitting on the HBM base die** (right where those wires terminate and connect into the HBM stack) — that's the D2D PHY. The wires themselves are mostly just a passive conduction path; the heat problem is concentrated at the transistor-dense interface circuit that has to drive and receive signals across that interposer at extremely high speed, packed into a small area, sitting right on top of (and thermally coupled to) an already very hot GPU die.

**Why this location specifically is the worst spot, thermally**

The base die's PHY is squeezed between two heat sources simultaneously: it's generating its own switching heat, and it's sitting directly on the interposer right next to the GPU, which is radiating substantial heat sideways into the same small region. That combination — self-generated heat plus incoming heat from the neighboring die, concentrated at a physically small, transistor-dense circuit block — is exactly why SK hynix and Samsung both singled out the D2D PHY as the priority target for a dedicated cooling structure, rather than treating the whole package uniformly.

----
Yes — and this is actually one of the more concrete, well-defined engineering obstacles in your research area, not just a vague concern. Let's go through the specific mechanisms, since "synchronization issues" is doing a lot of work in that sentence and it's worth unpacking exactly where they come from.

## Why bank groups are electrically tightly coupled to begin with

Recall from earlier in our conversation: bank groups exist specifically because they **share** command/address decode logic and, critically, the **data I/O path** to some degree — this is exactly why tCCD_L (same bank group) is longer than tCCD_S (different bank group), since consecutive accesses to the same bank group contend for shared internal resources that different bank groups don't. That shared-resource relationship is precisely what makes independent frequency scaling of individual bank groups hard — you're trying to decouple things that were deliberately designed to be coupled for performance reasons.

## Concrete synchronization problems

**1. Clock domain crossing (CDC) on the shared bus**  
All bank groups within a channel ultimately feed data onto the _same_ shared DQ/DQS output path to the host. If bank group A is running at a different frequency than bank group B, but both need to place their read data onto that single shared bus, you need proper clock-domain-crossing logic (synchronizer FIFOs, handshaking) at the boundary — this is a classic, well-understood but real source of added latency, area, and design complexity in any multi-clock-domain chip. It's not free, and every crossing you add is a place bugs (metastability, data corruption from improperly synchronized signals) can hide if not handled with real rigor.

**2. Command/address timing reference gets ambiguous**  
Every DRAM timing parameter we've discussed (tRCD, tRP, tCCD, etc.) is defined in clock cycles, referenced to a specific clock domain. If different bank groups run at different frequencies, the memory controller has to track, per bank group, what its _current_ frequency is and translate cycle-count timing parameters into actual elapsed time correctly for each one independently — this adds real controller complexity and a real chance of subtle timing violations if the bookkeeping is imperfect.

**3. Bank-group interleaving breaks down**  
A big part of DRAM/HBM's effective throughput comes from **interleaving accesses across bank groups** specifically to hide each other's latency (issue to bank group A while bank group B is still busy from a prior command). If bank groups are running at different frequencies, this fine-grained interleaving scheme — which assumes a shared, common timing reference across bank groups — becomes much harder to schedule correctly, and you risk losing exactly the parallelism benefit that bank groups exist to provide in the first place.

**4. DLL/PLL relock overhead, multiplied**  
Recall our DLL discussion: changing frequency requires the DLL to re-lock before the chip can reliably use the new clock. If every bank group can independently change frequency, you either need **separate DLL/PLL circuits per bank group** (real silicon area and power cost, since these aren't tiny blocks) or a shared DLL that has to serialize/arbitrate relock requests across groups — either way, added complexity and potential relock-latency stalls exactly when you're trying to react quickly to a thermal event.

**5. Voltage domain crossing, if you're doing DVFS (not just DFS)**  
If bank groups also run at different _voltages_ (true DVFS, not just frequency scaling), any control/address signal crossing from one voltage domain to another needs **level shifters** — extra circuits, extra latency, extra power overhead, and extra physical design complexity (separate power delivery network rails have to physically route to each domain without excessive IR drop, which becomes its own layout challenge at fine granularity).

**6. Refresh command complexity**  
REFRESH commands (and their thermal-driven refresh-rate escalation, when temperature rises) are typically scheduled with some assumption of a shared timing base across banks. If bank groups run at different frequencies, keeping each one's actual refresh interval (in real time, not cycles) correctly met while their clocks drift independently relative to each other adds real scheduling complexity to the controller.

## Why this matches what the literature says

This is exactly the tradeoff called out explicitly in the Tasa paper I found earlier: _"Traditional DVFS lowers the frequency of all cores to reduce temperature, but this approach decreases overall performance more than it needs to, as some cores are far from reaching their thermal limits. However, adopting finer-grained DVFS introduces significant overhead and synchronization challenges."_ That's the core tension your research sits right in the middle of: **whole-stack DVFS is simple but wasteful** (you throttle cool regions unnecessarily along with hot ones), while **finer-grained DVFS (bank-group or below) is more efficient but pays real synchronization costs** — the ones listed above.

## Why this matters for your research direction

This is a genuine open design-space question, and a few implications worth considering for your work:

- **This is likely why most real/patented designs stop at channel or pseudo-channel granularity**, not bank-group — channels in HBM are more naturally electrically separable (more independent I/O, less shared internal resource contention) than bank groups within a channel, making channel-level DVFS a much more tractable sync boundary than bank-group-level.
- **The TSMC DDVFS patent family** (mentioned earlier) explicitly targets **per-cell-group** granularity as the state of the art, which suggests the industry has found _some_ way to manage this — worth digging into that patent family specifically to see what synchronization scheme they propose, since that's likely the most current real answer to exactly your question.
- **A legitimate research contribution** could be: quantifying exactly how much synchronization overhead (latency, area, power) bank-group-level DVFS costs versus the thermal/performance benefit it buys, to establish where the real Pareto-optimal granularity boundary sits — channel-level, bank-group-level, or somewhere in between (e.g., grouping multiple bank groups into a shared "thermal zone" that shares one V/F domain, trading some of the fine-grained thermal benefit for a large chunk of the synchronization-cost reduction).

**One-line summary for interview/research-framing purposes:** "Bank-group-level DVFS causes synchronization issues because bank groups deliberately share tightly-coupled resources — the I/O data path, command timing reference, and often DLL/PLL circuitry — precisely the resources that make independent frequency/voltage domains hard to decouple cleanly. This is why the literature identifies fine-grained DVFS overhead as a real cost, and why most practical designs settle for coarser granularity (channel-level) or specialized per-cell-group circuit techniques (like TSMC's DDVFS patents) rather than naive per-bank-group scaling."

---

combination of low power consumption and high bandwidth make this category of memory ideal for embedded systems.

Besides the increasing average-performance requirements, applica-
tions used in automotive and avionics carry real-time requirements,
where the total worst-case execution time (WCET) of all tasks should
never exceed their respective dedicated deadlines. Therefore, the
consolidation of HBM in critical systems requires careful analysis of
its timing predictability properties with emphasis on timing isolation.
This enables the safe execution of mixed-criticality software and
predictable and tight worst-case memory access latency so that it can
be shown that the benefits of HBM in average memory performance
remain for worst-case memory performance.

To study the main memory system from the real-time perspective,
two system components need to be analyzed: the memory device
itself and the on-chip memory controller managing accesses to the
device.

---

Partial answer, and the details matter for your research design: **low-power states are quite feasible at pseudo-channel granularity with modest overhead, but true DVFS (frequency change) is not, because of what pseudo-channels do and don't share.**

## What pseudo-channels actually share vs. keep separate

This is the crux, and it's a different sharing pattern than either bank groups or full channels:

- **Shared**: each HBM channel is split into two pseudo-channels that **share the same row and column command/address bus**, and critically, **share the same input clock (CK)** for that command interface.
- **Separate**: each pseudo-channel has its **own set of banks** (typically half the channel's banks), its **own sense amplifiers and data path**, and — per the HBM2/3 spec — **its own mode registers**, meaning some configuration state (including certain low-power/refresh settings) can genuinely differ per pseudo-channel.

So pseudo-channels sit in an intermediate position: more independent than bank groups (separate data path, separate banks, separate mode registers), but less independent than full channels (shared command bus, shared clock).

## Why this breaks clean DVFS at pseudo-channel granularity

Since both pseudo-channels within a channel **receive the same command clock**, you cannot run pseudo-channel A's circuitry at one frequency and pseudo-channel B's at another without splitting that shared clock domain — which reintroduces exactly the CDC (clock-domain-crossing) problem we discussed for bank groups, just localized to the command/address interface rather than the full data path. You'd need to insert a synchronizer or generate a separate divided/re-timed clock specifically for one pseudo-channel's internal circuits while the other still tracks the original shared CK — real added complexity, not something you get "for free" just because the data paths are separate.

**So: no, you can't cleanly get independent frequency per pseudo-channel without paying a real CDC cost — the shared command clock is the limiting factor**, not the data path (which actually is separate).

## Why low-power states are a different, much more tractable story

Low-power states are mostly about **gating activity/power in circuitry that's already structurally separate per pseudo-channel** — this doesn't require touching the shared clock domain at all:

- Since each pseudo-channel has its own banks and sense amps, you can put an **idle pseudo-channel's banks into precharge power-down** (recall: idle bit lines just sit at Vdd/2 — no active current draw beyond leakage) while the other pseudo-channel keeps banks open and active, all without needing any frequency change or clock-domain split.
- Since each pseudo-channel has **separate mode registers**, JEDEC's spec structurally supports configuring some per-pseudo-channel state independently already — this is exactly the kind of "knob" your DVFS/DTM research could exploit without inventing new hardware mechanisms, just by using per-pseudo-channel low-power entry/exit commands based on your workload/thermal signal.
- This matches the framing from the paper I found earlier — the Unified Core-Memory Power Regulation work explicitly separates DVFS (needs frequency change → real sync cost) from **DRAM low-power states (LPM)** as two distinct knobs, and treats LPM as the finer-grained, lower-overhead lever specifically because it doesn't require clock manipulation.

## Practical implication for your research design

This suggests a genuinely useful, low-risk angle for your work: **use pseudo-channel-granularity for low-power-state control** (power-gating idle banks/data paths per pseudo-channel based on per-region thermal/activity signals — cheap, no sync overhead) **while reserving true DVFS for coarser channel-level granularity** (where the shared clock problem is already solved by design, since channels do have their own clock generation). This gives you two-tier control: fine-grained power-down where it's nearly free, coarser frequency/voltage scaling where the sync cost is justified by the larger thermal/performance impact of a whole channel.

**One-line summary for your research framing:** "Pseudo-channels share a single command clock, so independent frequency scaling per pseudo-channel reintroduces the same CDC cost as bank-group DVFS. But since pseudo-channels have separate banks, data paths, and mode registers, per-pseudo-channel low-power-state control (power-gating idle banks) is achievable without touching the shared clock domain — making it a much cheaper, finer-grained complement to channel-level DVFS rather than a substitute for it."

---

**tWL (Write Latency)** is the delay between the WRITE command and when the **first data bit appears on the DQ bus**, not when it actually gets written into the memory cell/capacitor.

**Why this delay exists at all**: recall our earlier discussion of internal command/timing pipelines and DLL-based clock alignment — the DRAM's internal circuitry needs some fixed number of cycles after receiving and decoding the WRITE command before it's ready to actually accept incoming data at its input receivers/write drivers. tWL is exactly that "get ready" window, defined from the controller's perspective so it knows precisely when to start pushing data out.

### What happens _after_ the data lands on the bus

Once the write data actually arrives at the DRAM (after tWL has elapsed, and the burst of write data comes across the DQ pins over the burst length), a separate sequence happens internally:

1. The **write driver** (recall our earlier discussion) takes that incoming data and forces it onto the selected bit line(s) through the column mux
2. This drives the storage capacitor of the selected cell(s) to the new charge level
3. This actual internal charging process, plus the time needed before the row can be safely closed (recall the earlier tRAS discussion — the row needs to stay open long enough to guarantee the value is properly settled), is governed by a **different** parameter: **tWR (Write Recovery Time)** — the minimum time after the last write data is transferred before a PRECHARGE can safely be issued to that bank, ensuring the cell has actually finished being written before you disturb it.
---
Each channel of the HBM interface
is independently clocked and has its own command / data interface.
Channels do not need to be synchronous to each other 

unlike DRAM, HBM
is not organized into ranks. Instead, each HBM channel has a
number of banks (e.g. 8 or 16), and each bank is divided into
half banks [19] [20]. Each half, under the so called legacy-mode,
produces 64 bits, so that a full bank produces 128 bits in each access
(having the column width of 128 bits). Each set of half banks have
64 dedicated I/Os.
For DDR DRAMs, when a specific memory location is fetched via row,
column and bank address, each chip across the rank supplies either
4, 8, or 16-bit (column width) data from the same location (based on
the device type, which is so-called x4, x8, or x16). Hence, assuming
8 chips, this results in a 64 bit width per rank.

Observation 1. HBM offers wider connections to the processing unit
(1024 bits) with respect to DRAM (e.g. 64 bits).

Observation 2. A processor can be connected to several independent
HBM stacks residing on the same silicon interposer.

Observation 3. HBM channels, even those in the same core die,
operate independently via private data and address/control signals
and buses. Each HBM channel is connected to the logic die via a
non-shared 128-bit TSV.

Observation 4. While HBM is based on DRAM banks, unlike DRAM
it is organized into channels, pseudo channels, (logical) banks and
half (physical) banks.

Observation 5. In DDR3/4 the access granularity is either 4, 8, or
16 bits per physical bank and 64 bits for the DRAM rank, while in
HBM each bank supplies 128 bits

---

Good question to pin down, because the phrasing in that paper is easy to misread. **The 128 bits is the width of the whole channel's data bus, and because only one bank within a channel is ever active/outputting at a time, a single bank's access ends up supplying the full 128 bits per transfer** — so "each bank supplies 128 bits" and "the channel is 128 bits wide" are actually describing the same physical fact, not two separate numbers.

## Why the observation is phrased "per bank" then

This is making a _direct structural comparison_ to the DDR3/4 side of the sentence, and the comparison only makes sense once you see what's different between the two architectures:

**In DDR3/4 (chip-based, rank aggregation):**

- A single DRAM _chip_ (e.g., x4, x8, or x16) only has that many DQ pins of its own — so one chip's bank, by itself, can only ever output 4, 8, or 16 bits per access, no matter how wide the array internally is.
- To get to a useful 64-bit-wide transfer, you need to physically combine **multiple separate chips** side-by-side into a **rank** (recall our earlier 8×x8-chip example) — the 64-bit figure only exists at the _rank_ level, as an aggregate across multiple chips, each independently contributing its narrow 4/8/16-bit slice.
- So in DDR, "bank granularity" (4/8/16 bits) and "rank granularity" (64 bits) are genuinely two different numbers, because reaching 64 bits requires assembling multiple physically separate chips.

**In HBM (monolithic stack, per-channel width):**

- There's no "rank" concept at all in HBM (confirmed directly by that same McMaster paper — DRAM ranks do not exist in HBM). A channel is a complete, self-contained 128-bit-wide interface belonging to one HBM stack — not an aggregation of several separate chips' narrow outputs.
- Within that channel, multiple banks exist, but **only one bank drives the channel's shared 128-bit data path at a time** (exactly the same "one bank/one driver active on a shared bus" principle we discussed for the DDR chip's internal bus earlier in our conversation, just scoped to HBM's channel instead of a DDR chip's external pins).
- Since the _entire_ channel width (128 bits) is already reached by whichever single bank is currently being accessed — there's no need to combine multiple banks' outputs together the way DDR needs to combine multiple chips — the bank-level granularity **is** the channel-level granularity. They're the same number because HBM's per-channel width is achieved by one active bank alone, not by aggregating several.

## Direct answer to your question

**Both descriptions are correct, and they refer to the same physical quantity**: one bank, when accessed, supplies data across the _entire_ 128-bit channel-wide bus, because that bus (128 bits) belongs to the channel as a whole, but only one bank within that channel is ever actively driving it during any given access. The paper says "per bank" specifically to make the comparison to DDR's per-_chip_-bank granularity apples-to-apples — highlighting that HBM reaches full bus-width granularity from a single bank access, whereas DDR needs multiple chips' banks (a whole rank) working together to reach its full bus-width (64-bit) granularity.

**One-line summary for interview/research purposes:** "In HBM, a channel's 128-bit data bus is entirely serviced by whichever single bank is currently active — there's no rank-style aggregation across separate chips needed. In DDR3/4, a single chip's bank can only supply its own narrow pin-width (4/8/16 bits), so multiple chips must be combined into a rank to reach the full 64-bit bus width. The observation is highlighting that HBM achieves full bus-granularity at the bank level alone, which is structurally impossible in chip-based DDR ranks."

---

This turned up excellent, precise data — let me go through your questions in order.

## How do you get 64 bits from a pseudo-channel access?

Same principle as before, just scoped down one level: a pseudo-channel has its own dedicated **64-bit data interface** (in HBM2/2E/3), and — same as the channel case — only **one bank within that pseudo-channel** drives its shared data path at any given moment. So the full 64 bits comes from one active bank's column mux output, exactly analogous to how one bank supplied the full 128 bits at the whole-channel level. Note HBM3 actually narrows this further per pseudo-channel: HBM3's pseudo channel mode divides a channel into two individual sub-channels of 32-bit I/O each as compared to 64-bit I/O in HBM2 — so the exact per-pseudo-channel width has shrunk generation over generation even as total channel count doubled, but the underlying "one active bank drives the whole pseudo-channel bus" principle stays the same.

## Banks per channel, by generation

- **HBM2**: 8 channels total, and in pseudo-channel mode, a pseudo channel comprises two bank groups, with each group containing four banks — so 8 banks per pseudo-channel × 2 pseudo-channels = 16 banks per channel.
- **HBM3**: 16 channels per stack (32 "virtual" channels counting pseudo-channels), and supports 16, 32, or 48 banks per channel based on device density and channel — a notably wider range than HBM2, scaling with capacity.
- **HBM4**: doubles channel count again, from 16 channels (HBM3) to 32 channels with 2 pseudo-channels per channel — banks-per-channel figures for HBM4 specifically weren't in a clean citation, but following the established pattern, expect it to scale with density similarly to HBM3's approach.

## Does HBM use bank groups?

**Yes — and this directly overturns the assumption in your question.** This is confirmed explicitly for HBM2: a pseudo channel comprises two bank groups, with each group containing four banks, and separately for HBM3, an HBM3 memory model spec sheet explicitly lists "Supports Bank grouping" as a feature. So bank groups exist in HBM, same as DDR4/5, and for the exact same underlying reason (shared internal resources within a group, hence the tCCD_L vs tCCD_S distinction we discussed earlier) — the "only one bank is active" fact doesn't eliminate bank groups, because bank groups aren't about how many banks are active at once, they're about which banks share tightly-coupled internal circuitry (and thus have specific timing constraints between consecutive accesses to them).

## Now, the DVFS question — this is the important one to correct

Your reasoning is intuitive but has a flaw worth being precise about, because it's a common trap in this kind of analysis.

**The flaw**: "only one bank is active at a time" describes which bank is currently driving the _data path_ — but it does **not** mean only one bank's clock matters at that instant. Here's why:

1. **The clock is a shared physical wire reaching all banks simultaneously, all the time** — it's not switched or routed to "whichever bank happens to be active right now." Every bank in the (pseudo-)channel receives the same clock edge at the same moment, all the time, regardless of which bank is currently transacting.
    
2. **"Active" bank changes extremely rapidly and unpredictably** — a memory controller interleaves requests across many banks specifically to hide latency (recall bank-level parallelism from earlier). Multiple banks can also have rows _open_ simultaneously even if only one is driving the shared data bus at a given instant — so at any moment, several banks may be in the middle of ACT/PRE sequences even though only one is on the data bus right now. Which bank is "the active one" can change every few nanoseconds.
    
3. **If you tried to scale frequency to just "the currently active bank,"** you would need to change the clock frequency dynamically, essentially per-access — but changing frequency requires a DLL/PLL relock (recall our earlier discussion), which takes far more time than the interval between successive bank accesses. You'd pay a relock penalty constantly, far exceeding any benefit, and you'd also disrupt every _other_ bank that happens to have a row open and is relying on that same clock staying stable for its own internal timing (tRAS, refresh windows, etc.).
    

**So the correct conclusion is actually the reverse of your intuition**: since all banks in a (pseudo-)channel are wired to the _same_ physical clock net, you were **already** constrained to channel-level (or pseudo-channel-level, for the shared CA/clock scope) granularity for true frequency scaling — the "one bank active" fact doesn't grant you anything finer-grained, because the clock domain boundary is set by the shared wiring, not by how many banks happen to be transacting at once. What it _does_ support well (as we discussed last turn) is **low-power state gating at the bank level** — since each bank has independent power/precharge state, you genuinely can power down individual idle banks without touching the shared clock at all. That's a real, exploitable independence. Frequency, though, is stuck at whatever granularity the shared clock wiring defines — which, per everything we've found today, is the pseudo-channel/channel level, not the individual bank.

**One-line summary for your research framing:** "Only one bank driving the data bus at a time is a data-path fact, not a clock-domain fact — the clock reaches every bank in a (pseudo-)channel simultaneously and continuously, regardless of which bank is transacting. So bank-level DVFS isn't cheaper just because access is serialized; the clock-domain boundary (and therefore the real DVFS granularity limit) is set by shared C/A/clock wiring, confirmed to be pseudo-channel/channel scope in HBM2/3. Bank-level independence is real and exploitable for low-power-state gating (since power/precharge state is per-bank), but not for frequency scaling."

___
C. HBM’s Core Memory Cells

Although HBM has a completely different structure w.r.t. conven-
tional DRAMs, it still uses conventional DRAM cells as its memory
storage core, i.e., HBM banks are organized (and accessed) the
same way as in DRAMs. Accordingly, all commands have the same
associated timing constraints as in conventional DRAM, which are
dictated by the JEDEC standard both for DRAM [17] and HBM [12].

![[hbm2_arch.png]]

---

Close, but there's one important correction: **a single command does not go to both pseudo-channels simultaneously to jointly produce 128 bits.** In pseudo-channel mode, a command targets **one specific pseudo-channel only** — the two pseudo-channels operate as semi-independent virtual channels, not as two halves of one combined access.

## Where the correction comes in

From the JEDEC-based sources we found: the two pseudo channels function semi-independently: they share the same channel's row and column command bus as well as clock inputs, but **decode and execute commands individually**. And more specifically, from the Intel HBM2E interface documentation: address bit BA4 directs commands to either pseudo-channel 0 (BA4 = 0) or pseudo-channel 1 (BA4 = 1) — the HBM controller handles the addressing requirements of the pseudo-channels.

So the actual flow is:

1. Memory controller sends an address/command onto the **shared** command/address bus and clock (both pseudo-channels physically receive this signal, since it's a shared wire — but that's just electrical broadcast, not functional targeting).
2. A specific address bit (**BA4**) tells each pseudo-channel's decode logic whether _this particular command_ is meant for it or not.
3. Only the pseudo-channel whose BA4 bit matches actually acts on the command — decodes it, selects its bank group, selects its bank, and drives its own 64-bit data path.
4. The **other** pseudo-channel ignores this command (from its own decode logic's perspective) — it's free to be independently working on a **completely different command with a different address**, targeting a different bank in its own bank groups, at essentially the same time.

## Why this matters — it's the entire point of pseudo-channel mode

This is actually the key benefit pseudo-channel mode is designed to deliver: instead of a whole 128-bit channel being tied up serving one request at a time (legacy/non-pseudo-channel mode), splitting it into two 64-bit pseudo-channels lets the controller issue **two independent, unrelated requests concurrently** — better exploiting bank-level parallelism and improving effective command throughput, at the cost of narrower per-access data width (64 bits instead of 128).

## So when _does_ a single access produce 128 bits?

That's **legacy (non-pseudo-channel) mode** — where the channel isn't split at all, a single command engages the whole 128-bit-wide channel, one bank is selected from across the _entire_ channel's banks (not scoped to a "half"), and that one bank's access supplies the full 128 bits in one shot. This is the scenario the "each bank supplies 128 bits" paper observation was describing.

## Corrected version of your chain, for pseudo-channel mode specifically

Address/command broadcast on shared C/A bus and clock → **only the pseudo-channel matching the address's BA4 bit decodes and acts** → within that one pseudo-channel, one bank group is selected → one bank within that group is selected → that bank outputs **64 bits** total for this request → those 64 bits go out via that pseudo-channel's own TSVs to the base die, then to the processor. **Meanwhile**, the other pseudo-channel may simultaneously be handling an entirely separate command/address, independently producing its own 64 bits for a different request — not combining with the first pseudo-channel's 64 bits into one 128-bit word for the same access.

**One-line summary for your research framing:** "Pseudo-channel mode doesn't split one 128-bit access into two parallel 64-bit halves of the same request — it uses an address bit (BA4) to route each command to exactly one pseudo-channel, letting the two pseudo-channels service two independent, unrelated 64-bit accesses concurrently. Getting a genuine single 128-bit access from one bank only happens in legacy (non-pseudo-channel) mode."

---

Yes, exactly — that's precisely correct, and it's a good way to frame it. Since the C/A bus and clock are shared, only **one command can be issued at a time** across the two pseudo-channels — so to keep both busy, the controller has to **time-multiplex its commands** on that one shared bus, alternating (or otherwise interleaving) which pseudo-channel each successive command targets.

## Why this still gives you real parallelism, despite the serialized commands

The key insight is that **issuing a command takes far less time than actually executing it** (recall ACT→RD/WR→data-out involves multiple cycles of internal latency — tRCD, CAS latency, burst transfer time, etc.). So the sequence looks like:

1. Cycle N: controller issues a command (say, ACT or RD) targeting **PC0**, via BA4=0
2. Cycle N+1 (or whenever the bus is next free): controller issues a **different** command targeting **PC1**, via BA4=1
3. While PC1's command is now being internally processed (row activation, sensing, column access, etc. — all internal to PC1's own banks/data path), **PC0 is simultaneously still working through its own internal latency** from the earlier command
4. Once each pseudo-channel's internal processing completes, each drives its own 64-bit data path independently, at whatever point in time its own latency finishes — not necessarily at the same moment as the other

So the C/A bus is genuinely a shared, serialized resource — but the _execution_ time within each pseudo-channel (the expensive part, latency-wise) overlaps, because each pseudo-channel has its own banks and internal state machine working independently once it receives its command. You're pipelining short command-issue slots against long internal-execution windows, which is exactly the same principle as bank-level parallelism we discussed way earlier in this conversation (issue ACT to bank 0, then issue ACT to bank 1 while bank 0's row is still opening) — just applied one level up, at the pseudo-channel granularity, with the constraint that the two pseudo-channels' commands can never occupy the same C/A cycle.

## Why this is a good design, not just a workaround

The C/A bus, while shared, is relatively lightly loaded compared to the data path — commands are short (a handful of bits) and issued relatively infrequently compared to how long each command's internal execution takes. So even though it's serialized, the C/A bus rarely becomes the actual bottleneck; the real benefit of splitting into pseudo-channels is exactly what you're describing — using the brief moments when one pseudo-channel is "busy internally but not currently needing the C/A bus" to slip in a command for the other, keeping both banks-worth of internal parallelism actually utilized instead of leaving one pseudo-channel idle while all commands funnel to just one.

One more detail worth knowing, from the HBM3 spec side: HBM3 supports **semi-independent row and column command interfaces** — meaning row commands (ACT, PRE) and column commands (RD, WR) can, to some extent, use separate command paths, which gives the controller a bit more flexibility in scheduling these interleaved commands across the two pseudo-channels without every single command type contending for exactly the same slot.

**One-line summary for your research framing:** "Because the C/A bus is shared, commands to the two pseudo-channels must be serialized in issue order, but their internal execution (activation, sensing, data readout) proceeds independently once issued — so the controller interleaves short command slots against each pseudo-channel's much longer internal latency window, extracting real overlap/parallelism from a resource that's serialized only at the point of issue, not during execution."

---

The processor reads **64 bits per pseudo-channel transaction**, not a combined 128-bit word for a single request — the 128 bits is just the total physical pin count available in that channel, split into two independently-operating 64-bit buses.

## Why it's 64 bits, not 128, per access

Since each pseudo-channel decodes and executes its own commands independently (as we just established — BA4 routes each command to exactly one pseudo-channel), each pseudo-channel's data burst is a **complete, self-contained 64-bit transfer for its own specific request**. There's no combining step where the processor waits for both pseudo-channels to finish and then stitches their outputs into one 128-bit word — that would only make sense if both pseudo-channels were servicing the _same_ request, which, as we covered, they're not.

## But both 64-bit buses can be active at the same physical moment

This is the subtlety worth being precise about: **at the pin/PHY level**, the processor's interface to that HBM channel does have 128 total data pins (64 dedicated to PC0, 64 dedicated to PC1), and both halves _can_ be toggling simultaneously — but each half is carrying data for a **completely different, unrelated request**. So from the processor's perspective, it's not "reading one 128-bit chunk" — it's "concurrently receiving two independent 64-bit chunks," which happen to arrive on physically adjacent pins at potentially overlapping times, purely because the two pseudo-channels' internal execution latencies happened to finish around the same moment (or not — they could just as easily finish at different times, since each pseudo-channel's timing depends on its own independent command history).

## How this looks to the memory controller / software

This maps directly onto what we found earlier about pseudo-channel addressing: each pseudo-channel effectively behaves like its own small, independent memory channel with a 64-bit interface, own banks, and own mode registers. From a system/controller-design perspective, you generally treat each pseudo-channel as its own logical unit of 64-bit-granularity transfers, tracked and scheduled independently — the "128-bit channel" framing is really just describing the physical/electrical grouping and total pin budget, not the logical unit of data the processor consumes per transaction.

**One-line summary for your research framing:** "The processor reads 64 bits per transaction, scoped to whichever pseudo-channel serviced that particular request — the 128-bit figure is just the combined physical pin count of both pseudo-channels' independent 64-bit buses, which can be simultaneously active carrying two unrelated 64-bit transfers, not jointly forming one 128-bit word for a single access."

---

Yes — that's exactly the right instinct, and it's a completely standard, necessary part of memory controller design, not something specific to HBM's pseudo-channel setup (though pseudo-channels do make it more likely to happen).

## Why buffering is necessary in the first place

The memory controller's job is to bridge two different worlds: the DRAM/HBM side, where data arrives at whatever moment each pseudo-channel's internal latency happens to finish (unpredictable, dependent on ACT/RD timing, row-buffer state, etc.), and the processor side, which typically wants to consume completed transactions through its own internal bus/interconnect at its own pace, one at a time (or via its own parallel ports, but still needs _some_ orderly hand-off mechanism). These two sides rarely align perfectly in time, so a buffer is the standard way to decouple them.

## The actual mechanism: per-channel read queues/buffers

Real memory controllers maintain dedicated **read return buffers/queues** — essentially small FIFOs (First-In-First-Out buffers) — one per channel (and typically tracked per pseudo-channel too, given they're semi-independent). When data comes back from a pseudo-channel:

1. It's latched into that pseudo-channel's read-return buffer entry, tagged with metadata identifying which outstanding request it corresponds to (memory controllers track a queue of in-flight requests specifically so they can match returning data back to the original request that asked for it, since requests can complete out of order).
2. If **two pseudo-channels' data return at the same clock cycle**, both simply get written into their own respective buffer entries — since they're logically and physically separate data paths (different sets of TSVs, different registers inside the controller), there's no actual electrical or storage conflict; the controller has separate holding registers for each pseudo-channel already.
3. From there, the controller's scheduling/arbitration logic decides in what order to drain those buffered results out toward the processor's interconnect — this might be a simple round-robin, a priority scheme (recall the "command priority"/QoS mechanism mentioned in that Intel HBM2 controller documentation earlier — normal vs. high priority, served round-robin within the same priority level), or based on whichever original request is "older"/more time-critical.

## Why this matters architecturally — it's the same idea as the DRAM row buffer, one level up

This is conceptually similar to how a DRAM row buffer decouples the "slow" internal array access from the "fast" column-access-driven output we discussed early in our conversation — the read-return buffer in the memory controller decouples the "somewhat unpredictable, independently-timed" completion of multiple in-flight HBM requests from the "needs one orderly stream" consumption pattern the processor's own bus/cache-fill logic expects.

## One added nuance: out-of-order completion, not just simultaneous arrival

It's worth generalizing your question slightly, since simultaneous arrival is really just one special case of a more general problem: because different pseudo-channels (and even different banks within them) can have wildly different latencies depending on whether they hit an open row (page hit, fast) or need a full activate/precharge cycle (page miss, slow), requests routinely **complete out of order** relative to the order they were issued — not just occasionally colliding at the exact same cycle. So the buffering/tagging/reordering mechanism isn't just there to handle the rare simultaneous-arrival case — it's the fundamental mechanism that makes out-of-order memory completion work at all, and simultaneous arrival is just the most obvious instance of the more general problem it solves.

**One-line summary for your research framing:** "Yes — the memory controller maintains per-channel/per-pseudo-channel read-return buffers (FIFOs) tagged with request IDs, specifically because pseudo-channels complete independently and often out-of-order relative to issue order, not just when they happen to land on the exact same cycle. Simultaneous arrival is handled trivially since each pseudo-channel already has its own separate buffer entry — the harder general problem this buffering solves is reordering/matching arbitrarily-timed completions back to their original requests before handing results to the processor."

---

Let's build this from the mechanics we've already covered — burst length, DDR transfer timing, and why the data bus itself is the actual constraint here.

## Why tCCD exists at all

Recall from our earlier discussion: after a column (CAS) command, the DRAM has to serialize the burst-length-worth of data out over the shared data bus, one clock edge at a time (DDR = 2 transfers per clock). If you issued a _second_ column command too soon — before the first one's burst has fully finished draining onto the bus — the two bursts would collide, corrupting both. **tCCD is exactly the minimum spacing needed to prevent that collision**, and its floor is set by how long it physically takes to drain one burst off the bus.

## Deriving the formula: tCCD_min = BL/2

Since DDR transfers **2 bits per clock cycle per pin** (recall our earlier DDR discussion — one on the rising edge, one on the falling edge), a burst of length BL takes:

**BL transfers ÷ 2 transfers-per-cycle = BL/2 cycles** to fully drain off the bus.

That's the physical minimum time the bus is "occupied" by one column command's data — so the next column command can't safely fire until that many cycles have passed.

## Applying it to DDR DRAM (BL=8)

**tCCD = 8/2 = 4 cycles.** This matches the classic DDR value you've probably seen quoted (tCCD_S = 4 in DDR4/5 for different bank groups) — it's not an arbitrary spec number, it falls directly out of the burst-length/bus-occupancy math.

## Applying it to HBM (BL=2 or BL=4)

HBM deliberately uses a **much shorter burst length** than mainstream DDR — up to BL=4 instead of DDR's BL=8. Plugging into the same formula:

- **BL=4**: tCCD = 4/2 = **2 cycles**
- **BL=2**: tCCD = 2/2 = **1 cycle**

Since the burst itself takes less time to drain (fewer bits per burst), the very next column command is allowed to fire much sooner — hence HBM's characteristically small tCCD values (1 or 2, versus DDR's 4).

## Why HBM chose a shorter burst length in the first place — and why this matters

This connects directly to two things we discussed earlier in this conversation:

1. **HBM already gets its bandwidth from width, not burst depth.** Recall the "64 bytes per command, matches a cache line" reasoning we worked through for DDR — that calculation assumed a fairly narrow rank (64 bits) needing a longer burst to accumulate a useful chunk of data per command. HBM instead already has an enormously wide per-channel bus (128 bits per channel, or 2,048 bits aggregate across a full stack) — so it doesn't need a long burst to accumulate a meaningful amount of data per access; a short burst over a very wide bus already moves plenty of data per command.
    
2. **Shorter tCCD directly means higher effective command throughput.** Since column commands can be issued more frequently (every 1–2 cycles instead of every 4), the controller can service far more distinct column accesses per unit time — which is exactly what you want for **fine-grained, high-parallelism access patterns** (like servicing many small, independent AI workload memory requests across many banks/pseudo-channels), rather than DDR's traditional model of occasionally pulling a large, cache-line-sized chunk per command.
    

## Tying back to your DVFS research

This is actually a nice structural example reinforcing something from your earlier questions: HBM's whole architecture — many narrow, independent channels/pseudo-channels, short bursts, reduced tCCD — is optimized around **many small, concurrent, independently-timed transactions** rather than DDR's model of infrequent, wide, long-burst transfers. That access pattern (lots of small, overlapping, independently-completing operations) is exactly why buffering/tagging at the controller (your last question) and fine command-level interleaving across pseudo-channels (a few questions back) matter so much more for HBM than for a traditional DDR rank.

**One-line summary for interview/research purposes:** "tCCD's floor is set by how long a column command's burst occupies the shared data bus — BL/2 cycles under DDR signaling. DDR DRAM uses BL=8, giving tCCD=4; HBM deliberately uses a much shorter burst (BL=2 or 4), giving tCCD=1 or 2, because HBM already achieves high bandwidth through sheer interface width rather than long bursts, and a short tCCD lets the controller issue column commands far more frequently — matching HBM's design intent of servicing many small, independent, highly parallel accesses rather than infrequent large ones."

---

Observation 6. Compared to DRAM, HBM has a reduced tCCD.

## Why do we need bursts at all?

This goes back to the prefetch architecture we discussed earlier. Modern DRAM cells and sense amps operate on a very wide **internal** data path (the whole prefetch-width chunk sensed from an open row), but the **external** pins are much narrower. A burst is simply the mechanism that reconciles this mismatch: instead of requiring a brand-new column command for every tiny piece of data, one column command triggers the DRAM to internally fetch a wider chunk (the prefetch width) and then **serialize it out** over several back-to-back clock edges.

The alternative — issuing a separate column command for every single narrow transfer — would be far worse, because:

- **Command overhead dominates**: recall tCCD is a real minimum spacing between commands; if you needed a full command cycle for every few bits, you'd waste huge amounts of time on command overhead rather than actual data movement.
- **You'd be fighting the physical array structure**: recall the array senses an entire **row** (page) into the row buffer as one costly operation (ACT). Once that expensive step is done, it makes sense to extract as much useful data as reasonably possible from that open row per command, rather than paying command overhead again and again for tiny slices of the same already-open row.

So bursts exist to amortize command overhead and match the wide internal array access to a narrower external pin interface — the same "wide internally, narrow externally, so serialize" principle we discussed for prefetch depth × rank width earlier.

## What is BL in HBM3/HBM4?

HBM has consistently used short burst lengths compared to mainstream DDR — the paper you're reading states HBM supports **BL=2 or BL=4** (versus DDR's BL=8), and this pattern has held from HBM2 through HBM3/HBM3E; I don't have a citation confirming HBM4 changed this fundamental burst-length choice, but everything about HBM4's design philosophy (even wider interface — 2,048 bits, more channels — 32, still emphasizing many small parallel accesses) points toward it continuing the same short-burst approach rather than reverting to DDR's longer BL=8, since a longer burst would work against exactly the fine-grained parallelism HBM's architecture is built around.

## Does one address fetch multiple columns due to bursts?

Yes — this is exactly what a burst does, and it's worth being precise about the addressing mechanism, since it ties back to the column-decoder discussion from earlier in our conversation.

When you issue a single column command (with a single starting column address), the DRAM doesn't just read that one column and stop — it reads that column **and then automatically increments through BL consecutive columns** (or a fixed pattern of columns, depending on the burst-addressing mode), all from the row that's currently open, without needing a new column address for each one. Internally, a **burst counter** takes the single supplied starting address and generates the sequence of column addresses needed to step through the burst, feeding each one into the column mux (recall our earlier mux/demux discussion) in turn, over successive clock edges.

So concretely: **one column command + one starting address → BL sequential columns automatically read out (or written), serialized onto the data bus over BL/2 cycles (DDR).** This is precisely why tCCD's formula (BL/2 cycles) makes sense — it's literally counting how long it takes to drain exactly this multi-column burst off the shared bus before the next command's burst can safely begin.

**Tying it back to why HBM's short burst matters for your research area**: a shorter burst (BL=2 or 4) means each command commits to reading fewer consecutive columns before "releasing" the bus — which is exactly the tradeoff that gives HBM its short tCCD and high command-issue frequency, well-suited to the many small, independent, thermally-scattered accesses your DVFS work is modeling, rather than DDR's model of occasionally pulling one large, long-burst chunk per command.

---

Observation 7. In HBM2, a single memory access (i.e., CAS
command) provides a 32B of data using BL = 4

Observation 8. Pseudo channels are semi-independent to each other:
while they share the row and column command buses and clock
inputs, they can decode and execute commands independently.

F. Dual Command Interface
Driven by cost constraints, DRAM adopts a shared column and
row address pins. On the other hand, with its wide I/O interface,
HBM deploys separate column and row address pins. Leveraging
this architecture, HBM employs dual address/command interface that
allows column-related commands (i.e. read and write) to be issued
simultaneously with the row-related commands (ACT and PRE) [22].

Observation 9. HBM has dedicated pins for column address that are
separate from row address pins. Hence, read/write commands and
addresses can be issued concurrently with row ACT/PRE commands.

Despite employing separate row/column address and command
bus, HBM needs to enforce usual DDR timings for ACT, RD/WR,
PRE since these timing constraints are imposed by the internal physi-
cal structure of the memory cells, which is still DRAM-based. Hence,
this does not change the timing of individual transactions (ACT,
RD/WR, PRE). However, this feature can reduce address/command
bus conflicts among different transactions

G. Implicit Precharge
In DRAM standard, a row in a bank can only be activated
after the previously open (active) row has been closed (precharged).
Under DRAM close-page protocol, this translates into the sequence
ACT-RD/WR-PRE. Under open-page if the row to access is open
RD/WR commands are issued, otherwise it is required to issue
PRE and ACT. Contrarily, when operating in pseudo-channel mode,
HBM controller can ignore the PRE command and issue the ACT
directly by leveraging the implicit precharge feature (Observation 10).
However, it is important to know that all associated ACT-to-PRE and
PRE-to-ACT constraints yet have to be satisfied.
Observation 10. In pseudo channel operation HBM allows a subse-
quent ACT command to be issued to another row in the same bank
without closing the previous row. In this case, the DRAM core itself
will issue an implicit PRE command to close the first row before
activating the second one.


---

Three distinct questions, let's take them one at a time — they connect nicely.

## Is it worth putting one pseudo-channel in LPM?

It's a genuine tradeoff, not a free win, and the answer is workload-dependent — which is exactly the kind of decision your DVFS/DTM research should be modeling explicitly rather than assuming.

**The cost side**: entering and exiting a low-power state isn't instantaneous. Depending on how deep the power-down is (precharge power-down vs. self-refresh, in DDR terms — HBM has analogous states), exiting typically requires re-establishing a valid clock/DLL lock and waiting a minimum exit latency before the pseudo-channel can safely accept commands again (recall the DLL relock discussion — you can't just instantly resume high-speed operation from a fully powered-down clock tree). If a request arrives for that pseudo-channel while it's parked, that request now pays this wake-up latency on top of normal access latency.

**The benefit side**: recall from earlier that idle bit lines/banks mostly cost leakage power, not much dynamic power — so the _power_ savings from LPM come primarily from reducing leakage and stopping unnecessary refresh/clock-toggling activity, which does matter over sustained idle periods but isn't dramatic over very short ones.

**So the rule of thumb**: LPM is worth it when the expected idle duration is long relative to the entry+exit latency overhead — a classic break-even calculation your research will likely need to formalize (amortize the fixed wake-up cost against the leakage saved over the idle window). For short, bursty idle gaps, thrashing a pseudo-channel in and out of LPM can cost more in latency (and even in dynamic energy, from the repeated relock transients) than it saves.

**Does it break data storage/access?** No — parking a pseudo-channel in a defined low-power state doesn't corrupt or lose stored data; refresh obligations are still met (either continued at a reduced rate or handled internally, depending on which power state), so the cells remain valid. The only functional consequence is **added latency for the next access to that pseudo-channel**, not data loss or corruption — as long as you exit the low-power state through the proper defined sequence before issuing commands again.

## Is data interleaved PC0/PC1/PC0/PC1... to maximize bandwidth?

Generally, yes — this is a standard **address interleaving** technique, and it directly explains why forcing one pseudo-channel into LPM has a bigger performance cost than you might first assume.

The memory controller's address-mapping function typically uses specific address bits (recall BA4 selecting which pseudo-channel) to decide, for consecutive or nearby memory addresses, which pseudo-channel (or channel, or bank) each one lands on. The whole point of this mapping scheme is exactly what you're describing: spread a stream of sequential/nearby accesses across independent resources so multiple requests can be served **concurrently** rather than piling up on one pseudo-channel while the other sits idle. This is the same underlying principle as bank-level interleaving, channel-level interleaving, and pseudo-channel interleaving — all instances of the same idea, just at different granularities.

**Why this matters directly for your LPM question**: if your address-interleaving scheme is actively spreading traffic evenly across both pseudo-channels (which is the default, bandwidth-maximizing behavior), then putting one pseudo-channel into LPM doesn't just "remove an idle resource" — it **actively degrades achievable bandwidth**, because roughly half your traffic (by design) was supposed to be landing there. Every one of those accesses now stalls on wake-up latency, and you've effectively lost the parallelism benefit the interleaving scheme was built to exploit.

**Implication for your research**: putting a pseudo-channel in LPM only makes sense when the controller can verify (or predict) that the workload's actual access pattern is genuinely skewed — i.e., real traffic isn't evenly hitting both pseudo-channels despite what a naive interleaving policy assumes, so parking the underused one doesn't meaningfully hurt achieved bandwidth. A workload-adaptive interleaving/LPM co-policy (dynamically re-mapping addresses to concentrate traffic on the _active_ pseudo-channel when the other is parked, rather than assuming a fixed 50/50 split) sounds like a legitimate, distinct contribution for your work — this connects your DVFS/thermal work to memory controller address-mapping policy, which isn't something I saw as an explicit focus in the papers we found earlier.

## Does lowering frequency (DVFS) risk data problems?

Only if you don't follow the proper switching sequence — done correctly (as JEDEC-style FSP mechanisms are designed to guarantee), it's safe. But the failure modes if done carelessly are real and worth understanding precisely, since they tie together several things we've covered:

**Why frequency changes are dangerous mid-operation**:

- **Burst timing depends on a stable clock period.** Recall tCCD/burst-length math assumes a constant clock period throughout the transfer — if frequency shifts mid-burst, the receiver's sampling edges (tied to the old period) would misalign with data still being clocked out at a new rate, causing bit errors or misread data.
- **DLL/PLL must relock.** Any frequency change invalidates the current DLL lock (recall our DLL discussion) — until it relocks, DQS/DQ launch timing is unreliable, so any command executed during that relock window risks corrupted reads/writes.
- **All cycle-count-based timing parameters change in real-time meaning.** tRCD, tRP, tCCD etc. are specified in _cycles_; at a new frequency, the same cycle count corresponds to a different absolute time — the controller must know to apply revised, correctly-scaled register/mode settings (recall the MRS/mode-register discussion) matching the new frequency, or it could violate real physical timing minimums even while nominally "following" the old cycle counts.

**Why the standard procedure avoids all this**: this is exactly why real DFS mechanisms (recall the LPDDR FSP discussion) mandate **quiescing the channel first** — ensuring no in-flight commands/bursts are active, all banks are idle/precharged — **then** switching frequency and relocking the DLL/PLL, **then** resuming traffic only after lock and settling are confirmed. Done this way, there's a clean handoff with no data at risk, because nothing is actively mid-transfer when the clock domain changes.

**So for your research**: as long as your proposed DVFS scheme changes frequency only at a properly quiesced boundary (all outstanding commands to that channel completed, no open bursts in flight) and goes through a defined relock/re-training step before resuming — matching how real FSP switching works — you won't introduce data corruption. The risk only appears if a scheme tries to change frequency instantaneously/opportunistically without that quiescing step, which is a genuine engineering hazard worth calling out explicitly if you're proposing something more aggressive than standard FSP-style switching (e.g., switching mid-idle-window based on a predictive thermal signal) — you'd need to guarantee the switch only actually executes once the channel is confirmed idle, not just "probably about to be idle."

---

You've actually just identified the real, load-bearing constraint of the whole scheme — and your skepticism is correct: **under a perfectly uniform round-robin interleaving pattern hitting every channel every few cycles, per-channel LPM genuinely has no benefit and can only hurt you.** If every channel is legitimately needed every 2-3 cycles, none of them are actually idle, and trying to park any of them would just manufacture wake-latency penalties for no power savings, since there's no real idle window to exploit in the first place.

So the honest correction to the earlier framing is: **LPM at channel/pseudo-channel/bank granularity is only useful when the actual access pattern is non-uniform — when some channels genuinely go idle for stretches meaningfully longer than the entry+exit overhead, while others stay busy.** It's never a good default policy to apply uniformly regardless of workload shape; it's a targeted response to skew.

## When does that skew actually happen in practice?

This matters for your research because you need a concrete answer to "when would a channel actually sit idle long enough," not just an abstract possibility:

- **Workload phase changes**: many real workloads (including the DNN layer-type variation NeuroCool/NeuroTAP exploit, which we found earlier) have non-uniform memory footprints over time — a phase that's compute-bound (matrix-multiply-heavy, weights already cached, little new data movement) can leave large swaths of memory quiet for a while, versus a phase that's bandwidth-bound and hits everything.
- **Sparse or irregular access patterns**: not every AI/embedded workload streams data uniformly across the whole address space — attention mechanisms, sparse models, or certain sensor-fusion pipelines can have working sets that concentrate on a subset of memory, leaving other regions (and thus their mapped channels) legitimately cold.
- **Explicit software-driven skew (Scenario B from before)**: if you _do_ have the capacity to migrate data (or the workload's natural data placement already happens to concentrate hot data on fewer channels), you can deliberately create this skew rather than waiting for it to occur naturally — this is the "migrate + reconfigure interleaving" path, which only makes sense as a _deliberate_ act, not something you'd try to layer on top of a workload that's already perfectly uniform.
- **The genuinely idle case (Scenario A)**: no computation happening at all — trivially, no channel is being interleaved into, so LPM is free.

## Why this actually sharpens your research contribution, rather than weakening it

This is a good thing to have surfaced, because it means the interesting research question isn't "should I do LPM" in the abstract — it's: **"how do I detect, in real time, whether the current workload phase has exploitable per-channel idleness, versus whether it's uniformly hammering every channel (in which case, don't even attempt fine-grained LPM at all, and fall back to coarse thermal handling like whole-stack DVFS or accepting the heat)."**

That detection logic — recognizing _when_ fine-grained LPM is worth attempting versus when it's actively counterproductive — is itself a legitimate, meaningful piece of a DVFS/DTM controller, and it's exactly the kind of nuance that separates a naive "always try to power down idle-looking resources" scheme (which your question just correctly showed can backfire under uniform interleaving) from a genuinely workload-aware one.

**One-line summary for your research framing:** "Per-channel LPM only pays off when the workload's actual access pattern is non-uniform enough to leave some channels genuinely idle for longer than the wake-up overhead — under a perfectly uniform round-robin interleaving pattern touching every channel every few cycles, no channel is ever a valid LPM candidate, and attempting it would only add latency without saving power. This means your scheme needs an explicit idleness/skew-detection stage that decides whether fine-grained LPM is even applicable to the current workload phase, rather than applying it unconditionally."

---

