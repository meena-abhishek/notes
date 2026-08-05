low power state in DRAM can reduce leakage power upto 10x and dynamic power to 0.

DRAM allows rank level power control.

in lower power state DRAM can retain data but switching back to active mode incurs a resynchronization (wake-up) time overhead (<10$\mu s$).
but still its better than overheads caused by migrating data.

insights:
- unlike turnoff low-power-states does not incur migration delays. we can use much finer granularity of temperature.
- placing high access rate channels in LP can provide better cooling

count of lower-layer ranks to be turned off in a channel is determined at runtime using access rates.

2d memory have higher energy per access.

distributed cooling -> quicker temperature reduction

placing 8 channels in LP state ensures max. temp of a channel $T_{max}<80C$
define 8 ranges, if $T_1 < T_{max} < T_2$ place one channel in LP, if $T_2 < T_{max} < T_3$ place 2 channels $\ldots$ 

if $T_{max} > T_9$ place all 16 channels in LP.

ML-DTM,
works in two phases,

high access rate phase, 
select k channels based on high access rate and put them in LP. if can't fullfill k needs, add channels that have least stall time.

stall equalization phase,
set $M$ for $k$ least stalled channels. put all channels in active state and then put masked channels in LP.

![[ML-DTM.png|500]]

ratio of $\tau$ and $\theta$ is significant,
- decreasing $\theta$ keeping $\tau$ same, increases stall time equalization phase, which causes few channels to be closed for long times.
- increasing $\theta$ keeping $\tau$ same, decreases $\%$ of high access rate phase, which causes heating and DTM overheads.

