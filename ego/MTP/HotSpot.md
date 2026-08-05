designing cooling solutions for worst-case power dissipation is expensive.
chips that can autonomously modify their execution and power-dissipation characteristics permit the use of lower-cost cooling solutions while still guaranteeing safe temperature regulation.

evaluation techniques for this dynamic thermal management (DTM) 
requires a thermal model.

*HotSpot* , a model based on equivalent circuit of thermal resistances and capacitances.

Finite Element simulation, 
is a numerical method used to approximate solutions to complex physical problems by splitting a large system into small elements.

power density, 
amount of power generated, consumed, dissipated per unit area or volume.
because of small transistors, high frequency, more transistors packed in same area. power density $P/A$ is increasing rapidly and so is the cooling cost.

localized heating occurs much faster than chip-wide heating.
since power dissipation is spatially nonuniform across the chip, this leads to "hot spots", this can cause timing errors of physical damage.

many low-power techniques have little or no effect on operating temperature, because they do not reduce power density in hotspots or because they only reclaim slack and do not reduce power and temperature when no slack is present.

DTM allows the thermal package to be designed for power densities exhibited by typical applications, with the chip itself adapting if temperatures approaches dangerous levels.

carrier mobility is inversely proportional to temperature.

register files, execution units, load-store queues are generally more hot than caches.

thermal behavior is diffusive, not inductive

we can replace a thermal problem with an analogous electrical problem, and then we can use traditional circuit simulators to find the voltages and currents at different places.
these values can then be mapped to temperature and power values.

temperature -> voltage
power -> current

![[electric-thermal.png|400]]

the power density is increasing, and transistors are getting smaller and more unrealiable. in addition, because of process variations (variation in transistor dimensions caused due to imperfections in the fabrication process), the leakage power tends to vary across the chip, and thus it is possible to have some areas of chip where the leakage power is very high. because of this probability of forming temperature hotspots is also elevated.

circuits consume power when there is a transition in the voltage levels (0->1 or 1->0). upon a transition there is a flow of current to charge or discharge gate capacitances and the resulting resistive loss is known as dynamic power consumption.

power per unit area (heat flux) is proportional to the gradient of the temperature.

heat flow can be described as "current" passing through a thermal resistance, leading to temperature difference analogous to a "voltage".
