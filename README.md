## Design and Performance Analysis of a 6T SRAM Cell and Peripherals in 90nm CMOS Technology

## Project Overview
**Project Report**  
**Technology:** GPDK 90nm CMOS  
**Tools:** Cadence Virtuoso

---

## Table of Contents
1. [Introduction](#introduction)
2. [6T SRAM Cell Design](#6t-sram-cell-design)
3. [Write Operation Analysis](#write-operation-analysis)
4. [Read Operation Analysis](#read-operation-analysis)
5. [Stability Analysis (Butterfly Curve)](#stability-analysis-butterfly-curve)
6. [Peripheral Circuits](#peripheral-circuits)
7. [Top Design: 4x4 Memory Array](#top-design-4x4-memory-array)
8. [Conclusion](#conclusion)

---

## Introduction

Static Random Access Memory (SRAM) is essentially the "muscle memory" of modern computing. It is used wherever speed is the absolute priority, such as in the Cache Memory (L1, L2, L3) of a CPU. The word "Static" means that unlike its cheaper cousin, Dynamic RAM (DRAM), SRAM does not need to be constantly refreshed to keep its data. As long as power is supplied, the data stays locked in.

While DRAM uses a single capacitor to store charge (which acts like a leaky bucket), SRAM uses a team of six transistors to form a latch. This makes it significantly faster and more robust, but also more expensive in terms of silicon area.

### Project Objective
The main goal of this project is to build a complete 16-bit (4x4) SRAM memory array from scratch. We are not just designing the storage cell; we are building the entire support system that makes it work. The project is divided into four key stages:
1.  **The Core:** Design and simulation of the 6T SRAM Cell (The storage unit).
2.  **The Health Check:** Analysis of Read and Write Stability (Butterfly Curves).
3.  **The Support Crew:** Design of Peripheral Circuits (Pre-Charge, Sense Amp, Write Driver).
4.  **Integration:** Connecting everything into a fully functional 4x4 Array.

---

## 6T SRAM Cell Design

The fundamental building block of this memory array is the 6-Transistor (6T) SRAM cell. This is where the single bit of binary information (Logic 0 or Logic 1) actually lives.

### Circuit Explanation
The circuit uses four transistors to form two cross-coupled inverters. Think of these as two people holding hands in a circle—if one lets go, the loop breaks. This loop creates a "Latch" that locks the data in a stable state. The remaining two transistors are the "Access Gates" (or doors) that connect this private loop to the outside world (the Bit Lines) only when we ask them to.

<img width="1292" height="675" alt="SRAM_circuit" src="https://github.com/user-attachments/assets/d9118ab7-3b8b-476d-a0bc-6d5d1ba4b52b" />


**How it works:**
*   **Hold Mode (Storage):** When the Word Line (WL) is Low (0V), the doors are closed. The internal latch is isolated from the noisy outside world. It sits there reinforcing its own data indefinitely.
*   **Read/Write Mode (Access):** When WL is High (1.8V), the doors open. The Bit Lines (BL and BLbar) can now "talk" to the internal storage nodes (Q and Qbar).

---

## Write Operation Analysis

Writing to an SRAM cell is an act of brute force. The internal latch wants to keep its old value. To change it, we use powerful external "Write Drivers" to overpower the internal transistors and force them to flip.

### Write Logic '1'
To write a '1', we need to force the internal node Q to High and Qbar to Low. We do this by setting the external Bit Line (BL) to a strong 1.8V and the Bit Line Bar (BLbar) to a strong 0V.

**Testbench Setup:**
<img width="812" height="647" alt="SRAM_final_testbench_write1" src="https://github.com/user-attachments/assets/f37ff547-7c35-4a46-9186-be219c17f4b5" />


**Timing Analysis:**
<img width="1896" height="693" alt="SRAM_timing_diagram_write1" src="https://github.com/user-attachments/assets/8dd808f0-b1c7-4894-bcea-7ffcbd8c341a" />


**The Write "1" Sequence:**
*   **The Setup:** Initially, the Word Line (Red trace) is Low. The cell is closed.
*   **The Force:** The Write Driver sets BL to High (Purple) and BLbar to Low (Cyan). They are waiting at the door.
*   **The Breach:** At 2.0ns, the Word Line goes High (Red pulse). The doors open.
*   **The Flip:** Look at the internal nodes Q (Green) and Qbar (Pink). Even though they might have been holding a '0' before, the strong external lines force Q up to 1.8V and Qbar down to 0V.
*   **Success:** The lines cross, and the cell now firmly holds a '1'.

### Write Logic '0'
To write a '0', we simply reverse the attack. We force the Bit Line (BL) to 0V and the Bit Line Bar (BLbar) to 1.8V.

**Testbench Setup:**
![Write 0 Testbench](SRAM/SRAM_final_testbench_write0.png)
<img width="785" height="608" alt="SRAM_final_testbench_write0" src="https://github.com/user-attachments/assets/5e209400-8f46-4a6f-a143-e7a72dc87043" />

**Timing Analysis:**
<img width="1912" height="700" alt="SRAM_timing_diagram_write0" src="https://github.com/user-attachments/assets/fc9167d8-c6e0-42f5-aba9-d7666285b03c" />


**The Write "0" Sequence:**
*   The inputs are flipped: BL is Low and BLbar is High.
*   When the Word Line (WL) opens the door, the strong 0V on BL drains the charge out of node Q.
*   Simultaneously, the 1.8V on BLbar rushes into node Qbar.
*   **The Result:** Q (Cyan trace) crashes down to 0V, and Qbar (Pink trace) shoots up to 1.8V.
*   The cell has been successfully overwritten to store a '0'.

---

## Read Operation Analysis

Reading is much more delicate than writing. In writing, we "shouted" at the cell. In reading, we have to "listen" to it. We pre-charge the lines to full voltage, disconnect the power, and then let the tiny SRAM cell gently pull one of the lines down.

### Read Logic '1'
The cell is holding a '1' (Q=1, Qbar=0).

**Testbench Setup:**
<img width="855" height="658" alt="SRAM_final_testbench_read1" src="https://github.com/user-attachments/assets/7ea448b6-597a-4952-aeed-7bb140fc2a3b" />


**Timing Analysis:**
<img width="1908" height="690" alt="SRAM_timing_diagram_read1" src="https://github.com/user-attachments/assets/3ab9af32-54c6-4adb-ac12-f6e05f8c1815" />


**The Logic of the Read:**
1.  **Pre-Charge:** Both BL and BLbar start at a full 1.8V (High). Imagine two full buckets of water.
2.  **Access:** When WL (Red) goes High, the cell connects to these buckets.
3.  **The Action:** Since the cell stores a '1', the internal Qbar side is at 0V (Ground).
4.  **The Discharge:** The 0V at Qbar acts like a drain hole. It starts sucking the charge out of the BLbar line.
5.  **The Evidence:** Look at the Cyan trace (BLbar). It starts drooping down. Meanwhile, the Purple trace (BL) stays perfectly full because the Q side is High.
6.  **Conclusion:** This splitting of the lines (one stays high, one drops) is how the system knows a '1' is stored.

### Read Logic '0'
The cell holds a '0' (Q=0, Qbar=1).

**Testbench Setup:**
<img width="852" height="682" alt="SRAM_final_testbench_read0" src="https://github.com/user-attachments/assets/c0138c4a-ef4e-4223-974f-ec7b15aa15a0" />


**Timing Analysis:**
<img width="1915" height="696" alt="SRAM_timing_diagram_read0" src="https://github.com/user-attachments/assets/55e0f202-259d-43de-9299-0789f23f4851" />


**The Logic of the Read:**
1.  Again, both lines start full (1.8V).
2.  When WL opens the door, the cell connects.
3.  This time, since Q is 0V (Low), it is the Q side that acts as the drain.
4.  **The Evidence:** In the waveform, you can see the Cyan trace (BL) dropping. The Red trace (BLbar) stays High.
5.  Because the *main* Bit Line dropped, the system knows a '0' was stored.

---

## Stability Analysis (Butterfly Curve)

How do we know the cell won't accidentally flip when we try to read it? We measure this using the **Static Noise Margin (SNM)**, visualized as the "Butterfly Curve."

**Testbench Setup:**
<img width="2564" height="1164" alt="testbench" src="https://github.com/user-attachments/assets/85b80302-c826-4f27-8622-7b0913c91c0d" />


**Butterfly Curve Result:**
<img width="3488" height="1216" alt="buterfly curve" src="https://github.com/user-attachments/assets/b651f074-6248-4813-8c1c-08dc862e67ab" />
<img width="3840" height="1338" alt="timing diagram" src="https://github.com/user-attachments/assets/ddd14288-1e64-4c96-aa6b-c55aced3b8a3" />



**Reading the Butterfly:**
This graph plots the strength of the left inverter against the strength of the right inverter.
*   The "Wings" are the voltage transfer curves.
*   The **"Eyes"** (the square openings in the middle) represent the stability.
*   **The Rule:** The larger the square that fits inside these eyes, the more stable the cell is. If the eyes were closed or small, a tiny bit of electrical noise could corrupt our data. Our wide, open eyes indicate a very robust design.

---

## Peripheral Circuits

### Pre-Charge Circuit
The Pre-Charge circuit is the "Cleaning Crew" of the memory system. Before we can read any data, we need the Bit Lines to be in a known state (1.8V). If we left them at random voltages from the last operation, we would get garbage data.

**Schematic:**
<img width="832" height="677" alt="Pre_Charge_circuit" src="https://github.com/user-attachments/assets/9c231e0e-515e-4346-9893-8808e6f56ff8" />


**Testbench:**
<img width="820" height="657" alt="Pre_Charge_testbench" src="https://github.com/user-attachments/assets/434c401b-6c79-4df9-bc85-aa22032c26ff" />


**Timing Analysis:**
<img width="1911" height="692" alt="Pre_Charge_timing_diagram" src="https://github.com/user-attachments/assets/7e978857-0751-44ea-a96d-e486628beae6" />


**How it works:**
1.  **The Reset Signal:** In the timing diagram, the Red trace is the Pre-Charge Enable signal.
2.  **The Action:** When this signal drops to 0V, the transistors turn ON. It connects the Bit Lines directly to the power supply.
3.  **The Result:** Look at the Bit Lines (Cyan and Green). They might start at 0V (empty), but the moment the Pre-Charge hits, they shoot up to 1.8V.

### Sense Amplifier
The Sense Amplifier is the "Spy" of the operation. The SRAM cell is weak; it takes a long time to pull a Bit Line all the way to 0V. We don't want to wait that long. The Sense Amp detects a *tiny* voltage difference (like 0.2V) and instantly amplifies it to a full digital "0" or "1".

**Schematic:**
<img width="1528" height="688" alt="Sense_Amplifier_timing_circuit" src="https://github.com/user-attachments/assets/27da8c62-a25e-422f-aba3-839e37c15028" />


**Testbench:**
<img width="845" height="642" alt="Sense_Amplifier_tb" src="https://github.com/user-attachments/assets/3cb94ec9-79fc-45d8-a209-c29096575dff" />


**Timing Analysis (Read 1 & Read 0):**
<img width="1917" height="695" alt="Sense_Amplifier_timing_diagram_read1" src="https://github.com/user-attachments/assets/e5ad3e50-507f-43c6-a083-13a0c0cbe6b6" />

<img width="1917" height="688" alt="Sense_Amplifier_timing_diagram_read0" src="https://github.com/user-attachments/assets/4692cb3f-7e9b-4f16-a5df-293663437dfb" />


**The Decision Moment:**
1.  **The Race:** Look at the inputs (Red and Green lines). They start falling, but one falls slightly faster. This gap is the "data."
2.  **The Trigger:** The Orange trace is the "Sense Enable" (SE) signal. At this moment, the Sense Amp wakes up.
3.  **The Split:** As soon as SE goes High, the amplifier catches that tiny gap and rips the lines apart.
4.  **The Output:** Look at the Cyan and Purple lines. One shoots to 1.8V, the other crashes to 0V. This proves the amplifier can take a "whisper" of a signal and turn it into a clear digital decision.

### Write Driver
The Write Driver is the "Bully." The SRAM cell tries to hold onto its data with a feedback loop. The Write Driver is designed with huge transistors to simply overpower that loop and force new data in.

**Schematic:**
<img width="966" height="653" alt="Write_Driver_circuit" src="https://github.com/user-attachments/assets/d27e3a1a-12ea-4ad0-be48-581382894c97" />


**Testbench:**
<img width="1300" height="524" alt="Write_Driver_tb" src="https://github.com/user-attachments/assets/f9d1336a-66b3-4ffb-984e-7837d5da48a1" />


**Timing Analysis:**
<img width="1918" height="701" alt="Write_Driver_timing_diagram" src="https://github.com/user-attachments/assets/4a49e665-1570-405a-b64a-cc7c0c68a47f" />


**How the Bully works:**
1.  **The Command:** The Top Red trace is the Data Input toggling between 0 and 1.
2.  **The Permission:** The Green trace is "Write Enable" (WE). The driver only acts when this is High.
3.  **The Force:** When WE is High, look at the Bit Lines (Bottom Cyan and Purple). They follow the Data perfectly and snap to 1.8V or 0V instantly.

---

## Top Design: 4x4 Memory Array

This is the Final Boss. We connect the Cells, the Cleaning Crew (Pre-Charge), the Spy (Sense Amp), and the Bully (Write Driver) into a complete 4-row by 4-column memory grid.

**Top Level Schematic:**
<img width="1057" height="664" alt="Top_Design_circuit" src="https://github.com/user-attachments/assets/7d9373ad-7569-4f6b-b266-211e17bc7fd6" />


### Full System Timing Analysis
We ran a "Full Cycle" simulation: Write a value -> Reset -> Read the value back.

**Write '1' / Read '1' Cycle:**
<img width="2774" height="1268" alt="Top_Design_tb_readwrite1" src="https://github.com/user-attachments/assets/8582cabf-c282-49ce-a853-56e6a34a36ee" />
<img width="3820" height="1327" alt="Top_Design_timing_diagram_readwrite1" src="https://github.com/user-attachments/assets/6ea3cfc5-54bc-445d-8cab-e565aca4b2f0" />


**The Story of the Simulation (Write 1 / Read 1):**
1.  **Write Cycle (First Half):** The Write Enable signal goes High. The Driver blasts data into the selected cell. You can see the internal node Q (Cyan trace) flipping to the new value.
2.  **Pre-Charge (The Gap):** The system resets. The lines charge up to 1.8V.
3.  **Read Cycle (Second Half):** The Write Driver stays quiet. The Word Line opens. The Sense Amp fires.
4.  **The Victory:** Look at the Sense Amplifier output (Purple/Pink traces). It transitions to the exact same logic level we wrote in step 1.

**Write '0' / Read '0' Cycle:**
<img width="2760" height="1212" alt="Top_Design_tb_readwrite0" src="https://github.com/user-attachments/assets/d1680159-bc3f-4026-a70c-f0944825422b" />

<img width="2760" height="1212" alt="Top_Design_tb_readwrite0" src="https://github.com/user-attachments/assets/3c1be2c7-4924-4ff6-8055-e76e857990a6" />


**The Story of the Simulation (Write 0 / Read 0):**
1.  **Write Cycle:** The Write Enable goes High with Data set to 0. The internal node Q drops to 0V.
2.  **Read Cycle:** After pre-charging, the Word Line opens. The Sense Amplifier detects the low value stored in the cell.
3.  **Confirmation:** The output stays Low, confirming that the '0' was successfully stored and retrieved.

---

## Conclusion

This project report detailed the design and analysis of a 16-bit SRAM array in 180nm CMOS technology. We successfully designed the 6T cell with proper sizing for stability. We verified that the Pre-Charge circuit cleans the lines, the Sense Amplifier detects weak signals, and the Write Driver successfully overwrites data. Finally, the integration of these blocks into a 4x4 array was simulated, demonstrating correct Write and Read operations with stable timing margins. The design meets the objectives of high-speed and reliable data storage.
