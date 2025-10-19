# EXPERIMENT 6a: CMOS Inverter: Power Supply Variation Studies

## Introduction and Background

The supply voltage (VDD) is a key factor that directly affects the performance of a CMOS inverter. Scaling VDD impacts several important characteristics, including the switching threshold, noise margins, propagation delay, and power consumption.

Advantages: 
- Increase in gain as the power supply (Vdd) is scaled down. The slope of the VTC curve of CMOS inverter increases, becomes more steep. This helps in analog applications (larger amplification of small signals).
- Energy = 1/2 * CV<sup>2</sup>, the voltage here is Vdd. So as Vdd decreases, the energy consumed by the circuit decreases which leads to longer battery life or just low power requirements.

Disadvantages:
- The transition time increases since charging and discharging requires time (lower Vdd, less current, lower charging/discharging rate for the same capacitance). This impacts the performance of the circuit.

Noise itself is categorized as harmful or not. Consider a glitch for e.g. and the below chart:
//add the glitch chart here
Glitches that reach beyond V<sub>IL</sub> are dangerous because, the signal may now be interpreted as _x_ (metastable stable - neither valid logic 1 or 0) or higher levels of glitches are interpreted as logic 1. Glitches that make a logic 0 into logic 1 are dangerous because
this gives the wrong output (but also for power and other reasons). Suppose, in a safety critical system, a logic 1 means enables or disables a function, it could lead to dangerous actions. Of course, the duration of a glitch is as important as it's magnitude. 

Studying the effect of VDD variations helps designers understand the trade-offs between speed, power, and robustness, and ensures that the inverter functions reliably across the expected range of supply voltages. This is especially important for low-power designs, voltage scaling techniques, and circuits operating under variable supply conditions.
This experiment scales down the Vdd value from 1.8 to 0.8 volts in steps of 0.2. The VTC curve for each case is plot, and the variation in Vm and noise margins is tabulated.

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
Vin in 0 1.8V

.control

let powersupply = 1.8
alter Vdd = powersupply
	let voltagesupplyvariation = 0
	dowhile voltagesupplyvariation < 6
	dc Vin 0 1.8 0.01
	let powersupply = powersupply - 0.2
	alter Vdd = powersupply
	let voltagesupplyvariation = voltagesupplyvariation + 1
      end
 
plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in dc6.out vs in xlabel "input voltage(V)" ylabel "output voltage(V)" title "Inveter dc characteristics as a function of supply voltage"

.endc

.end

```

- To view the netlist, navigate to `design` folder and give the command as: `gedit <netlist name>`. vim can also be used instead of gedit (any text editor can be used). 
- To simulate: `ngspice <netlist name>`
- The plot is automatically opened upon completion of the simulation.

The circuit is shown below:
//add circuit pic

**Smart SPICE simulation:**

Within `.control ... .endc`, we can do some scripting (or coding) to make our work easier, specially when simulations are to be done over various values of parameters. This is called smart SPICE simulation. The below is an example of it:

```bash
let powersupply = 1.8
alter Vdd = powersupply
	let voltagesupplyvariation = 0
	dowhile voltagesupplyvariation < 6
	dc Vin 0 1.8 0.01
	let powersupply = powersupply - 0.2
	alter Vdd = powersupply
	let voltagesupplyvariation = voltagesupplyvariation + 1
      end
```
This is where power-supply variation analysis happens:

1. Initialization:
   - `let powersupply = 1.8` → start with 1.8 V supply.
   - `alter Vdd = powersupply` → sets the VDD node to the current supply voltage.
   - `let voltagesupplyvariation = 0` → a counter for iterations.
2. Loop (dowhile): The loop repeats 6 times (voltagesupplyvariation < 6).

   In each iteration:
    - `.dc Vin 0 1.8 0.01` → sweeps input voltage from 0 to 1.8 V in steps of 0.01 V to generate the VTC curve for the current VDD.
    - `let powersupply = powersupply - 0.2` → decreases VDD by 0.2 V for the next iteration.
    - `alter Vdd = powersupply` → updates the VDD node to the new voltage.
    - `voltagesupplyvariation = voltagesupplyvariation + 1` → increment loop counter.

Effect: This produces 6 VTC curves corresponding to supply voltages: 1.8 V, 1.6 V, 1.4 V, 1.2 V, 1.0 V, and 0.8 V.

**Plotting multiple graphs**

```bash
plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in dc6.out vs in xlabel "input voltage(V)" ylabel "output voltage(V)" title "Inverter dc characteristics as a function of supply voltage"
```
- Plots the output voltage vs input voltage for all 6 DC sweeps in the same graph.
- Each `dcX.out` corresponds to one .dc sweep at a specific VDD.
- The graph shows how the VTC curve shifts and scales as the supply voltage decreases.

Except for these changes, the other parts of the netlist remains same as in Experiment_3 (CMOS inverter).

<details>
  <summary>Simulation log</summary>

//add the terminal pic here 

</details>


## Observations, Analysis and Tabulated Results

//power_supply_var_vm

As observed, the switching threshold decreases as Vdd decreases, because the supply voltage is decreasing and ideally, for a CMOS inverter, Vm = Vdd/2. So, as Vdd decreases, Vm decreases (though it's not equal to Vdd/2 since the inverter is not ideal cuz 
Wp = 2.33xWn, so the CMOS inverter is not ideal). 

//power_supply_var_lower
//power_supply_var_upper

These plots show the noise margin trend. 

//add the table here
Clearly, the low and high noise margins decrease as Vdd is decreased, which is not a good thing, because the lower noise margins mean lower noise immunity. Cells with lower Vdd, thus are more vulnerable to noise. 
This is again, a tradeoff between power consumption and noise immunity.

It is important to note that regardless of the power supply scaling, the shape of the curves remain constant, that means it's function doesn't change. This proves the robustness of the CMOS inverter (and CMOS circuit) against power supply variation with respect to it's operation and function.


## Conclusions
The experiment demonstrated how scaling the supply voltage (VDD) affects the CMOS inverter’s performance. As VDD decreases from 1.8 V to 0.8 V, the switching threshold voltage (Vm) reduces proportionally, and both low and high noise margins decrease, indicating reduced noise immunity. This shows that lower supply voltages, while reducing power consumption and energy per switching event, make the inverter more susceptible to noise. The study 
highlights the trade-off between low-power operation and robustness, emphasizing that designers must carefully select VDD to balance speed, noise tolerance, and energy efficiency in CMOS circuits.

---

# EXPERIMENT 6b: CMOS Inverter: Device Variation Studies

## Introduction and Background

Variations in the device during fabrication can lead to variations in the characteristics of the device. 

Sources of variation:

1. Etching process variations: Etching is the process of removing soluble photoresist to leave behind the desired pattern after photolithography. Poor etchning process can, thus, produce variations in the dimensions of the device itself.
2. Oxide thickness variation: Oxide layer makes up the gate terminal of the MOSFET (also provides insulation), and this is either grown or deposited on the wafer. Variations in the thickness of this oxide will lead to variations is the capacitance but also variations in the effects of other devices (perhaps due to differing dimensions, the insulation layer is thin or there are defects/cracks within, which can lead to poor insulation; poor insulation means when other devices are working, this device is affected)

In MOSFET, Id ∝ W/L and Id ∝ 1/t<sub>ox</sub>, thus, variations in etching and oxidation process produce variations in Id. Id is used to estimate the cell delay. So, variations in Id, produces variations in cell delay as well. 

Further, these variations are not constant but unique and slightly different for each device, because each device is surrounded by different set of devices, so they impact differently.

As an experiment to include process variations into SPICE simulations, we use variations in width of NMOS and PMOS to plot the VTC curves. Larger PMOS/NMOS means a strong PMOS/NMOS cuz they offer less resistance, and narrower device means it's weak. 
The channel length of the transistors remain constant, only the width changes. Wp = 7um and Wn = 0.42um. This represents a strong PMOS and a weak NMOS. Similarly, simulate for Wn = 7um and Wp = 0.42um, a strong NMOS and weak PMOS. Compare VTC, noise margin and delays.


## SPICE Netlist

```bash
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description


XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=7 l=0.15  # w=0.42 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.42 l=0.15   # w=7 l=0.15


Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op

.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
.endc

.end

```

- To view the netlist, navigate to `design` folder and give the command as: `gedit <netlist name>`. vim can also be used instead of gedit (any text editor can be used). 
- To simulate: `ngspice <netlist name>`
- To view the VTC curve, within ngspice environment, type: `plot out vs in`

The circuit is shown below:
//add circuit pic

Except for these changes, the other parts of the netlist remains same as in Experiment_3 (CMOS inverter).

Repeat the experiment with interchanged Wp and Wn values. 

VTC curve gives switching threshold and noise margins. Transient analysis gives the delays (refer Experiment_4 netlist for that).

<details>
  <summary>Simulation log</summary>

//add the terminal pic here 
//

</details>


## Observations, Analysis and Tabulated Results

//vtc

Clearly, as the width of weak PMOS has a lower switching threshold that strong PMOS. Also, the VTC curve seems to have shifted left. 

//transient analysis

For a strong PMOS and weak NMOS, the rise delay is low and fall delay is high. Because, wider PMOS, more current, charges the capacitance fast. But NMOS is narrow, less current, so discharge rate is slow, hence fall delay is high. The exact opposite happens for weak PMOS and strong NMOS.



## Conclusions
This experiment demonstrated how variations in device dimensions during fabrication impact the electrical behavior of a CMOS inverter. By altering the transistor widths (Wp and Wn), we observed shifts in the VTC curve, switching threshold, and propagation delays. When PMOS is strong (wide) and NMOS is weak (narrow), the switching threshold shifts lower, rise delay decreases, and fall delay increases. Conversely, when NMOS is strong and PMOS is weak, the switching threshold shifts higher, rise delay increases, and fall delay decreases. These results show that device mismatches cause asymmetry in inverter performance, affecting both speed and noise margins. Hence, maintaining precise control over fabrication parameters such as transistor width, length, and oxide thickness is essential for achieving consistent and predictable circuit behavior across all chips.


