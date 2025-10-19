# EXPERIMENT 5: CMOS Inverter: Noise Margin / Robustness Analysis

## Introduction and Background

In digital circuits, signals are represented as logic HIGH and logic LOW voltages. However, 
real circuits often experience voltage fluctuations due to noise, interference, or variations in manufacturing. 
The ability of a digital gate, such as a CMOS inverter, to tolerate these fluctuations without producing incorrect outputs is measured by its noise margins. 
High noise margins make digital circuits more robust and reliable, allowing them to function correctly even in noisy environments or when driving multiple stages.

To analyze how reliably a digital gate like a CMOS inverter operates under such conditions, certain voltage levels are defined from its Voltage Transfer Characteristic (VTC):
- V<sub>IL</sub> is the maximum input voltage that is still recognized as a logic LOW.
- V<sub>IH</sub> is the minimum input voltage that is recognized as a logic HIGH.
- V<sub>OL</sub> is the maximum output voltage that the inverter produces when output is supposed to be LOW.
- V<sub>OH</sub> is the minimum output voltage that the inverter produces when output is supposed to be HIGH.

Using these definitions, the noise margins of the inverter can be calculated:
- Noise Margin High (NM<sub>H</sub>) is the maximum voltage noise that can be added to a logic HIGH input without causing the output to drop below the recognized HIGH level.
- It is given by the formula: NM<sub>H</sub> = V<sub>OH</sub> - V<sub>IH</sub>.
- Noise Margin Low (NM<sub>L</sub>) is the maximum voltage noise that can be added to a logic LOW input without causing the output to rise above the recognized LOW level.
- It is given by the formula: NM<sub>L</sub> = V<sub>IL</sub> - V<sub>OL</sub>.

Noise generally includes glitches, cross-talk, power-supply variations, jitter, etc. Usually, in digital circuits, the noise induced when a signal travels from output of a cell to the input of another cell is more, 
since they are exposed to surrounding components and wires carrying other signals, compared to noise induced within the circuit. This is especially so with CMOS circuits. So, the margin for input low must be higher than output low and margin for input high must be lower than output high.

Typically: V<sub>DD</sub> >= V<sub>OH</sub> > V<sub>IH</sub> > V<sub>IL</sub> > V<sub>OH</sub> >= 0, as shown in the below pic: 
//add noise margin pic here.

This experiment uses VTC graph of a CMOS inverter to determine the noises margins.

## SPICE Netlist

```bash
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description


XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15


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
- To view the VTC curve, type the command within ngspice environment: `plot out vs in`

The netlist is the same as the one used in Experiment_3 (for VTC curve).

<details>
  <summary>Simulation log</summary>

![noise_margin_sim](/images/noise_margin_sim.png)

</details>


## Observations and Analysis

![noise_margin_plot](/images/noise_margin_plot.png)

In a VTC graph, there are 2 points where the slope is found to be -1. Draw a tangent at that point. The upper point is found to be: 0.74, 1.74 (Vin, Vout) and the lower point is found to be: 0.99, 0.09 (Vin, Vout).
- V<sub>IL</sub> = 0.74 V
- V<sub>IH</sub> = 0.99 V
- V<sub>OL</sub> = 0.09 V
- V<sub>OH</sub> = 1.74 V
- NM<sub>L</sub> = 0.74 - 0.09 = 0.65 V 
- NM<sub>H</sub> = 1.74 - 0.99 = 0.75 V

It can be clearly observed that V<sub>OL</sub> = 0 V (approx) and V<sub>OH</sub> = 1.8 V (Vdd, approx). Thus the output swing (V<sub>OH</sub> - V<sub>OL</sub>) = Vdd (approx). This is an almost complete output swing. The larger the swing, the more noise immunity present. 
This gives a high noise immunity because, now, when output of an inverter travels to another cell, before it reaches the input of the next stage, it can accomodate an increase of 0.65V (if the signal was low) and a decrease of 0.75 (if the signal was high) due to noise. 
Thus a CMOS inverter (or a CMOS circuit) is robust against noise. 


## Conclusions
The CMOS inverter’s noise margins were successfully determined from the Voltage Transfer Characteristic (VTC). The measured values, Noise Margin Low = 0.65 V and Noise Margin High = 0.75 V, indicate that the inverter can tolerate significant voltage disturbances without producing incorrect outputs. The output swing is nearly equal to Vdd, confirming almost full logic levels and strong noise immunity. This demonstrates that CMOS inverters are inherently robust, capable of driving subsequent stages reliably even in the presence of noise, voltage fluctuations, or interference, making them highly suitable for stable and dependable digital circuit operation.

---
