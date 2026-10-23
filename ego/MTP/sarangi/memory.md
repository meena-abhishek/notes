![[dram_cell.png|200]]


role of memory controller is to take all the memory requests from cache, queue them, schedule them, and send them to main memory.
scheduling algorithm is crucial determinant of overall performance.

memory wall, memory capacity is increasing very fast, but memory latency is decreasing slowly.

on-chip network NoC is important for fast communication between processing and memory elements.

each SRAM cell requires 6-transistors, storage capacity is limited

writing to a SRAM, set $BL$ to 1 , $\overline{BL}$ to 0, two outputs of cross-coupled inverter

for writing, we use write drivers that can pump a lot of current into the bit lines.

we can also use cross-coupled NANDs to store a bit but we need 8 transistors, 4 each.
but inverters can be made of 2 transistors, a CMOS made up of 1 PMOS transistor and 1 NMOS transistor. total transistors needed for cross-coupled inverter = 4. area efficient.

to allow enabling/disabling the SRAM cell, we connect two transistors to $Q$ and $\overline{Q}$. thus, total 6 transistors for a SRAM cell.

power is supplied to each cell, so value is continously generated, active bistable circuit.

when reading, the cell drives one bitline to 1 and other to 0, but this process is very slow.

we use *precharging*, which sets voltage of bitlines that is midway of logical 0 and 1 voltages. we measure voltage difference between bitlines. once it crosses a certain threshold, we can infer stored bit.

we apply +ve voltage in NMOS to turn on and -ve to PMOS

a sense amplifier is a differential voltage amplifier that converts a small voltage swing to a logical level 0 or 1.

DRAM sense amplifiers acts as buffer and also used to restore charge.

only subarray bitline gets precharged, not entire length.

- open bit line array architecture (more noise induced errors)
- folded bit line array architecture

writing a DRAM cell

first stage, precharge -> row decode -> enable row -> sense -> restore
second stage, column decode -> enable write drivers -> override sense amplifier, bitlines, dram cell.

the read process ineffect restores the charges in capacitors. we don't need column decoder, chip select for this.

burst refresh, freeze entire DRAM array, refresh all rows 1 by 1.

distributed refresh,
refresh accesses are interspersed with regular memory accesses, this hides the overhead of refresh operations, moreover, we don't refresh rows that do not contain valid data.

SRAM is faster coz:
- SRAM cells actively drive bitline using power supply current(fast) DRAM cells only perturb bitline via charge sharing (slow)
- read access is basically read+write(restore), overhead, bitlines are actively drived to 0 volts using sense amplifier cc inverters. so capacitor drains actively after perturbation.
- refresh overhead

eDRAM, embedded on same die as processor.

generic architecture,

a processor can have many memory controllers. the physical address space needs to be partitioned across these memory controllers.
each memory controller is connected to a set of DRAM arrays via a set of copper wires. These set of wires are known as *channels*.

a channel is typically 32-128 bits wide. channels widths are getting shorter with time mainly because if we are sending data at high frequency; it is hard to keep the data across the different copper wires in the channel synchronized.

the channels are connected to a set of PCBs that contain DRAM chips.
these PCBs are known as DIMMs.

each DIMM contains a set of DRAM *chips*. we typically divide a DIMM into multiple ranks (typically 1 to 4), where each *rank* contains a set of DRAM chips that execute in lockstep.
this keeps each individual DRAM device small and power efficient.

each DRAM chip has multiple *banks*. they operate independently of other banks on the same chip.
arrays within each bank work in synchrony.
more arrays increases bandwidth.

In a DIMM, the front and back contacts are not tied together internally, the one on the front carries a completely different, independent signal from the one directly behind it on the back. So for the same physical length of edge connector, you get twice as many independent electrical connections, which is why DIMMs can support a much wider data bus than SIMMs of similar physical size.

channel -> DIMM -> rank -> chip -> bank -> array -> row -> column

![[dram_heirarchy.png|500]]


channel: address, data, command, chip-select

chip-select is used to select a specific rank.

![[organization_dram.png|450]]

address/command bus have higher capacitive loading and are consequently much slower because they are connected to all DRAM banks(16banks). data bus is fast coz it is only connected to one rank (4banks).

memory controller needs to know timing and commands of devices attached to memory channels.

asynchronous transfer, 
memory and core don't share clock. strobe signal(DQS), data signal(DQ)

![[signal_async_dram.png|500]]

memory controller first sends row address on address bus and after some time $\overline{RAS}$ row address strobe, so that memory reads row address, we delay strobe so that row address becomes valid on address bus.
then it sends column address and $\overline{CAS}$ and one bit indicating read/write.

read access,
device sets DQ data bus and sets $\overline{CAS}$ to 1, indicating MC can read.

fast-page mode, no need to send row address again if reading from the same row stored in sense amplifiers.

extended data out, while data is trasferring from dram, send another column request instead of waiting for complete data to arrive and then sending address

burst extended data out, generate next column addresses internally.
column prefetching.

![[extended_data_modes.png|700]]


in async dram, we are approximating the time for a operation to finish by dram device, but using a clock make operations timings predictable.
this enables pipelining, because in async DRAM completion times aren't exact (risk of overlapping).

![[timing_sdram.png]]


![[timing_DDR.png]]


because of the fundamental asymmetry in the speeds of data bus and address and command buses, DDR SDRAM was developed.

DQS, used for clock synchronization, fixes clock skews

clock skew - clock arriving at differenct units at slightly diff. time

lets say rows are already opened in rank 1, then we can switch to it from rank 0 and issue READ without issueing ACTIVATE.

QDR, fixes half-duplex data bus problem by using two data buses, delay in switching directions of data bus, read/write.

DDR performs best with long streams of only reads or only writes.


$MT/s$ - million transfers per second
one transfer equal to bus width of data transfered.

dram internal clock rate is much lower than bus clock.
this can only be equalized if internal bus width of dram device is more

as we connect more DIMMs to the channel, capacitive loading on addr, data, cmd bus increases, which impacts their RC delay.
RC delay or the time constant is the time it takes to charge the bus to 63% percent of its final value.

more connected devices, more time to effect volatage transitions on the bus, this directly places limits on the bus frequency.

RDIMMs - registered DIMMs

use a register that buffers address,commands,control signals
reduce capacitive load on controller,
controller -> register -> dimm instead of controller -> dimm
by reducing capacitive load, signals transfers faster, increasing frequency.

popular in server market, but need specific motherboard


![[ddr4_states.png]]

READ - read column and keep the row active
READA - read and precharge the bitlines

row activation is extremely power hungry operation.

we discourage consecutive row activations in the same bank group, which needs to be done to limit power consumption and local temperature rise.
define a activation window, under which only certain number of row activations are allowed.

we use preamble/postamble to fix clock synchronization issues when operating at high frequencies.

we use DQS to synchronize receiving/sending data. after controller issues write/read, device expects that after some latency send/receive DQ + DQS.

same bank col/row access delay is larger than switching banks and then accessing row/col. 

refresh controller generates list of addresses needs to be refreshed.

*memory controller*

we typically have one memory controller per channel.

![[memory_controller.png]]

we divide DRAM chips in ranks, coz of their distance from memory controller might vary, we groups chips that have roughly the same degree of clock skew.

accessing a bank means, accessing the same bank id across all chips

write caching, stores all outstanding writes. read operation first check this write buffer.

we prioritize reads, because write takes recovery time, read in sense amplifier -> modify column -> charge/dis capacitor (this takes time)

open-page access policy, where after a read or write operation completes, we do not set the bank to the idle state.
most memory controllers reorder requests such that contiguous requests map to same open page.

equalization circuits maintain bitlines current and compensate leakage

close page access policy, closes the row after it has been accessed once. favors random access pattern.

most memory controllers uses hybrid access policy.

self & external refresh

we read/write data at granularity of 64-byte blocks, 
each row stores an integral number of 64-byte blocks,
we transer 64-bit in each beat.

we can read consecutive blocks in parallel if they are mapped to different channels and we use lsb to decide a channel.

switching rank takes time, different banks have different timing requirements and different clock skews, re-synchronization needed

banks are independent of each other, we can activate bank 2, while bank 1 is completing a read operation.
we can use a different bank group.

open-page, $r:k:b:g:l:c$
closed-page, $r:l:k:b:g:c$

BRR,RRR,Greedy

NVMs can be used as a cache for hard disk.

*HBM,HMC*

composed of multiple 2D layers stacked over each other.
3D stack of DRAM memory arrays connected to each other via TSVs (through silicon vias) or microbumps (small metallic structure) that form high-bandwidth connections across the layers.

we divide each layer into a set of blocks. column of blocks is a vault.

common config is to put silicon die and 3D memory side by side.
connected by high-bandwidth connection known as interposer.
typically consisting of several 128-bit wide links.

we can read many bits in parallel from vaults.
because of close proximity we can have very wide bus.

tsv,
drill trenches into silicon wafer, fill with conductive copper.

using a wide parallel data bus in DDRx system increases the problem of crosstalk and signal integrity.

each HMC memory consists of 32 or 64 vaults, each vault made up of 4 to 8 high DRAM stacks. each vault runs independently.

each vault has its own controller.

![[HBM3e.jpg]]
![[HBM4.webm]]

![[hbm.png|500]]


interposer,
this piece of silicon, contains very fine wiring (TSVs + micro-bumps), thousands of short wide wires. 1024 bits in HBM4

substrate, 
base that connects interposer to motherboard via solder balls
handles power delivery and lower speed signals

wide wires decrease resistance (capacitance slightly increased) -> 
signal integrity (less resistive loss), lower energy per bit needed

generalized neural netword behaviour,
arrange all the inputs and weights into a matrix, then multiply and accumulate the results using a systolic array.

![[tensors.png|600]]

TPUs performs tensor multiplication

- processors used in AI are powerful math machines.
- matrix multiply-accumulate (MAC)
- increase in math capability drives increase in memory bandwidth demand
- size of generative ai models and iteration derives increase in memory capacity

monolithic,
everything on one single large die on a wafer.
one failure can ruin whole chip, expensive for large dies

chiplet design,
processor split into multiple smaller dies packaged together, connected using high-speed on-package interconnects

![[chiplet_arch.png|600]]

![[packaging.png|300]]

![[3D_2.5D.png]]

HBM4 - 2048 bit bus

wider data bus allows for large data transfers at lower clock speeds, reducing power consumption while maintaining performance.

 
dram die -> channels -> bank groups

![[hmc.png|500]]

HMC,
- not connected to xPU via interposer with TSVs
- instead uses high-speed serial links, AXI (Advanced Xtensible Interface)
- interface is packet-based, not traditional memory-bus, accessed like a network device (ip packets)

HBM optimizes bandwidth per watt using proximity
HMC optimizes scalability using networking-style links

in hbm, we can use all channels to get 1024bits 16x64 if the workload has enough memory-level parallelism, memory controller interleaves addresses across channels.

pitch, center-to-center distance between adjacent TSVs
use power TSV to reduce power drop as we go up the stack.

chip to wafer hybrid bonding (C2W HB) can stack more dies in a limit z-height by removing gap-fill material and effectively control heat dissipation.


HBM uses large no. of pins, but they operate at much lower frequency and are very close to xpu.
where as HMC uses small no. of pins but at higher frequency.
because using large no. of pins at high frequency causes crosstalk, synchronization issues.

all memory traffic fully resides in SiP. using advanced packaging technologies like silicon interposers enables data bus to be much wider and at a much lower per pin data rate. this consumes less power.

within each channel, memory banks are further segmented into two half-banks, each utilizing 64 I/Os.
in legacy, both halves of a bank output 64bits, yielding a total of 128 bits per access.
pseudo-channel mode further divides the channel into two semi-independent sub-channels, each with 64 i/os.


#### DRAM Timing
power on -> reset -> initialize -> idle

Power on is exactly what it sounds like, voltage rails are ramping up to their target levels, Vdd and Vpp reaching stable operating voltage, and during this phase the DRAM isn't ready to respond to anything yet, it's just electrically stabilizing.

Reset comes next, and this is where the DRAM holds itself, or is held externally via a reset pin, in a known safe state while those voltages fully settle and internal clocks and circuits get into a defined starting condition, this prevents any garbage commands or undefined behavior while things are still unstable.

Initialize is the actual setup phase, this is where the memory controller sends a whole sequence of specific commands to configure the DRAM, things like loading the mode registers, which set parameters like burst length, CAS latency, and drive strength, it typically also involves things like a ZQ calibration command to properly calibrate output driver impedance, and issuing some initial refresh commands to make sure all the cells actually have valid, refreshed data before real use begins.

Idle is the resting operational state once initialization is complete, the DRAM is now fully configured and ready, sitting there waiting for the controller to issue real commands like activate, read, write, or precharge, it's not doing anything active, just powered, configured, and listening.

The *Mode Register (MR)* is a small set of configuration registers inside a DRAM chip (SDRAM, DDR, etc.) that controls how the DRAM behaves internally. it's a control register that the memory controller writes to during initialization (and sometimes during operation) to configure things.

These are programmed using a special command called **MRS (Mode Register Set)**, issued by the memory controller during the boot/init sequence — the address/data bus lines are repurposed to carry configuration bits rather than actual memory addresses.

**CAS**(CL) stands for **Column Address Strobe**. CAS Latency is the delay — measured in clock cycles — between:
1. The memory controller sending a **READ command** (along with the column address) to the DRAM, and
2. The DRAM actually placing the requested data onto the data bus.

For frequency changes specifically, LPDDR uses something called Frequency Set Points, FSPs. The mode registers actually hold multiple sets of timing parameters simultaneously, essentially pre configured profiles, so instead of reprogramming registers on the fly, the controller can pre load a second FSP with the new frequency's timings while still operating at the current frequency, then switches over to that FSP with a lightweight mode register command, and this switching happens without the DRAM ever losing training or communication with the controller.

For voltage changes, it's handled through dedicated power switching circuitry rather than a reset cycle, the memory device has a power switch that handles switching between multiple drive voltage levels, controlled by commands stored in the mode register, so voltage transitions are managed as a controlled switch between two established rails rather than a power cycle. In practice frequency and voltage changes are often coordinated together for the DVFS transition, but the DRAM stays powered and in a live, trained state throughout, just briefly quiesced or put in a safe hold, not reset from scratch.

### Frequency Set Points (FSP) in LPDDR

LPDDR (LPDDR4/4X and LPDDR5/5X especially) supports **multiple frequency set points** — essentially pre-configured "profiles" of timing and operating parameters that let the memory controller switch the DRAM's operating frequency quickly, without re-running full initialization or training each time.

#### Why they exist

Mobile/embedded systems constantly scale DRAM frequency up and down for power management — dropping to a low frequency when the screen is idle, jumping to high frequency during a burst workload like camera capture or gaming. Fully retraining timing parameters (CL, write latency, ZQ calibration, etc.) every time you change frequency would be far too slow and power-hungry for this use case. Frequency Set Points solve this by letting the controller pre-load multiple operating configurations into the DRAM's mode registers ahead of time, then switch between them almost instantly.

#### How it works

- LPDDR4/4X typically defines **two Frequency Set Points**: **FSP-OP[0]** and **FSP-OP[1]** (operating points), plus a separate concept called **FSP-WR** (write) used during the mode register write process itself.
- Each set point has its own associated **CAS latency, write latency, and other MR-configured timing values** stored in a duplicated set of mode registers.
- The memory controller pre-programs _both_ set points during initialization — one typically for a lower frequency, one for a higher frequency (or more, depending on generation).
- Switching frequency then just involves toggling which set point is "active" (via a control signal / mode register field) rather than reprogramming everything from scratch, cutting switching latency dramatically.
- LPDDR5 extends this further with more flexible frequency scaling, including features like **Dynamic Frequency and Voltage Scaling (DFVS)**, letting the system change frequency and voltage together for finer-grained power/performance tuning.

#### Contrast with DDR (desktop/server) DRAM

Standard DDR3/4/5 generally runs at one fixed frequency for the session (no need for on-the-fly mobile-style power scaling), so it doesn't need this dual-profile mechanism — mode registers are programmed once at boot. LPDDR's frequency set points exist specifically because mobile power management demands frequent, fast frequency transitions.

Neither — **HBM (High Bandwidth Memory) is its own distinct JEDEC standard**, separate from both DDR and LPDDR. It doesn't reuse either spec, though it shares some underlying DNA with DDR-style DRAM cell design.

### Why HBM is different

| Aspect                  | DDR (desktop/server)          | LPDDR (mobile)                       | HBM                                                                                              |
| ----------------------- | ----------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Interface width         | 64-bit per channel            | 16/32-bit per channel                | **1024-bit per stack** (via interposer)                                                          |
| Physical form           | Discrete DIMM/chip on PCB     | Discrete chip, soldered              | **Stacked dies** on top of each other                                                            |
| Connection to processor | PCB traces, DIMM slots        | PCB traces, short routing            | **Through-Silicon Vias (TSVs)** + silicon interposer, sits right next to the GPU/accelerator die |
| Clock speed per pin     | High (GHz range)              | Moderate                             | **Lower per-pin speed**, but massive parallelism                                                 |
| Bandwidth strategy      | Speed up the interface        | Balance speed vs. power              | **Go extremely wide instead of extremely fast**                                                  |
| Typical use             | PCs, servers, general compute | Phones, tablets, laptops (low power) | GPUs, AI accelerators, HPC (e.g., NVIDIA/AMD GPUs, some CPUs)                                    |

### The key architectural difference

- **DDR/LPDDR** scale bandwidth mainly by increasing **clock frequency** per relatively narrow (16–64 bit) interface.
- **HBM** instead uses an **extremely wide bus** (1024 bits per stack, vs. 32/64 bits) running at comparatively modest per-pin frequency, achieved by **stacking multiple DRAM dies vertically** and connecting them with **TSVs** running through the silicon, all mounted on a silicon interposer immediately next to the compute die. This is a fundamentally different packaging/interconnect approach, not just a spec variant.

### What HBM _does_ share with DDR/LPDDR

- The underlying DRAM cell architecture (capacitor + access transistor), refresh requirements, and general command structure (activate/read/write/precharge, CAS latency concepts, mode registers) are conceptually similar since it's still DRAM at heart.
- HBM does have its own **mode registers** and its own **CAS latency** parameters — the concepts you asked about earlier still apply, just within HBM's own command/timing spec (defined in JEDEC JESD235, now on HBM3/HBM3E).

Modern DRAMs support two kinds of refresh modes: external refresh and self refresh. An external
refresh means that the memory controller sends the REF command to the device, and then it enters the Refreshing state. However, if the CPU is powered down, the DRAM can still maintain its values by using the self refresh mechanism. In this case, it has a built-in timer that generates two commands SRE (self refresh enter) and SRX (self refresh exit). These are used to enter and exit the Self refreshing state, respectively. If there is no activity, we can save power by entering the Power Down state.

Power down specifically means the clock is gated internally and the DRAM stops responding to most commands, but the data in the array is still preserved and refresh can still happen if needed, so it's a relatively light, fast to exit state, good for short idle gaps between bursts of activity.

Self refresh is a deeper low power mode, here the DRAM takes over refresh entirely on its own internal timer, without needing the controller to issue refresh commands at all, this saves more power since the external clock can often be fully turned off too, but it takes longer to exit and resynchronize before you can access data again.

Beyond power down and self refresh, there are a few more low power states, especially in LPDDR. Deep power down is the most aggressive one, it actually loses the data in the array entirely, so anything stored has to be reloaded from elsewhere after waking up, it's basically as low power as you can get short of turning the chip fully off. Partial array self refresh, PASR, is a clever middle ground where only specific banks or segments of the memory array get refreshed during self refresh, while unused regions are simply left unrefreshed and their data is considered invalid, this saves refresh power when an application only actually needs a portion of memory active. There's also what's sometimes called clock stop or deep sleep mode in some LPDDR generations, where the DRAM enters an even lower power idle state than standard power down but doesn't go as deep as full self refresh.

As for training, that's actually a really important concept I should unpack properly. At the extremely high speeds DDR and LPDDR operate at now, tiny variations in trace length, temperature, and voltage cause small timing skews between signals, especially between the data lines, DQ, and their corresponding strobe signal, DQS, that's supposed to tell the receiver exactly when to sample each bit. If that timing is off even slightly, you get bit errors. Training is a calibration process where, usually right after power up or after certain low power transitions, the memory controller and DRAM go through a structured sequence, sending known test patterns back and forth, and adjusting internal delay elements until the data and strobe signals are precisely aligned in both timing and voltage. This is why some deep low power states, like full self refresh exit or definitely deep power down, require expensive retraining before you can trust the data again, but lighter states like power down don't lose that trained alignment at all, which is exactly why they exit so much faster.

"Phase difference is the timing misalignment between the external clock and the clock actually driving internal circuits, caused by propagation delay through the chip's clock distribution network. A DLL corrects this by passing the external clock through a tunable delay line, using a phase detector in a feedback loop to find the delay setting that makes the internally-distributed clock re-align with the external reference — so output data (DQ) and the strobe (DQS) launch synchronized to the true external clock edge, which source-synchronous DDR reads depend on."


In power down mode, the DRAM does not manage refresh internally, the controller is still fully responsible for that. Since power down typically only lasts for short idle gaps, the controller has to actually exit power down periodically, issue a refresh command like it normally would during active operation, and then can go back into power down. There's actually a spec restriction on how long you can stay in power down continuously without violating the refresh window requirement, so it's inherently meant for brief pauses, not long idle periods.

Self refresh flips that responsibility entirely, the DRAM has its own internal oscillator or timer and autonomously issues its own internal refresh cycles without the controller doing anything, that's precisely why self refresh is meant for longer idle stretches, the controller can essentially walk away and even shut off its own clock output, trusting the DRAM to keep its own data alive on its own.

When the DRAM is idle and a bank is in the precharged state, yes, the bit lines are sitting at Vdd over 2, held there by the precharge circuitry, this is the standby, ready state, not something that happens continuously on its own, it's actually the result of the last precharge operation having been done.

Then you send ACT, activate, along with a row address, this does two things at once, it opens the access transistors for that entire row via the word line, and it lets the sense amplifiers do their job, sensing the tiny charge shift and then actively driving that whole row's bit lines fully to Vdd or ground based on what each cell held, that's when the row buffer becomes valid and ready.

Once activated, you can then send READ or WRITE commands, and importantly, you can actually issue multiple reads or writes to different columns within that same open row without needing another activate, that's the whole benefit of row buffer locality, it's already sitting there sensed and amplified.

When you want to access a different row, either in the same bank or you're just done with this one, you send PRECHARGE, this closes the row, meaning the sense amplifiers disconnect from the array and the bit lines get driven back to that Vdd over 2 baseline again, resetting the bank so a future ACT can open any row cleanly. So the order is precharged idle, then ACT opens and senses a row, then READ or WRITE happens against that open row, potentially several times, then PRECHARGE closes it back down before the next ACT.


Activate Command
The activate command ACT is used to activate a given row in a bank, read all the columns into the sense amplifiers, and wait for a subsequent read or write access. Note that each row (or page) can be fairly large: 1024 or 2048 bits. Whenever we activate a row, all of this data is brought into the sense amplifiers, and subsequently the column multiplexers choose a subset of the bits. This is thus an expensive operation.

Precharge Command
The precharge commands, PRE and PREA, are used to precharge the bit lines in a given bank, or in all the banks respectively. The row is subsequently deactivated, and the bank enters the Idle state. For a subsequent access, we need to activate a row first.

![[ACT_timing.png|500]]

![[PRE_timing.png|500]]

because a row activation is an extremely power-hungry operation. We need to read an entire row of
512-2048 cells, and store their contents in the sense amplifiers. Furthermore, since reads are destructive, the data needs to be restored. Consequently, we need to have a minimum delay between row activations such that we can limit the power consumption.
The first is the row-to-row delay (tRRD). This is the minimum time interval between activating a given row and activating another row. There are two types of row-to-row delays: tRRDS (short) and tRRDL(long). tRRDS is the minimum delay when the rows are in different bank groups. Whereas, tRRDL is the minimum delay when the rows belong to different banks in the same bank group. tRRDL ≥ tRRDS because we wish to enforce a power constraint for each bank group. Here same means same as the previous access, and the term different is defined likewise. This definition holds for the rest of the commands that use the same nomenclature. This means that we are discouraging consecutive row activations in the same bank group, which needs to be done to limit power consumption and local temperature rise.

Another parameter that limits the device-wide power consumption is tFAW (Four-bank Activation
Window). This means that in any sliding window of time that is tFAW cycles wide, we can have at most four row activations. For example, if this window is 50 cycles wide, then there is no 50-cycle window of time in which there are more than four row activations. This limits the overall power consumption of the DRAM device.

To initiate a read, the memory controller sends the READ command along with the number of the
column (on the address bus), and the address of the bank group. Subsequently, the device starts to read the values by issuing the internal read command IntREAD.

![[Read_PRE_POST.png|300]]

**How temperature and voltage affect propagation delay**

Every signal traveling down a wire and through transistors has to charge and discharge capacitance along the way, that's fundamentally what causes delay, not the wire being "slow" itself but the RC nature of the path, resistance and capacitance combining to slow down how fast a voltage transition can happen.

Temperature affects this because as temperature increases, electron and hole mobility inside the silicon decreases, meaning charge carriers move a bit more sluggishly through the transistor channels, so transistors take slightly longer to fully switch on or off, and resistance in metal interconnects also increases with heat, adding more delay along the wire itself. So hotter chips generally run slower per transistor switch, a real, measurable effect at gigahertz timescales.

Voltage affects it more directly and dramatically. Transistor switching speed depends heavily on how much drive current it has, and drive current is roughly proportional to how far above the transistor's threshold voltage your supply voltage sits. So if supply voltage sags even slightly, due to noise, IR drop from many transistors switching at once, or an imperfect regulator, drive current drops, and the transistor takes measurably longer to fully switch, so every buffer and gate connected to that rail delays a little more. Higher voltage does the opposite, faster switching, less delay.

**How DQS works and why it drifts together with data**

DQS, data strobe, is essentially a clock like signal that travels down its own dedicated wire, physically routed right alongside the data lines it's paired with, DQ, rather than being generated once, centrally, and distributed everywhere.

The key idea is source synchronous timing, the same driver on the DRAM chip that's sending out the data bits on DQ also sends out the DQS strobe from that same location, at that same moment, through wires that are matched in length and routed through very similar physical conditions on the board and package.

Because DQS and DQ physically travel side by side through nearly identical trace lengths, similar layers of the PCB, similar proximity to heat sources, similar coupling to nearby signals, whatever temperature or voltage induced delay affects one of them tends to affect the other by almost the same amount, they experience what's called common mode drift. So even if the absolute delay from chip to controller varies a lot depending on conditions that day, the relative timing between DQS and DQ, which is the only thing that actually matters for correctly capturing each data bit, stays much more stable, since both signals are drifting together in the same direction by roughly the same amount, rather than one drifting while a separate, centrally distributed clock stays fixed and increasingly out of sync.

That's really the elegant trick, instead of fighting drift by trying to eliminate it, *source synchronous design* just makes sure the reference signal experiences the same drift as the data it's measuring, so the receiver samples data relative to DQS's edges rather than relative to some absolute, drift prone global clock.

Good question, and this connects directly to that source synchronous idea.

Here's the subtlety, DQS is great for keeping data and strobe in sync while they're actively toggling together, but DQS is not toggling constantly, it only becomes active right when a burst of data is actually being transferred. The rest of the time, when there's no read or write happening, the DQS line is sitting idle, and idle lines in high speed digital systems tend to float, or sit at some undefined or high impedance state, they don't have a clean, known logic level.

So the problem is, the very first edge of DQS coming out of that idle floating state can't be trusted, the receiver's circuitry needs the line to be actively driven to a stable, known level for some duration before it can reliably detect the first real, meaningful clock edge that tells it "the data burst starts now." That's exactly what the read preamble is, a short period right before the actual data burst where DQS is driven to a defined low or stable level, essentially warming up and stabilizing the line, giving the receiver's clock recovery circuitry a clean, predictable reference to lock onto before the real toggling begins.

The postamble does the same job in reverse, at the end of the burst, DQS needs to be held in a defined state for a short period after the last real data driven edge, rather than instantly snapping back to floating or high impedance, this prevents the receiver from misinterpreting a floating, undefined transition right at the tail end as if it were one more legitimate data edge, which could corrupt the very last bit or two of the burst.

So essentially, preamble and postamble exist because DQS isn't a continuously running clock, it's a burst mode signal, and any burst mode signal needs defined transition boundaries so the receiver knows unambiguously where real, trustworthy toggling begins and ends, versus where the line is just sitting in an unreliable idle condition.

![[pre_post_timing_diagram.png]]

DQS is neither simply "active high" nor "active low."
It is a timing/strobe signal, not a conventional enable signal.
receiver knows how many data beats to receive.

**The DQS-per-byte convention**

In standard DDR (DDR2/3/4/5), the data bus is divided into **byte lanes** — a group of 8 DQ pins gets exactly **one DQS pair** (DQS and its complement, DQS#, since it's driven differentially in DDR3/4/5). This 8-bit grouping isn't arbitrary — it directly maps onto how DRAM chips are commonly organized: a x8 chip has 8 DQ pins and outputs one DQS pair; a x16 chip effectively has two independent byte lanes internally, each with its own DQS pair (so a x16 chip has 2 DQS pairs total, covering 8 bits each).

Good catch — let me clarify, because "pair" here refers to something different from "two DQS per byte lane." There's still only **one DQS per byte** (8 data bits) — but that one DQS signal is transmitted as a **differential pair**, meaning it uses **two physical wires** to carry that single logical strobe signal.

**What "differential pair" means**

Instead of sending DQS as a single wire referenced to ground (single-ended signaling), DDR3/DDR4/DDR5 send it as **two complementary signals**: DQS and DQS# (also written DQS_c and DQS_t, or DQS/DQS̄). These two wires always carry **opposite** voltages relative to each other:

- When DQS is high, DQS# is low
- When DQS is low, DQS# is high

The receiver doesn't look at either wire's absolute voltage against ground — it looks at the **difference** between the two wires (DQS minus DQS#) to determine the logic level and, critically, the precise clock edge timing.

**Why differential signaling is used for DQS specifically**

- **Noise immunity**: Any electrical noise picked up along the trace (crosstalk, power supply noise, EMI) tends to affect both wires of a tightly-routed differential pair almost identically (common-mode noise). Since the receiver only cares about the _difference_ between the two signals, that common noise cancels out — it doesn't corrupt the differential signal.
- **Precise edge timing**: DQS's entire job is to mark exact sampling edges for the data. A differential pair crosses over (one line rising through the other falling) right at the switching point, giving a cleaner, more precise, more noise-resistant edge-detection point than a single-ended signal crossing some fixed voltage threshold — this matters enormously at the very high frequencies DDR operates at, where even small timing jitter eats into already-thin margins.

"Data isn't latched based on DQS being 'high' or 'low' as a level — it's latched at the precise crossing point where DQS and DQS# swap places, and DDR uses both the rising and falling crossings to get two transfers per cycle. DQS/DQS# aren't ambiguous or 'roughly equal' during an active burst; the only time they're not meaningfully differential is during the defined preamble/postamble around a burst, or when idle and tri-stated between bursts entirely."

"Idle describes the functional state of not transferring data; tri-state is the electrical mechanism that achieves it — the output driver disconnects from both Vdd and ground, leaving the pin floating (high-impedance) rather than actively driven. This is essential for bidirectional, shared pins like DQ/DQS, since both the controller and DRAM chip take turns driving the same wires, and both must tri-state when it's not their turn to avoid bus contention."

A write operation is slightly slower than a read because after every write we need to do some more work. tWTR cycles are needed to ensure that we are able to write the correct state to the sense amplifiers and the DRAM cells. This requires time because some bit lines need to undergo voltage transitions.

----

In a standard 1T1C DRAM cell, the convention is:

- **Bit line → connected to the drain** of the access transistor
- **Storage capacitor → connected to the source** of the access transistor
- **Word line → connected to the gate**, which turns the transistor on/off to allow access

So the access transistor acts as a switch: when the word line is asserted (goes high), the transistor turns on and connects the bit line (drain) to the capacitor (source), allowing charge to flow in either direction — into the capacitor during a write, or out of the capacitor onto the bit line during a read (which is why DRAM reads are destructive and need to be followed by a write-back/refresh).

One nuance worth knowing for an interview: in an NMOS transistor, "source" and "drain" aren't fixed by geometry — they're defined by which terminal is at the lower potential during operation (current flows from drain to source). So strictly speaking, during a write of "1," current flows from the bit line into the capacitor, making the bit-line side the drain and the capacitor side the source, consistent with the naming convention above. During certain read conditions the actual voltage relationship could technically reverse which terminal is "acting" as source vs drain, but the industry-standard naming — bit line = drain terminal, storage node = source terminal — is fixed by convention regardless of the direction of current flow in a given operation. If asked in an interview, giving the conventional answer (bit line = drain) is what's expected, with that caveat as a bonus if you want to show deeper understanding. 

**One-line summary for the interview:** "Source and drain in the DRAM access transistor aren't fixed by wiring — they're determined dynamically by which terminal has the lower voltage. During writes, whichever side you're driving low is the source. During reads, the bit line is precharged to Vdd/2 and the direction of charge sharing — and thus which terminal briefly acts as source — depends on whether the cell holds a 1 or a 0."

unlike an SRAM cell, a capacitor cannot maintain a steady voltage for a long period of time.
Due to some current leakage between the parallel plates, ultimately all the stored charge will leak out. Even if the leakage current is very small, these capacitors will ultimately lose their stored charge. It is thus necessary to reduce the leakage current to as small a value as possible. The standard technique to handle this situation is that we periodically read the value of a DRAM cell and write it back. This ensures that even if the potential has dropped due to a leakage of charge, the voltage across the capacitor can be restored to the ideal level. This process is known as a refresh, and DRAM cells require periodic refresh operations to ensure that we do not lose any data.

#capacitors

![[trench_capacitor.png | 400]]

![[trench_capacitor_explanation.png]]

![[stacked_capacitor.png]]

A stacked capacitor is still much better than a regular planar capacitor that is made on silicon
because it is a 3D structure. The capacitor can be fabricated above one of the terminals of the access transistor, and thus we can increase density.


Recall that in an array of cells, the address is first sent to a row decoder, which enables one of the
word lines in the 2D array of DRAM cells. This enables all the cells in a given row, which is also called a page. They can then be accessed by the bit lines. Note that in this simplistic design we have a single bit line per memory cell. Each bit line is connected to a sense amplifier that senses the voltage and converts it to a logical 0 or 1. Unlike SRAM arrays, in a DRAM array we can only read a single column in one cycle. We have a column multiplexer/demultiplexer (mux/demux) that chooses the right column to read or write. It is controlled by a column decoder that takes as input a set of bits from the address.
The column mux/demux is connected to read and write buffers that buffer the values that are read or need to be written. The value that is read then needs to be sent on the CPU-memory bus.
An important point to note here is that in DRAM arrays the sense amplifiers appear between the
bit lines and the column mux/demux. This was not the case in SRAM arrays. In SRAM arrays the bit lines were directly connected to the muxes/demuxes, and this structure was then connected to the sense amplifiers.

An SRAM cell (typically 6T) is a **bistable latch** — two cross-coupled inverters that actively hold their state as long as power is applied. Because of this:

- **The cell drives the bit lines itself, continuously.** There's no charge-sharing, no need to "sense" a tiny voltage perturbation the way DRAM does. The full-strength inverters directly pull the bit line pair to valid logic levels.
- **Reading is non-destructive.** Since the cell isn't relying on a capacitor's finite charge, reading it doesn't disturb the stored value. No restore step is needed afterward.
- **There's no activate/precharge cycle at all.** DRAM needs the activate step because the row's data doesn't exist as a stable, driven signal until the sense amps latch it. SRAM cells are _always_ actively holding and driving their bit lines — there's nothing to "open" or "restore."

Contrast with DRAM, where multiple columns can be read from an open row too (that's the row-buffer/page-hit mechanism), but only _after_ the relatively slow activate-and-sense step completes and only while that row stays open. If you switch rows, DRAM pays the precharge/activate penalty again. SRAM has no equivalent bottleneck — every row is effectively "always open," so essentially the entire array is available for immediate parallel column access at any time.

![[array_of_dram_cells.png|600]]

consider the case where the capacitor in the DRAM cell stores a logical 1. When we enable the
access transistor via the word line, the capacitor starts to charge the bit line. This means that stored charge from the capacitor flows towards the bit line and increases its voltage. This further means that the voltage across the capacitor in the DRAM decreases. The next time that we read this cell, the voltage across it might not be enough to infer a logical 1. This means that the DRAM cell will lose its value, which is not desirable. This phenomenon is known as a destructive read. The only way to avoid this situation is to ensure that we rewrite the value after it is read. This is known as restoring the value that has been read. This is essential in a DRAM and adds to the latency of a read operation.

DRAM sense amplifiers are special in the sense that they function both as differential amplifiers as well as buffers.
This data is sent to powerful driver circuits that send the data over the bus to the CPU.

A sense amplifier has two inputs. It compares the voltage difference between them and senses the direction in which the voltage difference is progressing. If the voltage difference exceeds a positive threshold, then we can infer a logical 1, and likewise if it decreases beyond a negative threshold, then we infer a logical 0.

However, in this case, we have a single bit line, and thus we have a single input. It is thus
necessary to add an additional input.
In this design, we split the entire array by dividing each bit line into multiple segments as shown in
Figure 10.5. We then connect the bit lines for segments i and i + 1 to the same sense amplifier.

**The setup: why segment the bit lines at all?**

A DRAM bit line runs vertically through an entire column of cells. Every single cell's access transistor sitting on that bit line adds a tiny parasitic capacitance to it (drain junction capacitance, wire capacitance along its length, etc.). If you let one bit line run the _entire height_ of the array — say, connected to 1024 or 2048 rows — it accumulates a lot of parasitic capacitance.

That matters because of charge sharing: when a cell's tiny storage capacitor connects to that long bit line, the resulting voltage perturbation is roughly:

ΔV ≈ Vcell × (C_cell / (C_cell + C_bitline))

A bigger C_bitline means a smaller, harder-to-sense signal, which means slower, less reliable sensing. So the design **splits the array into shorter segments** (fewer rows per segment), each with its own shorter bit line, so each bit line only picks up capacitance from a limited number of cells — keeping that signal big enough to sense quickly and reliably. That's the "capacitive loading and consequent latency" line.

**The problem this creates: sense amps need two inputs**

A sense amplifier is a _differential_ circuit — the cross-coupled inverter pair we discussed earlier needs two complementary inputs (BL and BL̄) to compare against each other and amplify the difference. If you now have many short, separate bit-line segments, naively you'd need a dedicated pair of bit lines _per segment_, each needing its own sense amp with its own complementary reference line — extra wiring, extra area.

**The clever trick this paragraph describes**

Instead of adding a dedicated complementary bit line for every segment, they pair up **two physically adjacent segments** and share one sense amplifier between them: segment i's bit line becomes one input (say BL), and segment i+1's bit line becomes the _other_ input (BL̄) to that same sense amp.

PRE is bank level.

Yes, that's exactly right. If a bank hasn't been activated (or has been precharged and not touched since), its bit lines just sit at Vdd/2 the whole time it's idle — that's the resting/idle state.

![[open_bitline.png|400]]

![[folded_bitline.png|300]]

**Design of a Sense Amplifier**

Sensing the voltage difference is a two-stage process. We first equalize the voltages of the two bit
lines. This is a very simple circuit that is connected to the two bit lines. When the EQ line is set to a logical 1, transistor T1 gets enabled. After this the potential difference between the two bit lines (1 and 2) becomes roughly zero. Next, we need to ensure that it is equal to the precharged voltage: Vdd/2. Look at transistors T2 and T3. After EQ is set to Vdd, transistors T2 and T3 will turn on and the bit lines will get set to the voltage Vdd/2. Once this is done, both the bit lines are said to be precharged.

![[dram_equalizer.png|400]]

Then in the second stage, we enable a row of the DRAM array and allow the bit lines to gradually
get charged or discharged. Next, we need to sense the difference in the voltages between Bit line 1 and Bit line 2 (see Figure 10.8). Note that we are deliberately avoiding the notation BL and BL over here because these are two separate bit lines that are connected to different sets of DRAM cells. the voltage on Bit line 1 (V1) is slightly higher than the voltage on Bit line 2 (V2), where V2 = Vdd/2. At this point of time let us set the voltage on SAN to 0 V and the voltage on SAP to Vdd (assume logical 1 is Vdd volts). This enables the sensing operation.
![[sens_ampli_working.png]]
helped us to store a bit as well. Finally, note that once the respective bit line gets charged or discharged, the DRAM cell can also “restore” its value. For example, if the cell stored a logical 1, the charged bit line can restore the charge of the capacitor.
![[dram_sense_amplifier.png|400]]
We thus see that the sense amplifier serves several purposes at the same time. First, it senses small
changes in the voltages of the bit lines and amplifies the difference such that the bit lines quickly get fully charged or discharged. Once, the voltages of the bit lines have been set, they will remain that way and keep restoring the value of the DRAM cell till we disable the word line. For accessing a new row, we need to activate the equalizer circuit once again and set the voltages of both the bit lines back to the precharge voltage: Vdd/2. To disable the sense amplifier at this point, we can set the voltages of SAN and SAP to Vdd/2; this will ensure that all the four transistors are in the cut-off state.

The sense amplifier and the precharge circuit are connected to powerful write drivers via access
transistors as shown in Figure 10.9. The access transistors are controlled by a chip select line (CS),
which effectively enables the DRAM chip.

![[dram_circuit.png|500]]

![[write_recovery.png]]

a refresh can be thought of as a dummy read operation. Note that we do not need to use the
column decoder or enable the chip select line.

Typically, the memory controller issues a command to a given rank. All the DRAM chips that are a part of the rank work in lock-step to execute the command. The main advantage of grouping DRAM chips together is to provide a high bandwidth memory. For example, if we need to supply 64 bits every cycle, then it makes sense to create a rank of 16 chips, where each chip supplies 4 bits. This keeps each individual DRAM device small and power efficient.

Subsequently, each rank has multiple banks (grouped into bank groups in the DDR4 protocol). A
bank is a set of arrays within a DRAM chip that operates independently with respect to other banks on the same chip. A bank typically contains multiple arrays that cannot be independently addressed. The arrays within each bank work in synchrony. For example, if we have 4 arrays in a bank, we access the same row and column in each array while performing a bank access. We read 4 bits in parallel.
This is conceptually the same as assuming that we have one large array where each cell or each column is 4 bits wide. Using more arrays increases the bandwidth of a DRAM device because we can read more bits in parallel. In a DRAM chip all the banks have the same number of arrays.
This is typically specified as follows. When we say that we have a x4 DRAM, this means that we
have 4 arrays in a bank, and we read and write 4 bits at a time. An xN DRAM has exactly N arrays in a bank.

Each array is a matrix of DRAM cells. We first access the row and then the column to read or write a single bit. The entire hierarchy of structures in a DRAM is as follows: channel → DIMM → rank → chip or device → bank → array → row → column.

the chip select signal is used to enable a specific rank of devices.
If we have 4 ranks in a DIMM, then we need a 2-bit chip select signal to select the specific rank.

We can either use separate address and command buses, or have a single bus to carry the information for both memory addresses and commands. It is possible to fuse them because we typically send the address and commands at different points of time.

DRAM memories are passive memories. There are no computing elements embedded inside the memory. As a result, the entire job of scheduling all the accesses, ensuring that there are no conflicts, and providing fairness is left to the memory controller.
The memory controller needs to be aware of the timing details of the memory devices, and then it needs to schedule all the requests that it gets from the cores and the cache banks.

"The memory controller doesn't measure DRAM timing dynamically — it reads the SPD EEPROM on the DIMM at boot to learn the chip's rated timing parameters (tCL, tRCD, tRP, etc.) for various speed bins, then the BIOS selects an operating speed and programs both the controller's timing registers and the DRAM's own mode registers via MRS commands so both sides agree on the same timing contract. On top of that, the system runs training routines at boot to fine-tune signal-level timing margins for the specific board and DRAM part, since real electrical behavior varies even among parts that meet the same JEDEC spec. For soldered memory with no SPD, the timing values are simply hard-coded into firmware at design time."



The receiver monitors the strobe, and whenever there is a transition in its voltage level,
it reads the data bus (DQ signal).

This is done by sending the row address on the address bus, and then after some time asserting
the RAS signal (setting it to 0). The reason that we do this is as follows. We want the data on the
address bus to be stable before the device starts reading it.

In the FPM protocol, we send the column address, then we wait for the data to be read, and then we send the next column address, and so on. This process can be optimized further. We can send a column address, internally buffer the data that has been read, and while that data is being transferred, send the address of the next column. This will however require another strobe signal (DQS) as shown in the figure. This strobe signal indicates to the memory controller that the data is available on the data bus.

after one column address is sent, we can generate the next k column addresses internally, read those columns, and send their data over the data bus. We do not have to waste time in sending column addresses separately. our prefetch length is 2 (read two columns after sending a single column address). We do not need a separate strobe signal, we can use the CAS signal to provide the timing.

double data rate memory (DDR memory), the data bus runs at twice the speed of the address and command buses.

Additionally, we have a new signal, a data strobe signal DQS, that is sent along with the data. This is because we can have minor clock skews, and for the receiver to read the data correctly, it needs to use the DQS signal for clock synchronization. This method of transmission is known as source synchronous transmission.
this signal can take three values: logical 0, logical 1, and an intermediate voltage that represents the undefined state. Furthermore, its voltage can be set by either the memory controller or the DRAM device depending upon the direction of data transmission.

To switch the rank it takes one cycle because the new rank has to re-synchronize its clock with the data strobe.

This is because while reading data, the DRAM device is the sender. It is much easier for it to transmit data at clock boundaries.

The memory controller can then recover the clock from the strobe and latch the data at the correct time instants. This requires more circuitry but is an acceptable overhead at the side of the memory controller.

Note that the data bus in this case runs in half-duplex mode, Which means that it does not allow
simultaneous transmission of data in both directions. Thu,s DDR memory is the most efficient when we either have only reads, or only writes. To remedy this problem, QDR (quad data rate) memory was proposed that has two data buses: one read bus and one write bus. This allows us to effectively achieve a higher bandwidth by interleaving reads and writes.

"Modern mainstream DRAM — DDR4, DDR5, LPDDR5, and HBM — is DDR, transferring on both clock edges. QDR is a separate, niche SRAM technology used in networking, not something found in commodity system memory. High-end graphics memory (GDDR6X/GDDR7) gets extra effective bandwidth per pin not through QDR, but through PAM multi-level signaling, which is a conceptually different trick from simply doubling the clock-edge count."

the transfer rate of the DRAM device measured in millions of transfers per second (MT/s). A transfer is defined as the transfer of a 64-bit data packet (equal to the channel width) from the memory controller to DRAM or vice versa.

if the bus frequency is 400 MHz, then we perform 800 million transfers per second (MT/s) because of double data rate transmission. The standard nomenclature that we use to label DRAM devices is of the form (⟨Technology⟩ − ⟨Transfer rate⟩). For example, a DDR3-1600 technology means that we are using the DDR3 technology, and we perform 1600 million transfers per second.

As we connect more DIMMs to the channel, the capacitive loading on the
address, command and data buses increases, which adversely impacts their RC delay. Recall that the RC delay or the time constant is the time that it takes to charge the bus to 63% percent of its final value. This means that with more connected devices, it takes more time to effect voltage transitions on the bus, and this directly places limits on the bus frequency.

- Instead of a narrow serial bus solving a wide shared-bus loading problem (FB-DIMM's approach), HBM does the opposite: it uses an **extremely wide parallel interface** (1,024 or 2,048 bits) over very short traces through the interposer, because the whole point is to avoid needing high per-pin frequency (and thus avoid the very signal-integrity/loading problems FB-DIMM was designed around) by just adding more parallel wires instead.
- There's no buffer chip analogous to AMB in the data path at all — the memory controller (in the SoC/GPU) talks essentially directly to each HBM stack's PHY





