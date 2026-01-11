# Design and Performance Analysis of a 6T SRAM Cell and Peripherals

**Project Report** **Implemented using 90nm CMOS Technology**

**Author:** Ishaan Singhal  
**Institution:** Delhi Technological University

---

## 1. Introduction

Static Random Access Memory (SRAM) is a critical component in modern computing architectures, serving as high-speed cache memory (L1, L2, L3) for Central Processing Units (CPUs). The term "Static" indicates that, unlike Dynamic RAM (DRAM), SRAM utilizes a bistable latching mechanism to retain data as long as power is supplied, eliminating the need for periodic refresh cycles.

In the context of 90nm CMOS technology, designing SRAM poses specific challenges regarding leakage currents and stability. While DRAM relies on a single capacitor for charge storage (which is subject to leakage and requires constant refreshing), SRAM employs a six-transistor configuration to form a robust storage cell. This architecture offers significantly lower latency and higher stability, making it ideal for time-critical data access, albeit at the cost of increased silicon area and complexity compared to DRAM.

### Project Objective
The primary objective of this project is to design, simulate, and analyze a complete 16-bit (4x4) SRAM memory array. The scope extends beyond the fundamental storage unit to include the necessary peripheral circuitry required for practical operation. The project is structured into four key phases:

1. **Core Design:** Schematic design and transient simulation of the 6T SRAM Cell, focusing on transistor sizing for read/write stability.
2. **Stability Analysis:** Extraction of the Static Noise Margin (SNM) using Butterfly Curves to quantitatively assess the cell's immunity to noise.
3. **Peripheral Circuit Design:**
    * **Pre-Charge Circuit:** Ensures bit lines are equalized to $V_{DD}$ prior to read operations to prevent accidental data corruption.
    * **Sense Amplifier:** Detects and amplifies small differential voltages on the bit lines, allowing for faster read speeds.
    * **Write Driver:** Provides the necessary high-current drive to overwrite the cell's internal feedback loop during write operations.
4. **Integration:** Interfacing all components to form a functional 4x4 Memory Array and verifying timing constraints.

This report presents the schematic design, testbench configurations, and comprehensive timing analyses for each subsystem.

---

## 2. 6T SRAM Cell Design

The fundamental building block of the memory array is the 6-Transistor (6T) SRAM cell, capable of storing a single bit of binary information.

### Circuit Explanation
The circuit topology consists of two cross-coupled CMOS inverters forming a positive feedback loop. This configuration creates a bistable latch capable of maintaining two stable states (Logic 0 or Logic 1). The storage nodes, denoted as Q and Qbar, are always complements of each other. These nodes are accessed via two NMOS pass-transistors (Access Transistors), controlled by the **Word Line** (**WL**).

![Schematic of the 6T SRAM Cell.](SRAM/SRAM_circuit.png)

**Operational Modes:**
1. **Hold Mode:** When the **Word Line (WL)** is Low (0V), the access transistors are cut off (OFF state). The internal latch is physically isolated from the bit lines. The cross-coupled inverters reinforce each other's state, maintaining the stored data indefinitely as long as power is applied.
2. **Read/Write Mode:** When **WL** is High (1.8V), the access transistors turn on. This creates a low-impedance path between the bit lines (**BL** and **BLbar**) and the internal storage nodes. This connection allows external circuits to either sense the stored voltage (Read) or force a new voltage level (Write).

---

## 3. SRAM Write Operation Analysis

The write operation is a "force" operation. The internal cross-coupled inverters naturally resist any change in state. To overwrite the data, the external Write Drivers must be significantly stronger than the internal pull-up transistors of the SRAM cell.

### Write Logic '1'
To write a logic '1' into a cell currently storing '0', we must raise the voltage of node Q to $V_{DD}$ and lower Qbar to GND. This is achieved by driving the external bit line **BL** to 1.8V and **BLbar** to 0V.

![Testbench Setup for Write '1' Operation.](SRAM/SRAM_final_testbench_write1.png)

![Waveform results for Write '1'.](SRAM/SRAM_timing_diagram_write1.png)

**Analysis of Waveforms:**
1. **Setup:** Initially, the **Word Line** (Red trace) is Low, and the cell is in Hold mode.
2. **Driver Activation:** The **Write Driver** is activated, driving **BL** to High (Purple trace) and **BLbar** to Low (Cyan trace). At this stage, the cell is still isolated.
3. **Access & Overpowering:** At 2.0ns, the **Word Line** is asserted High. The access transistor connects the strong 0V on **BLbar** to the internal node Qbar.
4. **State Transition:** The external driver pulls node Qbar (Pink trace) down to 0V, overpowering the internal weak PMOS. Due to the cross-coupling, as Qbar drops, it turns ON the PMOS on the Q side, helping pull node Q (Green trace) up to 1.8V.
5. **Completion:** The feedback loop is now locked in the new state, storing a '1'.

### Write Logic '0'
To write a logic '0', the polarity is reversed. We must force node Q to 0V and node Qbar to 1.8V.

![Testbench Setup for Write '0' Operation.](SRAM/SRAM_final_testbench_write0.png)

![Waveform results for Write '0'.](SRAM/SRAM_timing_diagram_write0.png)

**Analysis of Waveforms:**
1. **Initialization:** The inputs are configured with **BL** Low and **BLbar** High.
2. **Discharge Path:** Upon **WL** activation, the access transistor connecting **BL** to node Q turns on. Since **BL** is held strongly at 0V by the write driver, it acts as a sink.
3. **Flipping the Latch:** The charge on node Q (Cyan trace) is drained through the access transistor. Simultaneously, the High voltage on **BLbar** charges node Qbar (Pink trace).
4. **Result:** Node Q transitions to 0V, and Qbar transitions to 1.8V. The cell has successfully flipped its state to store a '0'.

---

## 4. SRAM Read Operation Analysis

The read operation is delicate because it must be non-destructive. Unlike the write operation where we overpower the cell, during a read, the cell must be strong enough to influence the bit lines without flipping its own state (Read Stability).

### Read Logic '1'
Condition: The cell is storing a '1' (Q=1, Qbar=0).

![Testbench Setup for Read '1'.](SRAM/SRAM_final_testbench_read1.png)

![Waveform results for Read '1'.](SRAM/SRAM_timing_diagram_read1.png)

**Operational Logic:**
1. **Pre-Charge Phase:** Before the read begins, both **BL** and **BLbar** are pre-charged to exactly $V_{DD}$ (1.8V). This equalization is crucial to prevent offsets.
2. **Access:** When **WL** goes High, the cell is connected to the floating bit lines.
3. **Discharge Mechanism:** Since the internal node Qbar is Low (0V), the access transistor on that side creates a path from **BLbar** to ground.
4. **Sensing:** The voltage on **BLbar** (Cyan trace) begins to drop slowly as it discharges through the small SRAM cell transistors. Meanwhile, **BL** (Purple trace) stays at 1.8V because it connects to the High node Q.
5. **Differential Development:** This process creates a voltage difference ($\Delta V$) between the two bit lines. The Sense Amplifier will detect this split to interpret a logic '1'.

### Read Logic '0'
Condition: The cell is storing a '0' (Q=0, Qbar=1).

![Testbench Setup for Read '0'.](SRAM/SRAM_final_testbench_read0.png)

![Waveform results for Read '0'.](SRAM/SRAM_timing_diagram_read0.png)

**Operational Logic:**
1. **Pre-Charge:** Both bit lines start at 1.8V.
2. **Discharge Mechanism:** Upon access, node Q is at 0V. Therefore, the current flows from the **BL** bit line into the cell, discharging **BL**.
3. **Signal Generation:** **BL** (Cyan trace) voltage drops, while **BLbar** (Red trace) maintains its charge because Qbar is High.
4. **Detection:** The drop in **BL** relative to **BLbar** indicates to the system that a '0' is stored.

---

## 5. Static Noise Margin (SNM) Analysis

### Objective
To determine the read stability of the SRAM bit-cell by extracting the Static Noise Margin (SNM) from the voltage transfer characteristics (VTC) of the cross-coupled inverters. The SNM is the maximum noise voltage that can be tolerated by the SRAM cell without flipping its state.

### Methodology
The SNM was calculated by finding the side length of the largest square that fits within the "eyes" of the butterfly curve. This was achieved mathematically using a coordinate rotation method to measure the maximum diagonal width of the lobes, effectively transforming the coordinate system to align with the VTC diagonal.

### Formula Used
The stability at any given voltage point was calculated using the following expression, effectively rotating the coordinate system by $45^{\circ}$ to find the perpendicular distance between curves:

$$
SNM = \frac{1}{\sqrt{2}} \times |V_{out} - V_{inverted}|
$$

In the Cadence Calculator, this is implemented as:  
`abs((Vout - Vmirror) * 0.707)`

### Testbench
![Testbench for Stability Analysis.](SRAM_butterfly_curve/testbench.png)

### Graphical Analysis

<img width="844" height="709" alt="butterfly curve v2" src="https://github.com/user-attachments/assets/ffe0ff62-3ad7-4e90-b5a8-13d67b145fff" />


**Butterfly Curve:** This figure displays the superimposed DC transfer characteristics of the two inverters. Two stable states (lobes) are visible, representing logic '1' and logic '0' retention. Visual inspection reveals an asymmetry between the pull-up and pull-down networks, resulting in unequal eye openings. The "eye" represents the safe margin for operation; a larger eye implies a more robust cell.

<img width="646" height="736" alt="batman curve" src="https://github.com/user-attachments/assets/62dfebbe-baf2-4cfa-ad6a-1210b5c9ba49" />



**SNM "Batman" Plot:** This is a graphical representation of the eye width across the voltage sweep. The peaks of this plot correspond directly to the SNM of each stable state. The height of the "wings" in this plot indicates the numeric value of the stability margin.

### Simulation Results
* **Left Lobe Stability (Eye 1):** 627.5 mV
* **Right Lobe Stability (Eye 2):** 381.1 mV

### Conclusion
The overall Static Noise Margin is determined by the weakest link in the cell. Therefore, the final SNM is **381.1 mV**. The disparity between the two lobes suggests a sizing imbalance in the bit-cell transistors (specifically the pull-up to pull-down ratio), which determines the cell's immunity to noise during read operations. However, 381.1 mV is generally considered a sufficient margin for 90nm technology.

---

## 6. Pre-Charge Circuit

The **Pre-Charge Circuit** is a vital support block. Before any read operation, the bit lines are essentially large capacitors that may hold residual voltages from previous cycles. The Pre-Charge circuit resets these lines to a known high voltage ($V_{DD}$).

### Circuit and Testbench
![Pre-Charge Circuit Schematic.](Pre%20Charge/Pre_Charge_circuit.png)

![Pre-Charge Testbench.](Pre%20Charge/Pre_Charge_testbench.png)

### Timing Analysis
![Pre-Charge Timing Diagram.](Pre%20Charge/Pre_Charge_timing_diagram.png)

![Current Trend during Pre-Charge.](Pre%20Charge/Pre_Charge_i_trend.png)

**Detailed Analysis:**
1. **Activation:** The circuit utilizes 3 PMOS transistors. The Red trace indicates the Pre-Charge Enable signal (active low). PMOS transistors are used because they pass a strong logic '1' (1.8V).
2. **Charging Phase:** When the enable signal drops to 0V, the PMOS transistors activate, creating a low-resistance path from the power supply to the Bit Lines.
3. **Rapid Rise:** The Bit Lines (Cyan and Green traces) charge rapidly to 1.8V. The speed of this charge determines the maximum frequency of the memory.
4. **Equalization:** The third transistor (the equalizer) connects **BL** directly to **BLbar**. This shorts the two lines together, ensuring that even if there is a slight manufacturing mismatch, the voltage difference between the lines is exactly zero before the read cycle begins.

---

## 7. Sense Amplifier

The **Sense Amplifier** is a differential sensing circuit designed to accelerate read operations. Since the SRAM cell has limited drive strength, discharging the heavy bit line capacitance fully to 0V would take a long time (nanoseconds). The Sense Amplifier detects a very small voltage difference (approx. 100mV - 200mV) and amplifies it to full-swing digital logic levels.

### Circuit and Testbench
![Sense Amplifier Schematic.](Sense%20Amplifier/Sense_Amplifier_timing_circuit.png)

![Sense Amplifier Testbench.](Sense%20Amplifier/Sense_Amplifier_tb.png)

### Timing Analysis
![Sense Amplifier Response (Reading '1').](Sense%20Amplifier/Sense_Amplifier_timing_diagram_read1.png)

![Sense Amplifier Response (Reading '0').](Sense%20Amplifier/Sense_Amplifier_timing_diagram_read0.png)

**Operation Analysis:**
1. **Input Development:** The SRAM cell slowly discharges one bit line, creating a small voltage difference ($\Delta V$) at the inputs (Red and Green lines).
2. **Activation:** The Orange trace represents the "Sense Enable" (**SE**) signal. This signal must be timed perfectly: if enabled too early, it reads noise; if too late, speed is lost.
3. **Positive Feedback:** Upon SE activation, the internal latch of the sense amplifier engages. The positive feedback loop amplifies the small $\Delta V$ exponentially.
4. **Full Swing Output:** The outputs (Cyan and Purple) snap to 1.8V and 0V almost instantly. This converts the analog differential voltage into a clean digital "1" or "0".

---

## 8. Write Driver

The **Write Driver** acts as a high-drive strength buffer. While the Sense Amplifier listens, the Write Driver commands. Its primary function is to override the weak internal feedback of the SRAM cell during write operations, ensuring data is successfully latched regardless of the previous state.

### Circuit and Testbench
![Write Driver Schematic.](Write%20Driver/Write_Driver_circuit.png)

![Write Driver Testbench.](Write%20Driver/Write_Driver_tb.png)

### Timing Analysis
![Write Driver Output Waveforms.](Write%20Driver/Write_Driver_timing_diagram.png)

**Functional Analysis:**
1. **Input Logic:** The Top Red trace shows the Data Input toggling between high and low states.
2. **Write Enable Control:** The Green trace is the "Write Enable" (**WE**) signal. The driver is active only when this signal is High.
3. **Driving the Lines:** When **WE** is High, the output inverters drive the Bit Lines (Bottom Cyan and Purple) to 1.8V and 0V. The driver transistors are sized much larger than the SRAM cell transistors to ensure they can force the bit line voltage quickly.
4. **High-Impedance (Hi-Z) State:** When **WE** is Low, the driver enters a high-impedance state (floating). This disconnects the driver from the bit lines, ensuring it does not short out the Sense Amplifier during a read operation.

---

## 9. Top Level Design (4x4 Array)

The final integration combines the SRAM Cell Array, Pre-Charge Circuitry, Sense Amplifiers, and Write Drivers into a 4x4 memory grid. This verifies that all components work together in a realistic timing sequence.

### Circuit Diagram
![Schematic of the complete 4x4 SRAM Array.](Top%20Design/Top_Design_circuit.png)

### Full System Timing Analysis
A "Full Cycle" simulation was performed to validate the system. The sequence is: **Write Operation** $\rightarrow$ **Pre-Charge** $\rightarrow$ **Read Operation**.

![Testbench setup for Top Design Read/Write '1'.](Top%20Design/Top_Design_tb_readwrite1.png)

![Full System Timing: Write '1' followed by Read '1'.](Top%20Design/Top_Design_timing_diagram_readwrite1.png)

**Simulation Cycle (Write 1 / Read 1):**
1. **Write Cycle (0 - 10ns):** The Write Enable signal is asserted. The Write Driver forces the selected cell's internal node Q (Cyan trace) to transition from Low to High.
2. **Pre-Charge (10ns - 15ns):** The Pre-Charge signal goes Low, resetting both bit lines to 1.8V. This prepares the array for the next operation.
3. **Read Cycle (15ns - 25ns):** The Word Line is activated. The Sense Amplifier detects the bit line differential.
4. **Verification:** The Sense Amplifier output (Purple/Pink trace) transitions High, matching the logic level written in step 1. This confirms successful storage and retrieval.

![Testbench setup for Top Design Read/Write '0'.](Top%20Design/Top_Design_tb_readwrite0.png)

![Full System Timing: Write '0' followed by Read '0'.](Top%20Design/Top_Design_timing_diagram_readwrite0.png)

**Simulation Cycle (Write 0 / Read 0):**
1. **Write Cycle:** Write Enable is High with Data input '0'. The internal node Q discharges to 0V.
2. **Read Cycle:** After the pre-charge interval, the Word Line opens for the read operation.
3. **Verification:** The Sense Amplifier output remains Low, correctly identifying the stored '0'. The stability of the output confirms that the read operation did not disturb the stored data (Read Stability).

---

## Conclusion
This project report has detailed the design and analysis of a 16-bit SRAM array implemented in 90nm CMOS technology. The 6T cell was sized to ensure adequate Static Noise Margin (SNM) of 381.1 mV, providing robustness against noise. The peripheral circuits were verified for functionality: the Pre-Charge circuit effectively equalizes bit lines to prevent offsets, the Sense Amplifier provides rapid differential sensing to speed up access times, and the Write Driver ensures reliable write margins by overpowering the cell. The integrated 4x4 array simulation demonstrates correct write and read functionality with stable timing, meeting the project objectives for high-speed reliable memory design.
