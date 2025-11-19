# 🔑 Digital Door Lock System (Zynq UltraScale+)

This project implements a **Digital Door Lock** with a **Finite State Machine (FSM)** controller designed for the **Zynq UltraScale+ MPSoC** platform using **Vivado 2024.1**.

### ✨ Features
* **4-Digit Code Entry:** Managed by a Finite State Machine written in Verilog.
* **Secure Logic:** Transitions between IDLE, PENDING, UNLOCKED, and ALARM states.
* **Platform Integration:** Utilizes the Zynq Processing System (PS) and AXI interfaces as shown in the Block Design (`MPoSC_ext_platform.bd`).
* **Visual Feedback:** Provides `unlocked` and `alarm` status signals, connected to LEDs.

### 🛠️ Hardware/Software Requirements
* **EDA Tool:** Xilinx Vivado (2024.1 or later)
* **Target Device:** Zynq UltraScale+ MPSoC
* **HDL:** Verilog

### 🚀 Setup Instructions
1.  **Clone the Repository:**
    ```bash
    git clone [https://github.com/your-username/DigitalDoorLock.git](https://github.com/your-username/DigitalDoorLock.git)
    cd DigitalDoorLock
    ```
2.  **Open Vivado:**
    * Open Vivado 2024.1.
    * Recreate the project using the source files in `src/hdl/` and the constraints in `src/constraints/`. Alternatively, you can copy your existing Vivado project folder (`vivado/`) into this directory.
3.  **Synthesize and Implement:**
    * Ensure the Block Design (`MPoSC_ext_platform.bd`) is properly included and validated.
    * Run Synthesis, Implementation, and **Generate Bitstream** for your target board.

### 🔍 File Structure
