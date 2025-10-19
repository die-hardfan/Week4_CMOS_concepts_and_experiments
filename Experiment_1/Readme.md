# EXPERIMENT 1 : MOSFET Behavior & Id vs. Vds Characteristics

## Introduction and Background

MOSFET is the most basic element of any logic cell/gate. Going further down, we enter the device physics domain (with electrons, holes, etc.) or the physical design domain (layout). MOSFET can be NMOS or PMOS, depending on the doping regions and biasing. Below shows the textbook structure of NMOS. 
//add nmos device image
The gate is an important terminal since it controls the switching (or on and off) action of the MOSFET. The body terminal is usually grounded in NMOS or connected to V<sub>DD</sub> in PMOS, and if not, it can control the threshold voltage of the MOSFET. 

### Operation of MOSFET
A MOSFET (Metal-Oxide-Semiconductor Field-Effect Transistor) has three terminals: Gate (G), Drain (D), Source (S). Its operation depends mainly on the gate-to-source voltage V<sub>GS</sub> and the drain-to-source voltage V<sub>DS</sub>.

A long-channel (channel length > 1um) MOSFETs operate in three regions:

1. Cut-off Region (OFF state)

Condition: V<sub>GS</sub> < V<sub>TH</sub> (threshold voltage)

Behavior:
- No conduction occurs between the drain and the source
- The MOSFET is OFF, like an open switch
- Drain current I<sub>D</sub> ≈ 0

2. Triode Region (Linear/Ohmic Region)

Condition: V<sub>GS</sub> > V<sub>TH</sub> and V<sub>DS</sub> < V<sub>GS</sub> - V<sub>TH</sub>

Behavior:
- Channel forms, electrons (for NMOS) flow from source to drain
- MOSFET behaves like a variable resistor
- Drain current I<sub>D</sub> increases linearly with V<sub>DS</sub>

Applications: Switching with low resistance, analog amplification

3. Saturation Region (Active Region)

Condition: V<sub>GS</sub> > V<sub>TH</sub> and V<sub>DS</sub> ≥ V<sub>GS</sub> - V<sub>TH</sub>

Behavior:
- Channel is pinched off near the drain, current becomes almost constant
- MOSFET behaves like a current source
- Drain current I<sub>D</sub> depends mainly on V<sub>GS</sub>, not V<sub>DS</sub>

Applications: Amplifiers, active load circuits

This experiment plots the drain current vs the drain-source voltage for different gate-source voltages. I<sub>D</sub> vs V<sub>DS</sub> graph is useful to understand and identify different regions of operation of the MOSFET, and plotting it for different V<sub>GS</sub> values helps us understand the impact of V<sub>GS</sub> on drain current magnitude. 

Also, plotting the I<sub>D</sub> vs V<sub>DS</sub> graph of NMOS and PMOS is the first step in finding the VTC (Voltage Transfer Characteristic) curve of a CMOS inverter (or any CMOS circuit). And the VTC curve is important to determine the cell delay and noise margins.

## SPICE Netlist

```bash
*Model Description
.param temp=27


*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt


*Netlist Description



XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2

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

- To view the netlist, navigate to `design` folder and give the command as: `gedit <netlist name>`. vim can also be used instead of gedit (any text editor can be used). 
- To simulate: `ngspice <netlist name>`
- To view the I<sub>D</sub> vs V<sub>DS</sub> plot, type the command within ngspice environment: `plot -vdd#branch`

<details>
  <summary> Detailed Explanation of netlist </summary>

- `.param temp=27` - Defines a parameter called temp with a value of 27. In SPICE, this is typically used to set the temperature in degrees Celsius for simulations.
- `.lib "sky130_fd_pr/models/sky130.lib.spice" tt` - Includes a SPICE model library file for the SkyWater 130nm process and "tt" specifies the typical-typical corner (process corner) for the simulation. This allows the use of predefined MOSFET models like `sky130_fd_pr__nfet_01v8` in the netlist description, which contains all the parameters to model the nfet that the simulator needs to produce the outputs (e.g. technology constants like mobility, Cox, lambda, gamma etc.).
- `XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2` - Defines a MOSFET device called XM1 and the terminal connections are: Drain = Vdd, Gate = n1, Source = 0, Substrate = 0 (in the order DGSS). The model used is sky130_fd_pr__nfet_01v8 from the included library. `w=5` and `l=2` specify the width and length of the MOSFET (in microns or um).
- `R1 n1 in 55` - Defines a resistor called R1 connected between nodes n1 and in. Resistance value is 55 ohms.
- `Vdd vdd 0 1.8V` - Defines a DC voltage source called Vdd. Positive terminal connected to node vdd, negative terminal connected to ground (0). Value of Voltage = 1.8V.
- `Vin in 0 1.8V` - Defines another DC voltage source called Vin. Connected between node in and ground. Value of Voltage = 1.8V. This acts as the input signal for the circuit.
- `.op` - Tells SPICE to perform a DC operating point analysis. Calculates node voltages and branch currents at steady state.
- `.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2` - Performs a DC sweep simulation.
    - Vdd 0 1.8 0.1 → Sweep Vdd from 0V to 1.8V in steps of 0.1V. 
    - Vin 0 1.8 0.2 → Sweep Vin from 0V to 1.8V in steps of 0.2V.
    - SPICE will compute I<sub>D</sub>, node voltages, and currents for all combinations of Vdd and Vin.
    - It's like, for every value of Vin, do a Vdd sweep. This is how we get multiple I<sub>D</sub> vs V<sub>DS</sub> graphs in the same window.
      
**Important NOTE**:
- Since a lone NMOS is being analyzed, V<sub>DS</sub> = V<sub>DD</sub>. So to sweep V<sub>DS</sub>, we're sweeping V<sub>DD</sub>. Of course, source names can be changed if required.
- Similarly, V<sub>GS</sub> = V<sub>G</sub> - V<sub>S</sub> where V<sub>G</sub> = V<sub>in</sub> (since no current flows through the gate terminal, the voltage drop across the resistor R is 0) and V<sub>S</sub> = 0 (ground). So sweeping V<sub>in</sub> means we're sweeping V<sub>GS</sub>.

- `.control … .endc` - Control block for SPICE commands. Commands inside this block are executed in the simulator. Inside the control block:
  - `run` → Execute the simulation.
  - `display` → Show results in the SPICE viewer.
  - `setplot dc1` → Select the plot named dc1 for viewing the DC sweep results.
- `.end` - Marks the end of the SPICE netlist.
</details>

In short, the netlist describes the following circuit:

![cmos_inv](/images/cmos_inv.png)

## Plots and Figures

![idvds](/images/idvds_nmos.png)

This plot is given when the command is: `plot -vdd#branch`

<details>
  <summary> The simulation log explanation </summary>

```bash
Note: No compatibility mode selected!


Circuit: *model description

Doing analysis at TEMP = 27.000000 and TNOM = 27.000000

Using SPARSE 1.3 as Direct Linear Solver
 Reference value :  8.00000e-01
No. of Data Rows : 190

No. of Data Rows : 1
Here are the vectors currently active:

Title: *model description
Name: op1 (Operating Point)
Date: Sun Oct 19 11:24:31  2025

    in                  : voltage, real, 1 long
    m.xm1.msky130_fd_pr__nfet_01v8#body: voltage, real, 1 long
    m.xm1.msky130_fd_pr__nfet_01v8#dbody: voltage, real, 1 long
    m.xm1.msky130_fd_pr__nfet_01v8#sbody: voltage, real, 1 long
    n1                  : voltage, real, 1 long
    vdd                 : voltage, real, 1 long [default scale]
    vdd#branch          : current, real, 1 long
    vin#branch          : current, real, 1 long
```

### NGSPICE Simulation Output Explanation

**Note: No compatibility mode selected!**  
NGSPICE supports different "compatibility modes" for older SPICE versions.  
This message just says it’s using modern NGSPICE defaults, nothing is wrong.

---

**Circuit:** *model description*  
Indicates which circuit block (from your netlist) is being analyzed.  
Here, the circuit title is *model description* (from your first comment line).

---

**Doing analysis at TEMP = 27.000000 and TNOM = 27.000000**  
The simulation is using 27°C as the operating temperature.  
TNOM is the nominal temperature for device models.

---

**Using SPARSE 1.3 as Direct Linear Solver**  
NGSPICE is solving the circuit equations using a direct solver called SPARSE.  
This handles the matrix equations generated from KCL (Kirchhoff’s Current Law) for all nodes.

---

**Reference value : 2.00000e-01**  
This is a default scaling reference used internally by NGSPICE for numerical calculations.

---

**No. of Data Rows : 190**  
If you ran a DC sweep, this means 190 combinations of VDD and Vin were simulated.  
Each "row" corresponds to one combination of inputs. 
Vdd : 0 to 1.8 in steps of 0.1, so 19 points, and Vin: 0 to 1.8 in steps of 0.2, so 10 points --> 19*10=190. 

---

**No. of Data Rows : 1**  
For the operating point analysis (`.op`), there is only 1 row because the simulation computes just one static solution.

<details>
  <summary>Details on Operating point analysis</summary>

  What is Operating Point Analysis (.op)?

Operating point analysis in SPICE is a DC analysis at a single instant, where the simulator calculates:
- The voltage at every node in the circuit.
- The current through every branch (especially voltage sources).
- It does not sweep any voltages or time—it just finds the steady-state solution of the circuit for the given input sources.
- Mathematically, SPICE solves Kirchhoff’s Current Law (KCL) equations for all nodes.
- This gives the DC voltage at each node and the currents in each branch.

Why is there only 1 row in .op output?
- .op calculates the solution for the circuit at the exact values of all sources you specified (here Vdd = 1.8V and Vin = 1.8V).
- It does not vary Vdd, Vin, or any parameter.
- Therefore, there is only one solution, i.e., one row of node voltages and currents.
- In a .dc sweep, SPICE would calculate multiple rows—one for each combination of swept voltages—but .op only cares about the single point.
</details>

---

**Here are the vectors currently active:**  
This lists all node voltages and branch currents that NGSPICE is tracking for this simulation.

| Vector Name                           | Type    | Meaning                                      |
|--------------------------------------|---------|----------------------------------------------|
| in                                   | voltage | Voltage at node in                           |
| m.xm1.msky130_fd_pr__nfet_01v8#body | voltage | Body (bulk) voltage of the MOSFET           |
| m.xm1.msky130_fd_pr__nfet_01v8#dbody| voltage | Drain-body voltage node inside the MOSFET   |
| m.xm1.msky130_fd_pr__nfet_01v8#sbody| voltage | Source-body voltage node inside the MOSFET  |
| n1                                   | voltage | Voltage at node n1 (gate of NMOS)           |
| vdd                                  | voltage | Voltage at node vdd (top of NMOS and resistor) |
| vdd#branch                           | current | Current through the Vdd voltage source      |
| vin#branch                           | current | Current through the Vin voltage source      |

</details>

<details>
  <summary>Simulation log</summary>
  
![idvds_sim](/images/idvds_sim.png)

</details>

Why `plot -vdd#branch` gives the correct graph? 

  - current direction for vdd#branch is measured through the voltage source vdd (top to bottom) but I<sub>D</sub> has a direction opposite to this (drain to source), so -vdd#branch flips the graph about the horizontal axis.
    
using the command: `plot vdd#branch` gives the below plot:

![plot_vdd_branch](/images/plotvddbranchvsvdd.png)

## Observations and Analysis

![annotated_idvds](/images/annotated_idvds.png)

In the region within the green box, the behaviour of NMOS is almost linear (resistive) and the point at which saturation is reached keeps shifting right as V<sub>GS</sub> increases (curves move up). In the region within the yellow box, the curve is almost flat, since for long channel MOSFETs, lambda (channel length modulation constant) is almost 0 and thus, NMOS behaves like an ideal current source. The curve parallel to x-axis in the bottom shows the cutoff region graph since Vin < V<sub>T</sub>. 

## Conclusions

This experiment served to get familiar with the Ngspice software and understand the basics of MOSFET behaviour. Theoretical concepts were understood and mapped to the simulation results which helped clearing the concepts further. 
