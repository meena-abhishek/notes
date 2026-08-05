$F=\frac{q_1q_2}{r^2}$

whenever $+$ and $-$ charges are separated, there is a force that tries to pull them back together, and this force can be represented by an electric field.

KCL, kirchoff's current law
all current going in must equal to all the current going out.

if an electron goes all the way around the circuit, the amount of energy given to it by the battery must equal the amount of energy lost as it travels around through the other devices in the loop.

KVL,
energy of a particle must come back to its starting energy.
adding all the voltage differences  between elements in a loop must equal to zero.

ground, zero voltage reference.

power, $P=iV = energy/time$ 

series, 
every charge that leaves one has no choice but to go through the other.
parallel,
as current flows from one node to other, charges can go through either elements, meaning that the current splits and goes through both elements in parallel.

steady-state,
equilibrium a circuit reaches after a sufficient period of time.

capacitors change the way in which voltages and current transition from one state to another.

the larger the area of the plates and smaller the gap, the higher the capacitance.
$Q=CV$
$i=C\frac{dV}{dt}$

it takes current to change voltage across a capacitor, and the faster you try to change the voltage, the more current that is required.
since most circuits can supply only a fixed amount of current,
capacitors will prevent the voltage across them from changing rapidly.

$E=\frac{1}{2}CV^2$ 

DC circuit, output a constant voltage across them. this means there is no change in voltage over time. so no current flow.
capacitors are DC open circuits.

charging a capacitor,
$V_c(t) = V_s(1-e^{\frac{-t}{RC}})$

RC $\tau$ , time constant

we know all real wires have some amount of capacitance (which can be modeled with a capacitor) and some amount of resistance.
therefore, we can now calculate how long it takes to drive a wire to a voltage we desire.
![[RC.png|250]]

by modeling the circuit as an RC circuit, we can calculate the time to reach the threshold above which pin returns HIGH.

capacitor discharging,
$V_c(t) = V_{dd}e^{\frac{-t}{RC}}$

output wire of logic gate cannot change from $V_{dd}$ to $Gnd$ immediately.
all logic takes non-negligible time to switch states.



