# 5-Stage Pipelined RV32I RISC-V Processor Core

This repository contains the RTL (Verilog) implementation of a 32-bit RISC-V processor featuring a classical 5-stage pipeline architecture. The design supports a subset of the RV32I Base Integer Instruction Set and includes a robust Hazard Unit to manage data dependencies via hardware forwarding.

## 🚀 Key Features

* **Architecture:** 32-bit RISC-V (RV32I ISA).
* **Pipeline Stages:** 1. Instruction Fetch (IF)
  2. Instruction Decode (ID)
  3. Execute (EX)
  4. Memory Access (MEM)
  5. Writeback (WB)
* **Hazard Management:** Integrated Hazard Unit for solving Read-After-Write (RAW) data hazards using Forwarding/Bypassing paths from the MEM and WB stages to the EX stage.
* **Control Hazards:** Branch target calculation and condition evaluation performed in the Execute stage.
* **Simulation:** Verified using Icarus Verilog and GTKWave.

---

## 🏗️ Pipeline Architecture

### Abstract Pipeline Stages
*(Reference: Slide 5)*
![Abstract Pipeline Architecture](images/page5.png)

### Basic Pipeline Datapath
*(Reference: Slide 6)*
![Pipeline Datapath](images/page6.png)

### Complete Datapath with Hazard Unit and Forwarding
*(Reference: Slide 32)*
![Datapath with Hazard Unit](images/page32.png)

---

## 📁 Repository Structure

* **Top Level Module:**
  * `Pipeline_Top.v`: The main wrapper stitching all 5 pipeline stages and the hazard unit together.

* **Pipeline Stages:**
  * `Fetch_Cycle.v`: Handles PC updates and fetches instructions from Memory.
  * `Decode_Cyle.v`: Houses the Control Unit, Register File, and Immediate Sign Extension.
  * `Execute_Cycle.v`: Contains the ALU, branch target calculation, and forwarding multiplexers.
  * `Memory_Cycle.v`: Handles Data Memory Read/Write operations.
  * `Writeback_Cycle.v`: Selects between Memory Data and ALU Result to write back to the Register File.

* **Core Components:**
  * `ALU.v`, `ALU_Decoder.v`, `Main_Decoder.v`, `Control_Unit_Top.v`
  * `Register_File.v`, `Instruction_Memory.v`, `Data_Memory.v`, `Sign_Extend.v`
  * `PC.v`, `PC_Adder.v`, `Mux.v`
  * `Hazard_unit.v`: Detects data hazards and generates `ForwardAE` and `ForwardBE` signals.

* **Simulation:**
  * `pipeline_tb.v`: Testbench for the pipeline.
  * `memfile.hex`: Hex file containing RISC-V machine code for initialization.
  * `pipeline.gtkw`: GTKWave save file for viewing waveforms.

---

## ⚙️ Hazard Unit & Forwarding Logic

To prevent pipeline stalls due to Data Hazards (specifically RAW dependencies), this core employs hardware bypassing. The `Hazard_unit.v` continuously monitors the destination registers of the instructions currently in the **MEM** and **WB** stages, comparing them against the source registers of the instruction in the **EX** stage.

**Forwarding Conditions:**
* **ForwardA/B = 2'b00:** Default. ALU operands come directly from the ID/EX pipeline register (Register File).
* **ForwardA/B = 2'b10:** EX/MEM Forwarding. The ALU operand is bypassed from the previous ALU result (Instruction currently in MEM stage).
* **ForwardA/B = 2'b01:** MEM/WB Forwarding. The ALU operand is bypassed from the Data Memory output or an earlier ALU result (Instruction currently in WB stage).

1. **Compile the design:**
   ```bash
   iverilog -o pipeline_sim pipeline_tb.v
