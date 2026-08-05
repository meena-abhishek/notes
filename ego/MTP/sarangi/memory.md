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
- read access is basically read+write(restore), overhead
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