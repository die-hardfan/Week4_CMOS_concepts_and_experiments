# EXPERIMENT 2a: Threshold Voltage Extraction

## Introduction and Background

The threshold voltage (V<sub>th</sub>) of a MOSFET is the gate-to-source voltage at which a conducting channel just forms at the semiconductor-oxide interface, causing the surface to undergo strong inversion. For an NMOS, the p-type substrate surface becomes n-type, while for a PMOS, the n-type surface becomes p-type. V<sub>th</sub> is positive for NMOS and negative for PMOS. It is affected by device parameters including substrate doping, oxide thickness, temperature, and the source-to-body voltage (body effect). At exactly V<sub>th</sub>, the drain current is small, and significant conduction occurs only slightly above this voltage. MOSFETs may be enhancement-mode (normally off, channel forms above V<sub>th</sub>) or depletion-mode (normally on, channel exists at V<sub>GS</sub>=0). Understanding V<sub>th</sub> is crucial for predicting MOSFET behavior in circuits and for designing devices with proper switching characteristics.

This experiment plots the I<sub>D</sub> vs V<sub>GS</sub> graph. It is essential to extract the threshold voltage of the NMOS. The experiment is performed on a short channel device with channel length = 0.15um (<1um). A short channel length also has some impact on the device characteristics compared to long channel MOSFET, so the comparison is also shown. 

## SPICE Netlist

```bash
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description

XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15 

R1 n1 in 55

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op
.dc Vin 0 1.8 0.1 

.control

run
display
setplot dc1
.endc

.end
```

- To view the netlist, navigate to `design` folder and give the command as: `gedit <netlist name>`. vim can also be used instead of gedit (any text editor can be used). 
- To simulate: `ngspice <netlist name>`
- To view the I<sub>D</sub> vs V<sub>DS</sub> plot, type the command within ngspice environment: `plot -vdd#branch`

The netlist describes the same circuit as shown in Experiment_1. The circuit is shown below:

![nmos_circuit](/images/nmos_circuit.png)

The only points of change are: 

- `XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15` - the width and length of the channel are now submicron level
- `.dc Vin 0 1.8 0.1 ` - dc sweep is done for V<sub>GS</sub> only, so the plot obtained will be I<sub>D</sub> vs V<sub>GS</sub> when the command `plot -vdd#branch` is used (cuz there's no other voltage values to plot against).

The experiment is repeated for long channel NMOS with `w=5 l=2`. Note that the aspect ratio (W/L) remains almost the same, so the only change is the dimensions themselves. 

<details>
  <summary>Simulation log</summary>

![vt_sim](/images/vt_sim.png)

![vt_long_sim](/images/vt_long_sim.png)

</details>


## Observations and Analysis

![vt_graph](/images/vt_graph.png)

The threshold voltage is found as the V<sub>GS</sub> value when I<sub>D</sub> starts increasing non-linearly (not exponential, that is for a diode). Draw a tangent at that point, extend it to intersect with the x-axis (V<sub>GS</sub> axis), and we get an approximate V<sub>Th</sub> value as 0.7 V. 

![vt_long_channel](/images/vt_long_channel.png)

Clearly, the threshold voltage has increased as the dimension decreased. Long channel NMOS V<sub>Th</sub> = 0.703 V and short channel NMOS V<sub>Th</sub> = 0.727 V. The change is not much, but it's present. 

## Conclusions

From the I<sub>D</sub> vs V<sub>GS</sub> graph of the short-channel NMOS device, the threshold voltage (V<sub>th</sub>) was determined as the gate voltage at which the drain current begins to increase significantly, indicating the formation of a conducting channel. The observed V<sub>th</sub> value (~0.7 V in this case) confirms that the device is an enhancement-mode NMOS. While a MOSFET’s I–V behavior is not identical to a diode, both exhibit a threshold-like voltage before appreciable current flows. Thus, with respect to the onset of conduction, the MOSFET can be qualitatively compared to a diode, though its current rises quadratically (or almost linearly in short-channel devices) rather than exponentially. The experiment demonstrates both the practical determination of V<sub>th</sub> and the conceptual similarity to PN junction turn-on behavior.

---

# EXPERIMENT 2b: Velocity Saturation Effect

## Introduction and Background

In submicron channel length (short channel) MOSFETs, there are 4 regions of operation, unlike 3 in long channel MOSFETs. The new addition is called **Velocity Saturation Region**. To incorporate this into the MOSFET model, the previous I<sub>D</sub> equations cannot be used. Hence a new equation is used, as shown below: 

```text
Id = kn. [(Vgt.Vmin)-Vmin^2/2].[1+λVds]

where
Vmin = min(Vgt, Vds, Vdsat)
Vdsat = Vd at which mosfet goes to saturation
Vgt = Vgs - Vt
```

The reason for this effect requires understanding of device physics, which is beyond the scope of this experiment. In short, due to this effect, the maximum achievable I<sub>D</sub> is decreased, relative to long channel NMOS, for a constant V<sub>DS</sub>. The MOSFET also behaves more linearly during saturation. 

This experiment compares the I<sub>D</sub> vs V<sub>GS</sub> graph for long and short channel NMOS. A comparative study of I<sub>D</sub> vs V<sub>DS</sub> graphs are also made. Short channel W/L = 0.39/0.15 (in um) and long channel W/L = 5/2 (in um), the ratio remains almost same, so the differences in I<sub>D</sub> seen is due to  the decrease in device dimensions and not because of the aspect ratio.

## SPICE Netlist

For I<sub>D</sub> vs V<sub>GS</sub> graph for short channel NMOS: 
```bash
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description

XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15 
R1 n1 in 55

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op
.dc Vin 0 1.8 0.1 

.control

run
display
setplot dc1
.endc

.end
```
To get the I<sub>D</sub> vs V<sub>GS</sub> graph for long channel NMOS, modify the w and l values as: `w=5 l=2`. 

For I<sub>D</sub> vs V<sub>DS</sub> graph of short channel NMOS: 
```bash
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description



XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15 

R1 n1 in 55

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands

.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control

run
display
setplot dc1
.endc

.end

```
The I<sub>D</sub> vs V<sub>DS</sub> graph for long channel NMOS is the one from Experiment_1. 

For both the graphs, follow the given commands for simulation:
- To view the netlist, navigate to `design` folder and give the command as: `gedit <netlist name>`. vim can also be used instead of gedit (any text editor can be used). 
- To simulate: `ngspice <netlist name>`
- To view the I<sub>D</sub> vs V<sub>DS</sub> plot, type the command within ngspice environment: `plot -vdd#branch`

The netlist describes the same circuit as shown in Experiment_1. The circuit is shown below:

![nmos_circuit](/images/nmos_circuit.png)

<details>
  <summary>Simulation log</summary>

![velo_sat_sim](/images/velosat_sim.png)

</details>


## Observations and Analysis

![short_long_vt](/images/short_long_vt.png)

Clearly in the I<sub>D</sub> vs V<sub>GS</sub> plot, the increase in current is more linear in short channel NMOS but non-linear (or quadratic) in long channel NMOS. This is because, due to velocity saturation effect, the NMOS saturates fast and current remains almost constant. 

![short_long_idvds](/images/short_long_idvds.png)

Similarly, in the I<sub>D</sub> vs V<sub>DS</sub> plot, the effect of channel length modulation (during saturation, Id is still dependent on Vds) is more prominent in long channel NMOS. In the short channel NMOS, in the saturation region, current is almost constant. Also, the maximum current reached (the maximum Id value for topmost curve - maximum Vgs) in long channel NMOS is more that in short channel NMOS, though according to the Id current equation, Id is inversely proportional to L (length of the channel). This difference is due to the velocity saturation effect, which limits the current value as channel length decreases, submicron. 

## Conclusions
From the comparative analysis of the long-channel and short-channel NMOS devices, it is evident that the reduction in channel length significantly alters the device’s electrical behavior. The short-channel NMOS shows a more linear I<sub>D</sub>–V<sub>GS</sub> characteristic, confirming the presence of velocity saturation. In the I<sub>D</sub>–V<sub>DS</sub> characteristics, the channel length modulation effect is more evident in the long-channel NMOS, while the short-channel device exhibits an almost constant current in saturation. Overall, the experiment clearly demonstrates that as MOSFET dimensions shrink to submicron levels, velocity saturation effect is introduced, leading to reduced current, a more linear response, and deviation from the ideal long-channel behavior predicted by the earlier MOSFET equations.
