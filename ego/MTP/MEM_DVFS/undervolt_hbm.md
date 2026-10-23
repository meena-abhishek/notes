

---

Undervolting is one such technique: it reduces the supply
voltage to decrease power consumption without reducing the device’s
operating frequency to avoid performance loss.
Undervolting takes
advantage of voltage guardbands put in place by manufacturers to
ensure correct operation under all environmental conditions. However,
reducing voltage without changing frequency can lead to reliability
issues manifested as unwanted bit flips.

Since HBM
targets high-performance applications, any power saving technique
with bandwidth overhead is undesirable.


No — not in the sense of issuing commands to both at once. Here's why.
You said it yourself: PCs share the **clock and command signals**. That shared command/address bus is time-multiplexed — in any given command cycle, that bus carries one command, addressed to _one_ pseudo-channel. You can't put two different row/column commands on the same wires in the same cycle. So PC0 and PC1 cannot both receive a new command in the same clock cycle.

The benefit isn't simultaneous command issue — it's **pipelined overlap** between command and data phases:

- Cycle N: issue a command to PC0
- Cycle N+1: issue a different command to PC1 (command bus is free again)
- Meanwhile, PC0's data burst is transferring on _its own_ data bus while PC1's command is being issued, and PC1's subsequent data transfer happens on _its_ bus, independent of PC0's

So the two PCs' **data transfers can overlap in time**, and each interprets/executes its command independently once received (independent row/column decode, independent sense amps, independent 256MB array) — but the _commands themselves_ are serialized onto the shared bus, just interleaved cycle-by-cycle between the two PCs.

Net effect,
This is exactly the JEDEC HBM2 pseudo-channel design intent: you don't get true dual simultaneous random access, but you get finer-grained addressing (smaller effective burst/access granularity) and better bus utilization than a single monolithic channel, because data-phase latency of one PC can be hidden behind command/data activity of the other. It's concurrency through pipelining, not through parallel command issue.

*What separate clock/command per PC would cost*
Command/address buses and clock trees aren't free — they need:
- **TSVs** (through-silicon vias) — every additional independent signal needs its own via running through the whole stack, and TSV area is genuinely scarce real estate on an HBM die (you saw in the patent's Fig. 2B how TSV arrays `226`/`230` already eat significant die area). Doubling the command/address/clock TSVs per channel roughly doubles that overhead.
- **Duplicate command decode logic and clock distribution trees** on each die, which costs power and area on every single die in the stack, not just once

### The 128b MC width is an _aggregate_ number, not a per-access guarantee

The MC's 128-bit width describes the **total physical data-bus width available to that channel** — it's the sum of the two 64-bit PC data buses running side by side. It does **not** mean a single access pulls 128 bits together as one atomic transfer.

### What actually happens on a single access

Since each PC has its own independent row/column decode and its own 64-bit data bus, but the **command bus is shared**, a single command can only target _one_ PC at a time. So:

- One command → PC0 → returns a 64-bit-wide burst (with standard burst length, e.g. BL4 → 32 bytes total)
- A separate command (issued in the next available command slot) → PC1 → returns its own 64-bit-wide burst

**So a single logical access only gets you 64 bits wide**, addressed to whichever PC that command targeted — not 128 bits.

### So when do you actually use the full 128b?

You get the full 128-bit _channel_ width utilized only when **both PCs are kept busy with back-to-back commands**, interleaved on the shared command bus, with their data transfers overlapping in time on their separate data buses (this is the pipelining benefit from your earlier question). Over a stretch of time, the channel's aggregate throughput approaches 128 bits' worth of bandwidth, but that's the _sum of two independent 64-bit streams_, not one 128-bit-wide burst.

### Can you deliberately fetch 128 bits for one logical access?

Only by issuing matched commands to both PCs (same address pattern, different PC) back-to-back and treating the two 64-bit results as one 128-bit chunk in the memory controller. This is possible, but it's a controller-level choice, not something the hardware does automatically as a single transaction — and it costs you two command slots instead of one, so you don't save latency by doing it, you just guarantee data locality.
### Core design ideas

**1. Separate channels for everything**  
AXI splits communication into 5 independent channels, each with its own handshake:

- **Read Address (AR)** — "I want to read from this address"
- **Read Data (R)** — the actual data coming back
- **Write Address (AW)** — "I want to write to this address"
- **Write Data (W)** — the data being written
- **Write Response (B)** — "your write completed"

Because these are separate, a master can send a new address while still waiting on previous data — this is what enables pipelining.

**2. Point-to-point, not shared bus**  
Unlike an old-style shared bus where only one transaction happens at a time, AXI is a **point-to-point interconnect** — multiple masters and slaves can have simultaneous, independent transactions in flight at once, arbitrated by an interconnect fabric between them.

**3. Outstanding transactions / burst support**  
A master can issue multiple outstanding requests before earlier ones complete (out-of-order completion is allowed, tracked via transaction IDs), and each request can be a burst (multiple data beats per single address phase) — reducing address-phase overhead for large transfers.

**4. Simple valid/ready handshake**  
Every channel uses the same two-signal protocol: `VALID` (sender has data) and `READY` (receiver can accept it). Data transfers only when both are asserted in the same cycle — this decouples producer and consumer timing.

An AXI port is 256b wide, which provides a 4:1
data width ratio over a PC (with 64b width). As a result, an AXI
port can operate at a clock frequency that is a quarter of the memory
data transfer rate (1:4 ratio) and yet take advantage of the maximum
HBM bandwidth provided by PCs [65]
The maximum clock
frequency allowed for memory arrays in our device is 900MHz, and
being a double data rate memory, it translates to a maximum data
transfer rate of 1800 Mega-Transfers per second (MT/s).

Below Vmin, a triple-factor trade-off is at the user’s disposal.
For example, up to 1.6X power savings is achievable for an
application that cannot tolerate any faults but can work with
smaller memory capacity, by using only 7 fault-free PCs
operating at 0.95V

---
DDVFS's fix: give different zones of the memory their own independent throttling, so only the actually-hot zone slows down.

Threshold voltage detector:
it infers local heating from _threshold-voltage drift_ of an actual transistor near the bank, rather than a dedicated thermometer.

