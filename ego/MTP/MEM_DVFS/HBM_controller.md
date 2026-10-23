intellectual property - IP
reusable block of chip design.

### Why this specifically helps, and why it's tied to PC mode

Recall our extended discussion: in pseudo-channel mode, the command/address bus is a **shared, scarce resource** between the two pseudo-channels — every command you send competes for a limited number of C/A bus slots. Implicit precharge **collapses a two-command sequence (PRE + ACT) into a single command (ACT alone)**, cutting the C/A bus traffic needed for a row-conflict access in half. Given everything we discussed about pseudo-channels needing to interleave commands carefully across a shared bus to keep both busy, this is a meaningful, direct efficiency win specifically in that context — which is presumably why the paper calls it "exclusive to PC mode": it's a feature JEDEC added specifically to ease the C/A bus contention problem pseudo-channel mode introduces, rather than a general-purpose DRAM feature available in every mode.


the HBM stack consists of M independent
channels, each interfacing with the controller through a physical layer (PHY). The PHY itself is external to the scope of
our controller, but we only require it to expose a standard Double Data Rate (DDR) PHY Interface (DFI) for HBM, as
specified in [13]. The independent operation of these channels allows them to be treated as discrete memory spaces,
with each channel managed by a dedicated HBM-Channel-Controller component. These components communicate with the interconnect fabric via two ports employing a custom protocol, one port per PC. 

![[HBM_controller.png]]


The interconnect facilitates communication between each processing unit (PU) and all HBM-Channel-Controller components. Processing units interface with the interconnect utilizing the Advanced eXtensible Interface (AXI) protocol, while the custom protocol adheres to a conventional valid/ready handshake mechanism.

Fair enough — here's the gist in a few minutes' worth of reading.

**What AXI actually is:** a point-to-point on-chip communication protocol (part of ARM's AMBA family) connecting a **master** (e.g., CPU, DMA engine — something that initiates requests) to a **slave** (e.g., memory controller, peripheral — something that responds). It's not "a bus" in the shared-wire sense we discussed for DRAM — each master-slave link gets its own dedicated point-to-point connection, and an interconnect/crossbar fabric handles routing between multiple masters and slaves.

**The five channels** — this is the one structural fact worth remembering:

1. **Read Address channel** — master sends the address + burst info it wants to read
2. **Read Data channel** — slave sends the actual data back
3. **Write Address channel** — master sends the address it wants to write to
4. **Write Data channel** — master sends the actual data to write
5. **Write Response channel** — slave confirms the write succeeded

Reads and writes each have their **own independent address and data channels** — this is a deliberate design choice allowing reads and writes to happen concurrently rather than sharing one path (a bit like how we discussed HBM's pseudo-channels splitting command/data paths for independence, just at the SoC-interconnect level instead of memory-chip level).

**Key properties people usually cite as AXI's selling points:**

- **Handshake-based**: every channel uses simple VALID/READY signals — the sender asserts VALID when data's ready, the receiver asserts READY when it can accept it; transfer happens only when both are high simultaneously.
- **Burst-capable**: one address can cover a burst of multiple data beats (conceptually similar to the DRAM burst/prefetch idea we covered — amortize address overhead across multiple data transfers).
- **Outstanding/multiple transactions with ID tags**: a master can have several requests in flight at once, tagged with IDs, and responses can come back out of order — same fundamental problem/solution as the memory-controller read-buffer/tagging mechanism we discussed for HBM pseudo-channels.
- **Variants**: AXI4 (full-featured), AXI4-Lite (simplified, single-transaction, good for simple control/register interfaces), AXI4-Stream (pure continuous data streaming, no addresses at all).

That's the shape of it — happy to go deeper on any one piece (burst types, ID ordering, handshake timing) whenever you actually need it.

 To achieve this, the address of the most recently
opened row is stored in a register. Upon receiving a new request, a comparator determines whether the requested row
matches the stored address. If they match, only the corresponding CAS command is added to the FIFO queue. Otherwise,
if the addresses differ, a sequence of PRE-ACT-CAS commands is enqueued. The REQ-to-CMD-Translator completes
the translation and enqueues commands within a single clock cycle, provided that there is enough space in the queue.
If the queue is full, the translation process is delayed until sufficient space becomes available. The queue length is a
configurable parameter that can be tuned according to the workload characteristics. For highly parallel workloads with
accesses targeting different banks, longer queues help prevent stalls by accommodating multiple concurrent requests.
Conversely, for more serial workloads, a shorter queue is preferable to avoid unnecessary resource consumption.

 When a command is sent to the Channel-Scheduler,
the corresponding command-type timer is reset. Essentially, these timers track the elapsed time since the last execution
of each specific command type. This allows the Checker to efficiently compare the incoming command type with
all the timers and determine whether the command can proceed. For example, when a CAS command arrives at the
Internal-Register, the Checker must verify the 𝐴�𝐴�𝐴�𝐴� timing constraint. It does so by examining the timer associated
with the ACT command type. If the timer value exceeds 𝐴�𝐴�𝐴�𝐴�, the CAS command is allowed to proceed, ensuring
that sufficient time has passed since the last ACT command. Of course, if the timing constraints are not met or the
Channel-Controller is not ready, the command remains stalled in the Internal-Register until both conditions are satisfied.
Note that since the next component, the Channel-Scheduler, includes additional pipeline stages and arbiters that may
reorder commands, a command forwarded to the Channel-Scheduler is not immediately issued to memory. This can
potentially lead to timing violations. To prevent such issues, the timers are not reset when the command is merely
forwarded to the Channel-Scheduler but rather when the Channel-Scheduler acknowledges to the Bank-Scheduler that
the command has been properly issued to memory. This acknowledgment is conveyed through additional signals from
the Channel-Scheduler to the Bank-Scheduler,


First, note that the memory operates at twice the frequency of the DFI interface, and
hence the controller. Consequently, each signal exchanged with the memory is duplicated and labeled with the phase suffixes "p0" and "p1". During the first clock cycle, the memory decodes and executes the signals designated as "p0,"
while during the second clock cycle, it processes the signals marked as "p1". Although the memory processes the signals
over two clock cycles, from the controller’s perspective, the operation completes in a single cycle. This effectively
creates the impression of two parallel interfaces. We leverage this opportunity to transmit commands and data for both
pseudo-channels concurrently within a single clock cycle. As a result, the controller treats the pseudo-channels as
independent entities, much like the channels themselves. The only command that does not conform to this framework
is the ACT command, as it occupies two memory clock cycles (i.e., both phase signals). Consequently, multiple ACT
commands cannot be issued for the two pseudo-channels simultaneously; one must wait for the other to complete.

In PC-mode, BL = 4, meaning
that each WR or RD command transfers 64 × 4 = 256 bits of data per pseudo-channel. Because the memory operates in
double-data-rate (DDR) mode, these four bursts are completed over two memory clock cycles. Hence, the controller
perceives the entire operation as occurring within a single clock cycle.

Let's unpack this piece by piece — it directly resolves the tension from a few turns ago, where we established the C/A bus is shared between pseudo-channels and concluded commands must be serialized. This paragraph explains the actual trick that makes two pseudo-channels _feel_ independent from the controller's point of view, despite that shared physical wire.

## What DFI is, and why the 2x frequency relationship exists

**DFI (DDR PHY Interface)** is the standard interface between the memory **controller** (the digital logic making scheduling decisions — everything we've discussed: ACT/PRE/RD/WR sequencing, bank tracking, address mapping) and the **PHY** (the analog/mixed-signal circuitry that actually drives the high-speed electrical signals out to the memory, including the DLL-based clock alignment we discussed earlier).

The controller logic is complex and runs comfortably at a lower clock frequency, while the actual memory interface needs to run much faster to hit DDR's high data rates. Rather than making the entire controller run at the memory's full speed (hard to design, verify, and close timing on), the industry-standard approach is: **the controller/DFI side runs at half the memory's clock frequency**, and the PHY handles converting between the two domains. This is exactly the kind of "wide-and-slow on one side, narrow-and-fast on the other, serialized in between" pattern we've seen repeatedly in this conversation (recall prefetch-depth × rank-width, and burst serialization) — same idea, just applied at the controller-to-PHY boundary instead of the array-to-pin boundary.

## What "p0" and "p1" actually represent

Since the controller/DFI operates at **half** the memory's frequency, every **one DFI clock cycle corresponds to two actual memory clock cycles**. So for any given signal the controller wants to send to memory, there are really **two opportunities to say something** within one DFI cycle — one corresponding to the memory's first clock edge, one corresponding to its second. These two opportunities are labeled **p0** (phase 0, first memory cycle) and **p1** (phase 1, second memory cycle).

So on the DFI bus, every signal type is literally duplicated into a "_p0" version and a "_p1" version — e.g., you'd see `dfi_address_p0` and `dfi_address_p1` as separate signal buses in the actual interface. The controller drives both simultaneously, once per DFI cycle, and the PHY is responsible for taking that single DFI cycle's p0/p1 pair and re-timing/serializing it out as two sequential events at the memory's actual (faster) clock rate.

## Why this "feels like one cycle" to the controller, but is really two to the memory

This is the key abstraction the paragraph is describing: from the **controller's perspective**, it issues one unified command/data payload per DFI cycle — it doesn't have to think about the memory-side timing at all, it just hands off a p0+p1 bundle and moves on. The **memory**, however, genuinely processes this as two sequential steps — decoding/executing whatever was in the p0 slot during its first clock edge, then whatever was in the p1 slot during its second. The abstraction layer (PHY) hides this two-step reality from the controller, making it look like "the whole thing happened in one controller cycle" even though two real, sequential actions occurred at the memory.

## The clever exploitation: using p0 and p1 for two _different_ pseudo-channels

Here's where this connects directly to our pseudo-channel discussion. Since you have **two independent slots** (p0 and p1) available within a single DFI cycle, and since the two pseudo-channels need to be commanded independently anyway (recall: BA4 routes each command to one specific pseudo-channel), the design puts **pseudo-channel 0's command in the p0 slot and pseudo-channel 1's command in the p1 slot**, within the _same_ DFI cycle.

So rather than the controller having to explicitly serialize — "send PC0's command now, wait, then send PC1's command next" — it just hands off both at once, packed into the p0/p1 structure that already exists for an entirely different reason (the controller/memory frequency ratio). The controller gets to treat the two pseudo-channels as if they're fully independent, simultaneously-addressable entities (exactly like full channels), **without actually needing a wider, separate, truly-parallel C/A bus** — it's reusing the existing p0/p1 duplication as a free, already-built mechanism for carrying two independent commands per cycle.

This is a neat resolution of the earlier tension: the C/A bus being "shared" doesn't mean strictly one command per DFI cycle — it means one command _per phase_, and there happen to be two phases per cycle, conveniently matching the two pseudo-channels.

## Why ACT is the exception

Here's the subtlety that breaks the otherwise-clean p0/PC0, p1/PC1 pairing: a **row address is wide** — it needs to specify which of potentially thousands of rows in a bank to open, which requires more address bits than a column command (RD/WR) does, since column addresses only need to select among a much smaller number of columns within an already-open row.

Because of this, **ACT's command payload doesn't fit into a single phase slot** — it needs the full combined bandwidth of _both_ p0 and p1 together just to carry one ACT command's complete row-address information. So when an ACT needs to be issued, it **consumes the entire DFI cycle's p0+p1 capacity for itself**, leaving no room to simultaneously slip in a second, different ACT command for the other pseudo-channel in that same cycle.

This is exactly what the paragraph states: _"multiple ACT commands cannot be issued for the two pseudo-channels simultaneously; one must wait for the other to complete."_ Concretely — if PC0 needs an ACT and PC1 also needs an ACT around the same time, they **cannot** be packed into the same DFI cycle the way a PC0-RD + PC1-WR pair could be; the controller must issue PC0's ACT in one full DFI cycle (using both phases), then issue PC1's ACT in a subsequent DFI cycle.

## Why this matters for your research

This is a genuinely important, concrete refinement to the "pseudo-channels share a C/A bus, so commands are time-multiplexed" picture we built earlier — it shows the _actual_ granularity of that sharing is more favorable than naive serialization for most commands (RD/WR can genuinely go out concurrently for both pseudo-channels, one per phase), but **ACT specifically is still a true bottleneck point** between the two pseudo-channels. For your DVFS/thermal research, this means: if your workload/thermal model is tracking C/A bus contention between pseudo-channels, **ACT-heavy phases (frequent row changes, poor row-buffer locality) will show real contention between PC0 and PC1's activation requests**, while **RD/WR-heavy phases (good page-hit locality) can proceed with much less inter-pseudo-channel command contention** — this access-pattern-dependent distinction could be a useful, precise signal to fold into your workload-skew detection logic.

**One-line summary for interview/research purposes:** "DFI runs at half the memory's clock frequency, so every DFI cycle carries two phase-slots (p0, p1) corresponding to the memory's two faster clock edges — the design exploits this by assigning one pseudo-channel to each phase, letting the controller issue two independent commands (one per pseudo-channel) in what looks like a single cycle. The exception is ACT, whose wide row-address payload needs both phases for itself, meaning the two pseudo-channels' ACT commands are forced to serialize across separate DFI cycles even though RD/WR commands can genuinely go out concurrently."

Good follow-up — this is a different kind of "why" than the DDR one. The DDR 2-transfers-per-cycle fact is a **hardware/signaling definition** (it's just what DDR means). The DFI-runs-at-half-frequency fact is a **deliberate design choice**, made for practical engineering reasons — not a law of physics, but a tradeoff the industry settled on.

## The core reason: digital logic doesn't scale to memory-pin speeds easily

Recall the whole DLL/phase-alignment discussion from earlier — getting signals to switch cleanly and reliably at the very high frequencies modern memory interfaces run at (multiple GHz-class data rates) requires careful, specialized circuit design: controlled impedance, precise clock distribution, tight timing margins, custom analog/mixed-signal techniques.

The **controller** (the part of the chip doing command scheduling, bank-state tracking, address mapping — everything in digital logic) is a large, complex block of standard digital logic. Digital logic synthesized from RTL (the kind of design flow used for controllers) doesn't comfortably reach the same raw clock speeds that a carefully hand-tuned, physically-optimized PHY can achieve. Trying to make the _entire_ controller run at the memory's full pin-speed clock would be extremely difficult to design, verify, and close timing on — you'd be forcing ordinary synthesized logic to operate at speeds it's not well-suited for.

## The solution: split the problem into two domains

Instead, the industry splits responsibilities:

- **Controller (DFI side)**: runs at a slower, more comfortable clock frequency, where standard digital design techniques work fine and timing closure is tractable. It does all the "thinking" — scheduling, which bank, which row, which command — without worrying about pin-level speed at all.
- **PHY**: a specialized, physically-optimized circuit block (closer to analog/mixed-signal design) that handles the actual high-speed signal generation, serialization, and the DLL/clock-alignment work we discussed earlier. This block is specifically engineered to handle the memory's true, fast clock domain.

The DFI standard defines exactly how these two pieces talk to each other, and the **2:1 frequency ratio is the specific ratio DFI chose** as the interface contract — it's low enough that the controller's digital logic stays comfortably implementable, while still being a simple, clean (power-of-2) relationship for the PHY to manage converting between domains.

## Why 2:1 specifically, rather than some other ratio

A clean 2:1 (or sometimes 4:1 in some DFI configurations, for even lower controller frequency) ratio is chosen because it's **easy to deserialize/serialize cleanly** — recall the p0/p1 phase structure from before: at a 2:1 ratio, each DFI cycle maps to exactly 2 memory cycles, a simple, fixed, predictable relationship. This keeps the PHY's job (re-timing between the two clock domains) straightforward and avoids any messy fractional-cycle alignment problems. A non-power-of-2 or irregular ratio would make this phase-mapping significantly harder to implement cleanly in hardware.

## Tying it back to your earlier question

So to directly connect the dots: DDR's 2-transfers-per-cycle is about **how many bits move per clock edge, physically on the wire** — a signaling-layer fact. DFI's 2:1 cycle ratio is about **which clock domain is convenient for which piece of logic to run in** — a chip-design/architecture-layer choice. They're independent decisions that happen to compound multiplicatively (as we saw: BL=4 → 2 memory cycles → exactly 1 DFI cycle), but they exist for completely different underlying reasons — one is inherent to the signaling standard, the other is a practical digital-design tradeoff for making controller logic implementable at reasonable speeds.

**One-line summary for your research framing:** "DFI's 2:1 memory-to-controller frequency ratio isn't a signaling requirement like DDR's double-data-rate — it's a deliberate chip-design choice that lets the controller's complex digital logic run at a comfortable, easily-timing-closed frequency, while a separately-optimized PHY block handles the actual high-speed pin-level signaling and clock-domain conversion, with a clean power-of-2 ratio chosen specifically to keep that domain conversion simple and predictable."

since each channel has its own controller, we can control freq at channel level.

Good question to loop back to, because this ties the data-storage-layout question directly to the LPM viability question we've been circling.

## First, to directly settle "does this cause data issues" — no, not corruption

Same answer as a few turns back: parking a pseudo-channel never causes data loss or incorrect reads — it just adds wake-up latency when that pseudo-channel's turn in the interleaving sequence comes up. What you're really asking, I think, is whether AI model storage patterns make this latency cost _unavoidably frequent_ (which would make PC-level LPM pointless) or whether there's exploitable structure (which would make it viable). Let's work through how model data actually gets laid out.

## How AI model weights actually end up interleaved across pseudo-channels

This happens at the **hardware address-mapping level**, transparently below what the software/framework sees — exactly the interleaving mechanism we discussed for channels generally, just now applied concretely to tensor storage:

- When a framework (PyTorch, TensorFlow, etc.) allocates memory for a weight tensor, it gets a contiguous **virtual/physical address range** — from the software's perspective, the tensor just looks like one unbroken block.
- The **memory controller's address-mapping function** then takes that linear address range and scatters it across the actual hardware resources — typically selecting specific bits of the address to determine channel, then pseudo-channel (recall BA4), then bank, then row/column. This is usually done at a **fine granularity** — often matched to a cache-line size (64 bytes, which we derived earlier from the DDR prefetch/burst-length math) — specifically **to maximize bandwidth utilization** for exactly this kind of large, sequential, bandwidth-hungry streaming access (streaming weight matrices through a matmul is about as close to the "ideal" high-bandwidth streaming pattern as workloads get).

So yes — **for a weight tensor actively being streamed during inference/training compute, the data is very likely interleaved tightly across both pseudo-channels (and across banks, channels) by design**, precisely because that's what makes the matmul fast in the first place. This is not an accident or an oversight — it's the hardware doing exactly what it's supposed to do to serve a bandwidth-bound workload.

## Why this directly confirms the "uniform access = no LPM benefit" conclusion from before

This is the same situation we already reasoned through two turns back, just now grounded in a concrete, realistic example: **while a given tensor is actively being read for compute, its fine-grained interleaving means both pseudo-channels are genuinely needed, continuously, in near-lockstep.** Parking either one during this active phase would not cause data errors, but it would cause exactly the "every 2-3 cycles I need this PC back" problem you identified — the wake-latency cost would likely exceed any power benefit, because there's no real idle window to exploit during active tensor streaming.

## Where the exploitable opportunity actually is

This points you toward the **temporal** dimension rather than trying to fight the spatial interleaving:

- **Between layers / between tensors, not within one**: a neural network executes layer by layer, or tensor by tensor. While layer N's weights are being actively streamed (both pseudo-channels genuinely busy, interleaved), the memory region holding layer N+5's weights (not yet needed) is sitting completely idle — regardless of how finely _that_ region's data happens to be interleaved internally. This is a real, predictable, model-structure-driven idle window your controller could exploit — not idleness in the "middle of streaming one tensor" sense, but idleness in the "haven't gotten to this tensor's memory region yet" sense.
- **Sparse/attention/MoE-style workloads**: as mentioned a few turns ago, not every modern architecture streams uniformly — Mixture-of-Experts models, for instance, only activate a subset of expert weight tensors per token, leaving other experts' memory regions genuinely unused for stretches — again, exploitable idleness at the _tensor/region_ level, not something you'd discover by looking at fine-grained interleaving within an active region.
- **Allocator/mapping granularity, if you control it**: if your research context allows co-design with the memory allocator (less likely if you're purely working at the controller/architecture level, but worth noting), choosing a coarser interleaving granularity for specific allocations — keeping some tensors entirely within one pseudo-channel/channel rather than striping them finely — would directly create the kind of per-PC idleness your LPM scheme wants, at the cost of giving up some bandwidth for that particular tensor's access. This is a genuine tradeoff knob, not something hardware forces on you by default.

## Direct answer to "how is an AI model actually stored"

A model's weights are stored as large, allocator-contiguous blocks in the _logical_ address space, but the memory controller's hardware interleaving function **physically scatters each tensor's bytes across pseudo-channels (and channels, banks) at fine granularity** — so a single tensor is essentially never confined to one pseudo-channel; it's deliberately spread to maximize the bandwidth available for reading it quickly during compute.

**One-line summary for your research framing:** "AI model weights are stored contiguously in logical address space but physically interleaved at fine (cache-line) granularity across pseudo-channels by the hardware address map, specifically to maximize streaming bandwidth — so during active compute on a given tensor, both pseudo-channels are genuinely needed continuously, making PC-level LPM pointless for that window. The real opportunity for PC-level LPM is temporal, not spatial: idle time _between_ tensors or layers not currently being accessed (or unused expert regions in MoE-style models), where a pseudo-channel's banks genuinely go idle for a predictable duration, rather than trying to find idleness within an actively-streamed tensor's own interleaved footprint."

Good effort, but there's a mixing-up of two different relationships — let's fix the math and the model.

## The core correction: PC0 and PC1 don't take turns on the data bus — they transfer concurrently, on separate wires

This is the key error. Recall: each pseudo-channel has its **own dedicated 64-bit data path** — its own separate set of TSVs, physically independent from PC1's. They are **not** sharing one wire and taking turns (that would only be true if they shared data pins, which they don't — only the C/A bus and clock are shared).

So within the same 2 memory clock cycles:

- **Memory cycle 1**: PC0 transfers its first 2 beats (128 of its 256 bits) on its own 64 wires, **simultaneously** PC1 transfers its first 2 beats (128 of its own 256 bits) on its own separate 64 wires
- **Memory cycle 2**: PC0 transfers its remaining 128 bits, **simultaneously** PC1 transfers its remaining 128 bits

Both pseudo-channels are active across **both** memory cycles concurrently, not PC0-then-PC1 sequentially. This is exactly consistent with what we established the command side does (p0 carries PC0's command, p1 carries PC1's command, within the same DFI cycle) — the data side mirrors this concurrency, just on genuinely separate physical wires rather than time-sliced phases of a shared wire.

## Fixing the total

- PC0: 64 bits × BL4 = 256 bits
- PC1: 64 bits × BL4 = 256 bits
- **Total per channel, per DFI cycle: 256 + 256 = 512 bits = 64 bytes**

This is exactly the cache-line-matching figure from way earlier in our conversation — good, it's self-consistent with everything we've built up. Your "256 bits" came from accidentally only counting one pseudo-channel's contribution (as if the other one's data arrived in a separate, later cycle rather than concurrently).

## "So do we access one channel per cycle?"

No — and this is the second thing worth correcting. **All channels in the stack can be doing this exact 2-pseudo-channel-concurrent-transfer dance simultaneously, every DFI cycle**, because — as we established a few turns back with direct citations — channels have their own independent clocks, their own independent TSVs, entirely separate from every other channel. There's no "wait your turn" between channels the way there is between commands sharing one C/A bus within a channel.

So per DFI cycle, the **aggregate** figure is: 64 bytes per channel × number of channels (8 for HBM2, 16 for HBM3, 32 for HBM4) — all happening concurrently. This is exactly how you get from "64 bytes per channel per cycle" up to the terabyte-per-second aggregate bandwidth figures from the very start of our conversation.

## Is data interleaved between channels?

Yes — same address-mapping mechanism we discussed for pseudo-channels, just one level up: specific address bits select which channel a given access lands on, and the controller's interleaving policy typically spreads consecutive/nearby addresses across channels for the same reason (maximize parallel utilization).

## "Between layers" — if you mean the stacked DRAM dies

This is a slightly different relationship than address interleaving, worth being precise about: channels aren't so much "interleaved across dies" in the address-mapping sense — they're **physically hosted on specific dies** to begin with. In a typical HBM stack, each physical DRAM die implements a fixed subset of the stack's total channels (e.g., commonly 2 channels per die in some configurations), wired down through its own set of TSVs to the base die. So when the controller's address map sends traffic to "channel 5," that traffic is, by construction, going to whichever specific physical die hosts channel 5 — the controller doesn't choose which die a channel lives on; that's fixed by the stack's physical design. The _illusion_ of dies working together comes from the fact that accessing many channels in parallel (which the controller's interleaving does encourage) naturally means accessing many different dies in parallel too, simply because each die owns a different slice of the channel address space.

**One-line summary for your research framing:** "PC0 and PC1 transfer concurrently on separate dedicated wires across both memory cycles, not sequentially — giving 512 bits (64 bytes) per channel per DFI cycle. All channels do this simultaneously too, since they're independently clocked and wired, which is how per-channel bandwidth aggregates up to the stack's total terabyte/s figures. Data is interleaved across channels by address mapping, while 'interleaving across dies' is really a side effect of channels each being physically hosted on specific dies, not a separate mapping mechanism."