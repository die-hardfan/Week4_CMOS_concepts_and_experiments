# EXPERIMENT 3: CMOS Inverter: Voltage Transfer Characteristic (VTC) 

## Introduction and Background

The CMOS (complementary MOS) inverter is made of 2 MOSFETs, an NMOS and a PMOS. The circuit is given below:

![cmos_inv](/images/cmos_inv.png)

The voltage transfer characteristic (VTC) curve is derived from the I<sub>D</sub> vs V<sub>DS</sub> graphs for PMOS and NMOS in this configuration. For an independent MOSFET, I<sub>D</sub> vs V<sub>DS</sub> graph typically
explains it's characteristics. But with a network or circuit of multiple MOSFETs, as in any CMOS circuit, to plot these graphs require internal node voltages which is not visible to users. If the circuits were characterised by 
internal voltages, then testing these circuits post-fabrication wouldn't be possible, because circuits would look like a black box with only input and output ports available for testing. Thus, we need to characterise circuits
in terms of it's input and output ports (V<sub>in</sub> and V<sub>out</sub>). The graph that tells the function/operation of a circuit using V<sub>in</sub> and V<sub>out</sub> is called Voltage Transfer Characteristic (VTC) curve. 
(VTC describes how the input voltage _transfers_ to output voltage in a circuit, which is usually unique hence it's a _characteristic_ of that circuit). 

Steps to find VTC curve from <sub>D</sub> vs V<sub>DS</sub> graphs for a CMOS inverter. (The same steps is used to find VTC curves for any CMOS circuit)
Assume the following:

![assumption_vtc](/images/assumption_vtc.png)

1. Find <sub>D</sub> vs V<sub>DS</sub> graph for PMOS in this configuration (it's different from the graph when PMOS is analyzed independently, since PMOS is on the top and in a circuit).
2. The directions of PMOS current and voltage is opposite of NMOS, so the graph obtained will be in the IIIrd quadrant. Vin = Vgsp + Vdd and Vout = Vdsp + Vdd, modify the graph accordingly. 
3. Find <sub>D</sub> vs V<sub>DS</sub> graph for NMOS (this is easier since the placement of NMOS in the circuit is similar to how it would have been analyzed independently). Vin = Vgsn and Vout = Vdsn, so nothing changes.
4. Superimpose both the graphs, and mark the intersection points.
5. Draw the VTC curve using these marked points.

![idvds_vtc](/images/idvds_vtc.png)

VTC curve is most important since it is used to estimate/determine the cell delay. This experiment considers the case where W<sub>p</sub> = 2.5*W<sub>n</sub> approximately. This is the ideal ratio to get an ideal switching threshold, 
V<sub>m</sub> = Vdd/2. 

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

The circuit is shown below:
![cmos_circuit](/images/cmos_circuit.png)

The netlist description changes accordingly, and other section remain similar to the netlist in Experiment_1. 

<details>
  <summary>Simulation log</summary>

![vtc_vm_sim](/images/vtc_vm_sim.png)

</details>


## Observations and Analysis

![vtc_plot](/images/vtc_plot.png)

The region of interest is highlighted by the orange box. To find the point where Vin = Vout (the switching threshold Vm), we need to zoom in (for ngspice-45+, right-click and drag results in the slope calculation, so just right click at a point near the curve, a small box is automatically selected and zoomed in - trial and error required). 
Vm = 0.876 V (it's accurate upto 3 decimal digits). Vm is not exactly equal to Vdd/2 (1.8/2 = 0.9) because Wp = 0.84 um and Wn = 0.36 um, Wp = 2.33*Wn (not exactly 2.5 times), so there's a decrease in accuracy. 

## Conclusions
The CMOS inverter’s Voltage Transfer Characteristic (VTC) curve was successfully plotted and analyzed.
The obtained switching threshold voltage (V<sub>m</sub> = 0.876 V) is close to the ideal value of V<sub>DD</sub>/2 = 0.9 V, confirming nearly symmetrical operation. The slight deviation is due to the Wp/Wn ratio (2.33 instead of ideal 2.5), causing a minor imbalance between the pull-up (PMOS) and pull-down (NMOS) strengths.

The VTC curve clearly demonstrates the three regions of inverter operation — logic high, transition, and logic low — and validates the inverter’s function as a basic digital logic element with sharp switching characteristics and high noise margins.

---
