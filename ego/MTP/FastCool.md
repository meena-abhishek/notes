#edp
Energy-delay product is a metric that combines how much energy a computation uses and how long it takes: $EDP=E×D$. A lower EDP means the system is using less energy while also finishing sooner, or at least not becoming much slower to save energy.
Delay is squared in $\text{EDP}$ because a slower design is often less desirable than a slightly more energy-efficient one. Squaring delay penalizes extra time more strongly, so the metric favors designs that save energy without making the computation much slower.


incorporating leakage power and thermal-leakage loop in a DTM strategy is foundation of this paper.

$\text{refresh overhead}= \frac{(\text{Time required to refresh a row}) × (\text{Number of rows})}{(Refresh Interval)}$

So the overall idea is: how much of the available refresh interval is consumed by refreshing all rows.

data migration delay = total data / bandwidth
available bandwidth = bandwidth x no. of channels x (1-refresh overhea)

*insights*
- instead of a predefined order close channels based on temperature, leakage, access rate
- migrating frequently accessed channel may lead to increase 2d memory power, load on 2d and application slowdown.
- closed adjacent channel provides cooling.

$EEFC$

pass1
- close high T, high L, low A. skip if neighbour is turned off
pass2
- close high T, low A.
pass3
- close low A

uses a remap table to keep track of channel mappings from 3d to 2d and track which channels are migrated.

Overall, due to channels being turned OFF in the proposed approach, the 3D memory gets cooled down much faster than the Baseline approach, which needs to stall longer and frequently to achieve cooling.

run benchmark on sniper performance simulator to get memory access trace, use energy-per-access value obtained from cacti-3dd to convert this access trace into a power trace (with 1ms epoch) and fed this to hotspot (after adding leakage power and refresh power) to generate temperature traces.

need to reduce longer and frequent cooling periods.

take temperature and access rate together into account, reduce temp only when access rate is higher than threshold, instead of throttling at 76.9, throttle at 79.4

select temperature that gives less EDP for the workload.

