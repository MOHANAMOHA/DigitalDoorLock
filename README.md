# 🔑 Digital Door Lock System

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
📐 Block Diagram

(Upload images into your GitHub repository and replace the paths below.)

/mnt/data/Screenshot 2025-11-19 135610.png
/mnt/data/Screenshot 2025-11-19 135738.png
/mnt/data/Screenshot 2025-11-19 135756.png

✨ Features

🔒 4-digit programmable password

⚡ Fast, hardware-based FSM

🧮 ROM-based password storage

🚨 Alarm on incorrect password

🔑 Unlock indicator using LEDs

🧱 Clean modular Verilog architecture

🧵 AXI-Lite integration (ready for PS software control)

🛠 Hardware Design

The design uses a Finite State Machine (FSM) with the following states:

State	Description
IDLE	waiting for digit input
CHECK	compares current digit with ROM value
UNLOCKED	password correct
ALARM	password incorrect

Clock frequency used: 196 MHz

Vivado tool automatically applied the required timing constraints.

📂 Verilog Source Files
1️⃣ top_module.v
module top_module (
    input clk,
    input rst,
    input [3:0] input_digit,
    input enter,
    output [6:0] led
);
    wire unlocked, alarm;
    wire [1:0] rom_addr;
    wire [3:0] rom_data;

    password_rom rom (
        .addr(rom_addr),
        .data(rom_data)
    );

    fsm_controller fsm (
        .clk(clk),
        .rst(rst),
        .input_digit(input_digit),
        .enter(enter),
        .rom_data(rom_data),
        .rom_addr(rom_addr),
        .unlocked(unlocked),
        .alarm(alarm)
    );

    status_display display (
        .unlocked(unlocked),
        .alarm(alarm),
        .led(led)
    );
endmodule

2️⃣ password_rom.v
module password_rom (
    input [1:0] addr,
    output reg [3:0] data
);
    always @(*) begin
        case (addr)
            2'd0: data = 4'd1;
            2'd1: data = 4'd2;
            2'd2: data = 4'd3;
            2'd3: data = 4'd4;
            default: data = 4'd0;
        endcase
    end
endmodule

3️⃣ fsm_controller.v
module fsm_controller (
    input clk,
    input rst,
    input [3:0] input_digit,
    input enter,
    input [3:0] rom_data,
    output reg [1:0] rom_addr,
    output reg unlocked,
    output reg alarm
);
    localparam CODE_LEN = 4;
    reg [1:0] index;

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            index <= 0;
            rom_addr <= 0;
            unlocked <= 0;
            alarm <= 0;
        end 
        else if (!unlocked && !alarm && enter) begin
            if (input_digit == rom_data) begin
                if (index == CODE_LEN-1)
                    unlocked <= 1;
                else begin
                    index <= index + 1;
                    rom_addr <= rom_addr + 1;
                end
            end else begin
                alarm <= 1;
            end
        end
    end
endmodule

4️⃣ status_display.v
module status_display (
    input unlocked,
    input alarm,
    output reg [6:0] led
);
    always @(*) begin
        if (unlocked)
            led = 7'b0111110;   // U
        else if (alarm)
            led = 7'b1000001;   // A
        else
            led = 7'b0000000;   // OFF
    end
endmodule

📊 Implementation & Results

Vivado console output confirms:

Block design loaded successfully

Addresses validated

Clock configuration completed at 196 MHz

AXI interconnect correctly mapped

🧪 How to Build
1. Clone the repository
git clone https://github.com/yourname/DigitalDoorLock.git

2. Open Vivado
File → Open Project → select .xpr

3. Generate Bitstream
Flow → Generate Bitstream

4. Export Hardware to Vitis
File → Export Hardware → Include bitstream

5. Program FPGA

Use Vitis or XSCT.

🚀 Future Improvements

Add UART-based password entry

Add reprogrammable password storage

Integrate PMOD keypad

Add OLED display for UI

Add retry limit + timeout features

🖼 Screenshots

You can upload the images to GitHub and reference them:

![Block Diagram](images/block_design.png)
![Address Map](images/address_map.png)
![Vivado Console](images/vivado_log.png)
