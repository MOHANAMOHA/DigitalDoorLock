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
// Digital_Door_Lock_verilog_files.v
// All Verilog source files for the Digital Door Lock project
// Included modules:
//  - top_module
//  - password_rom
//  - fsm_controller
//  - status_display
//
// Image references (snapshots from your Vivado project):
//  - /mnt/data/Screenshot 2025-11-19 135610.png
//  - /mnt/data/Screenshot 2025-11-19 135738.png
//  - /mnt/data/Screenshot 2025-11-19 135756.png
//
// You can copy each module into its own .v file if desired.


// =============================================================
// top_module.v
// Top-level wrapper that instantiates FSM, ROM and display
// =============================================================
module top_module (
    input clk,
    input rst,
    input [3:0] input_digit,
    input enter,        // pulse when a digit is entered
    output [6:0] led    // 7-seg style output (MSB..LSB)
);
    wire unlocked, alarm;

    // Small address/idx bus used by FSM to read ROM
    wire [1:0] rom_addr;
    wire [3:0] rom_data;

    // Instantiate password ROM (4-entry ROM)
    password_rom rom (
        .addr(rom_addr),
        .data(rom_data)
    );

    // Instantiate FSM controller
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

    // Instantiate display driver
    status_display display (
        .unlocked(unlocked),
        .alarm(alarm),
        .led(led)
    );
endmodule


// =============================================================
// password_rom.v
// 2-bit address -> 4-bit digit data
// =============================================================
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


// =============================================================
// fsm_controller.v
// Simple synchronous FSM to check a 4-digit password against ROM
// Behavior:
//  - On 'enter' pulse: capture input_digit and compare to ROM at current index
//  - If match, increment index; if index reaches CODE_LEN -> unlocked
//  - On mismatch -> alarm asserted and index frozen until reset
//  - rst clears state and index
// Notes: This is a compact, clear behavioral reference implementation
// =============================================================
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
    // Parameters
    localparam CODE_LEN = 4;

    // index tracks which digit we are checking (0..CODE_LEN-1)
    reg [1:0] index;

    // simple edge-detect for enter (assumes enter is a single-cycle pulse from PS or debounced button)
    // If enter may be multi-cycle, ensure external debouncing or implement pulse stretching here.

    always @(posedge clk or posedge rst) begin
        if (rst) begin
            index <= 2'd0;
            rom_addr <= 2'd0;
            unlocked <= 1'b0;
            alarm <= 1'b0;
        end else begin
            if (unlocked || alarm) begin
                // Once in terminal state (unlocked or alarm), ignore further enters until reset
                rom_addr <= rom_addr; // hold
                index <= index;
            end else if (enter) begin
                // Compare input with ROM data at current address
                if (input_digit == rom_data) begin
                    // correct digit
                    if (index == (CODE_LEN-1)) begin
                        unlocked <= 1'b1;
                        index <= index; // final
                        rom_addr <= rom_addr; // final
                    end else begin
                        index <= index + 1'b1;
                        rom_addr <= rom_addr + 1'b1;
                    end
                end else begin
                    // incorrect digit -> alarm
                    alarm <= 1'b1;
                end
            end else begin
                // no enter pulse: hold
                index <= index;
                rom_addr <= rom_addr;
            end
        end
    end
endmodule


// =============================================================
// status_display.v
// Maps unlocked/alarm signals to a 7-bit LED / 7-seg-like bus
// Simple mapping used for demonstration; change patterns as desired
// =============================================================
module status_display (
    input unlocked,
    input alarm,
    output reg [6:0] led
);
    always @(*) begin
        if (unlocked) begin
            // Show "U" pattern (example) when unlocked
            // 7-bit segments [6:0] arbitrary mapping for your board
            led = 7'b0111110; // example pattern
        end else if (alarm) begin
            // Show "A" or blinking pattern for alarm
            led = 7'b1000001; // example pattern
        end else begin
            // idle: all segments off
            led = 7'b0000000;
        end
    end
endmodule


// End of file
