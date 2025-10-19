# EXPERIMENT 4: CMOS Inverter: Transient Behavior: Rise / Fall Delays

## Introduction and Background

While the VTC (Voltage Transfer Characteristic) describes the static behavior of a CMOS inverter (i.e., how output voltage changes with input voltage under steady-state conditions), transient analysis focuses on the dynamic response — how fast the inverter switches between logic states when the input changes over time.
When a digital signal transitions, the output does not change instantaneously due to the capacitive loading at the output node and the finite drive strength of the MOSFETs. This results in two key timing parameters:
- Rise Time (t<sub>r</sub>): The time taken for the output voltage to rise from 10% to 90% of V<sub>DD</sub> when the inverter output transitions from LOW → HIGH.
- Fall Time (t<sub>f</sub>): The time taken for the output voltage to fall from 90% to 10% of V<sub>DD</sub> when the inverter output transitions from HIGH → LOW.

Correspondingly, two delay parameters are defined:
- Rise Delay (t<sub>PLH</sub>): The time difference between the input signal crossing 50% of V<sub>DD</sub> (on falling edge) and the output crossing 50% of V<sub>DD</sub> (on rising edge).
- Fall Delay (t<sub>PHL</sub>): The time difference between the input signal crossing 50% of V<sub>DD</sub> (on rising edge) and the output crossing 50% of V<sub>DD</sub> (on falling edge).

These parameters determine the speed and performance of the inverter, and by extension, the entire CMOS logic family. The transient behavior thus provides critical insight into propagation delay, power consumption, and signal integrity in digital circuits.

This experiment assumes Wp = 2.5*Wn (both submicron) and aims to find the rise and fall delays using transient analysis (the simulation of voltage values with respect to time). 

## SPICE Netlist

```bash
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description


XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15


Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

*simulation commands

.tran 1n 10n

.control
run
.endc

.end
```

- To view the netlist, navigate to `design` folder and give the command as: `gedit <netlist name>`. vim can also be used instead of gedit (any text editor can be used). 
- To simulate: `ngspice <netlist name>`
- To view the transient curve, type the command within ngspice environment: `plot out in`
- In general `plot signal1 signal2 ... signalN` shows all the N signals in the same window, in transient analysis the x-axis for all of them is time, by default.

Points of change in the netlist:
- `Cload` is assigned a value of 50fF for simulation purposes only, in reality, the calculation of C<sub>load</sub> requires some extensive theoretical concepts and equations. It is calculated using values of input capacitances of the next stage of circuit (i.e. the circuit connected to the output of this circuit).
- `Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)` - defining a voltage pulse input with voltage limits: 0 to 1.8, 0 initial shift (0 skew basically), 0.1ns rise delay and 0.1ns fall delay, 2ns pulse width and 4ns cycle time.
- `.tran 1n 10n` - command to perform transient analysis for 10ns (total) with step (increment) of 1ns. This means that at every 1ns, from intial time=0ns, the signal values (node voltages and branch currents) are calculated.

Note that operating point analysis is not included here since `.op` is not there.

<details>
  <summary>Simulation log</summary>

![trans_inv_sim](/images/trans_inv_sim.png)

</details>


## Observations and Analysis

![trans_inv](/images/trans_inv.png)

The orange boxes represent the interested region where rise and fall delay is to be calculated. when **output** goes from 0-->1, we calculate rise delay as: point 1 - point 2, and when **output** goes from 1-->0, 
we calculate fall delay as: point 3 - point 4. 
- Rise delay = 2.47 ns - 2.16 ns = 0.31 ns
- Fall delay = 4.32 ns - 4.03 ns = 0.29 ns
- Rise delay = fall delay (approximately) because Wp = 2.33xWn (if Wp = 2.5xWn, then rise delay and fall delay would be exactly equal). PMOS needs to be wider than NMOS for this condition because, mobility of holes is much lesser than electrons.
Greater mobility means more current cuz Id ∝ u (u is mobility of holes or electrons), referred to from MOSFET drain current equation. So, effectively, NMOS offers less resistance than PMOS. To equalise this resistance, Wp is increased because: R ∝ 1/A (area = W*L).
It happens that u<sub>n</sub> = 2.5xu<sub>p</sub> (approximately) which is why Wp = 2.5xWn is preferred.
- An equal rise and fall delay reduces pulse degradation, particularly when signals such as clock must travel across the area of the chip, through buffers/inverters as part of clock tree.
- Of course, cells with unequal rise and fall delays exist, and they are utilized in datapaths for a tradeoff between delay and area (wider transistors --> lower delay). 

## Conclusions
The transient analysis of the CMOS inverter demonstrated the dynamic switching behavior, allowing calculation of rise and fall delays. The measured rise delay (t<sub>PLH</sub> = 0.31 ns) and fall delay (t<sub>PHL</sub> = 0.29 ns) are approximately equal, confirming that the sizing ratio W<sub>p</sub> ≈ 2.33×W<sub>n</sub> effectively balances the slower hole mobility with the faster electron mobility to equalize switching speeds. This balance ensures minimal pulse distortion and uniform propagation delays, which is crucial for timing-critical digital circuits such as clock distribution networks. The experiment highlights the importance of output load capacitance and transistor sizing in determining inverter speed, validating theoretical principles of MOSFET current and mobility in practical circuit operation.

---
