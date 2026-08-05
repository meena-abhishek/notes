covalent bond - electrons are shared
ionic bond - electrons are transfered

when atoms get close, some electron orbitals overlap, shared electrons are attracted to both nuclie, this lowers the total energy of the system.

silicon has silicon dioxide, which acts as a insulator
we dope silicon, cause at room temperature it doesn't have enough free electrons for conduction.

![[why_doping.png|600]]
![[law_of_mass_action.png|500]]

electric field points from higher to lower potential, $+\to -$ 

higher potential doesn't mean positive, it just means numerically greater potential.

$10V$ battery means voltage difference between positive and negative terminal is $V_+-V_-=10V$ 

electron potential energy is $U=-eV$ , higher voltage means lower energy for electron.

battery chemistry separates charges, creating a electric field in the circuit. chemical forces electrons to move from positive side to negative side.

separated charges creates electric field and hence voltage difference

![[PN_junction.png|500]]


*law of mass action* states that, at thermal equilibrium, the product of electron concentration and hole concentration in a semiconductor is contstant. $np=n_i^2$

applying a voltage means creating a electric potential difference between two points.
![[npn.png|400]]

field effect transistor

![[mos.png|400]]

metal oxide semiconductor

$V_{GB} < 0$ - accumulation of holes in suface of semi-conductor

$V_{GB} > 0$ - depletion, depleting the bulk so that it can build surface potential in order to invert the channel.
pushing holes away, in other words boron holes gets filled with electrons, immobile charge carriers increase near surface and some free electrons also gather up. electric field in created between positive gate and these negative boron atoms and free electrons.

$V_{GB} > V_{th}$ - inversion
surface becomes as n-type as the body is p-type.

$V_{th}$ - gate potential needed to invert the surface. work done by gate to invert the channel

![[inversion.png|250]]

available voltage would be $V_{gs}-V_{th}$ in order to create more inversion charge so that current can flow.

reverse-bias PN junction, depletion layer grows. if source and drain are close, channel length will go down and gate has to do low work to invert the channel, low threshold voltage.
deplition region (extra electrons) goes far into p substrate. 
because we are applying +ve voltage to n-type drain.

![[short_channel_effect.png|500]]

when transistor is off, diffusion current dominates, when its on, drift current dominates
subthreshold leakage,
transistors behave likes BJT, cause the drain and source are so close to each other that the channel is so thin that it behaves like a base, so some current diffuses between drain and source even though gate voltage is below threshold voltage.

BJT action, small current at base allows much larger current between emitter and collector.
injection of electron from thin base.
we can't indefinetly reduce $V_{gs}$ , we get gate induced drain leakage after some point.

reducing threshold voltage will increase the subthreshold leakage exponentially.
we reduce threshold voltage to have larger current flowing through the channel.

forward bias - thinner depletion layer
reverse bias - thicker depletion layer

hi K dielectric $HfO_2$ 
increase gate thickness to remove quantum tunneling, use hi permitivity  dielectric to keep the capacitance same coz increasing thickness would cause capacitance to go down.
capacitance of gate $C=\frac{\epsilon_0 A}{tox}$

forward biasing means exponential increase in current.
beware of forward biasing PN junctions.
if we connect body to +ve voltage and source and drain to ground, then these two form PN junction with the substrate that is forward biased, there for current can leak to the substrate through both the source and drain n-type semiconductors.

*cmos inverter*

nmos transistor can only pass $V_{dd} - V_t$ 
as capacitor charges current is droppping, so charging is very slow

NMOS - current flows from drain to source
PMOS - current flows from source to drain
high potential to low potential

nmos can't charge to full $V_{dd}$ , but can discharge capacitor completely,
pmost can't discharge completely, but can charge to $V_{dd}$ 

electrons move from lower to higher potential coz they r -vely charged
holes move from higher to lower potential coz they are +vely charged

PMOS is active low,
low gate voltage makes the gate relatively negative compared to source/body, this attracts holes at the surface forming a channel for holes to flow
![[CMOS.png|350]]

 when PMOS is on , charging
 electrons flow from (drain) the $V_{dd}$ side of metal plate to (source) power supply $V_{dd}$.
 thereby making this plate positive, and the other plate connected to ground is negative compared to this plate, this creates a electric field and stores electrical energy.

when NMOS is on, discharging
electrons flow from (source) ground to metal plate not connected to ground (drain) and this neutralizes the positive charges stored on this plate, thereby discharging the capacitor by removing the electric field.

$E=\frac{1}{2}CV^2$,
C is the capacitance that comes from physical structure.

---

potential difference between two points of charges in a semi-conductor
$\frac{n_1}{n_2} = \epsilon^{\frac{\psi_{12}}{kT/q}}$ , maxwell-boltzmann law

$\frac{kT}{q}$ , thermal voltage, voltage equivalent of thermal energy
$kT$, thermal energy
energy/charge $\frac{kT}{q}$ = voltage
26mV at 300k

high temperature -> large thermal voltage
intrinsic carriers (without doping) increases with temperature


drift velocity, $\nu_d = \mu E$ 
$E=$ electric field
$\mu=$ mobility

in short-channel devices, source and drain are close. threshold voltage depends on drain-source voltage. DIBLowering.
![[subtheshold_leakage.png]]

$\mu_0$ -
zero bias mobility, carrier mobility increases when there is low vertical electric field from the gate.
$\nu_T$ -
thermal voltage, proportional to temperature
this is linear in temperature range 40c-80c.
