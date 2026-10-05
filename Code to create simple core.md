# code of simple core
This core supports only R-TYPE instructions 
this core code also includes codes of different
components like:
```text
ALU
REGISTER FILE
INSTRUCTION MEMORY
PC
DECODER
```
1]For`design.sv`
```text
   // ============================================================
// RV32I R-TYPE ONLY CPU CORE
//
// Supported:
// ADD, SUB, SLL, SLT, SLTU,
// XOR, SRL, SRA, OR, AND
// ============================================================


// ============================================================
// ALU
// ============================================================

module alu (
    input [31:0] a,
    input [31:0] b,
    input [3:0]  alu_op,
    output reg [31:0] result
);

    always @(*) begin

        case (alu_op)

            4'b0000: result = a + b;                 // ADD
            4'b0001: result = a - b;                 // SUB
            4'b0010: result = a << b[4:0];           // SLL

            4'b0011:
                result = ($signed(a) < $signed(b))
                       ? 32'd1 : 32'd0;              // SLT

            4'b0100:
                result = (a < b)
                       ? 32'd1 : 32'd0;              // SLTU

            4'b0101: result = a ^ b;                 // XOR
            4'b0110: result = a >> b[4:0];           // SRL
            4'b0111: result = $signed(a) >>> b[4:0]; // SRA
            4'b1000: result = a | b;                 // OR
            4'b1001: result = a & b;                 // AND

            default: result = 32'b0;

        endcase

    end

endmodule



// ============================================================
// REGISTER FILE
//
// 32 registers
// Each register = 32 bits
// x0 is always 0
// ============================================================

module register_file (
    input clk,
    input reset,

    input [4:0] rs1,
    input [4:0] rs2,

    input [4:0] rd,
    input [31:0] write_data,
    input write_enable,

    output [31:0] rs1_data,
    output [31:0] rs2_data
);

    reg [31:0] registers [0:31];

    integer i;


    // READ PORT 1
    assign rs1_data =
        (rs1 == 5'd0)
        ? 32'd0
        : registers[rs1];


    // READ PORT 2
    assign rs2_data =
        (rs2 == 5'd0)
        ? 32'd0
        : registers[rs2];


    // WRITE PORT
    always @(posedge clk) begin

        if (reset) begin

            for (i = 0; i < 32; i = i + 1)
                registers[i] <= 32'd0;

        end

        else begin

            if (write_enable && (rd != 5'd0))
                registers[rd] <= write_data;

            // x0 always remains zero
            registers[0] <= 32'd0;

        end

    end

endmodule



// ============================================================
// R-TYPE CPU CORE
// ============================================================

module rtype_core (
    input clk,
    input reset
);

    // ========================================================
    // PROGRAM COUNTER
    // ========================================================

    reg [31:0] pc;


    // ========================================================
    // INSTRUCTION
    // ========================================================

    wire [31:0] instruction;


    // ========================================================
    // INSTRUCTION MEMORY
    // ========================================================

    reg [31:0] instruction_memory [0:31];

    assign instruction =
        instruction_memory[pc[6:2]];


    // ========================================================
    // INSTRUCTION FIELDS
    // ========================================================

    wire [6:0] opcode;
    wire [4:0] rd;
    wire [4:0] rs1;
    wire [4:0] rs2;
    wire [2:0] funct3;
    wire [6:0] funct7;

    assign opcode = instruction[6:0];
    assign rd     = instruction[11:7];
    assign funct3 = instruction[14:12];
    assign rs1    = instruction[19:15];
    assign rs2    = instruction[24:20];
    assign funct7 = instruction[31:25];


    // ========================================================
    // REGISTER FILE SIGNALS
    // ========================================================

    wire [31:0] rs1_data;
    wire [31:0] rs2_data;

    wire [31:0] write_data;

    reg write_enable;


    // ========================================================
    // ALU SIGNALS
    // ========================================================

    reg [3:0] alu_op;

    wire [31:0] alu_result;


    // ========================================================
    // REGISTER FILE
    // ========================================================

    register_file regs (

        .clk(clk),
        .reset(reset),

        .rs1(rs1),
        .rs2(rs2),

        .rd(rd),

        .write_data(write_data),
        .write_enable(write_enable),

        .rs1_data(rs1_data),
        .rs2_data(rs2_data)

    );


    // ========================================================
    // ALU
    // ========================================================

    alu my_alu (

        .a(rs1_data),
        .b(rs2_data),

        .alu_op(alu_op),

        .result(alu_result)

    );


    // ========================================================
    // WRITE BACK
    // ========================================================

    assign write_data = alu_result;


    // ========================================================
    // R-TYPE DECODER
    // ========================================================

    always @(*) begin

        // Default values
        alu_op = 4'b0000;
        write_enable = 1'b0;


        // R-Type opcode = 0110011
        if (opcode == 7'b0110011) begin

            write_enable = 1'b1;

            case (funct3)

                // ADD / SUB
                3'b000: begin

                    if (funct7 == 7'b0100000)
                        alu_op = 4'b0001;       // SUB
                    else
                        alu_op = 4'b0000;       // ADD

                end


                // SLL
                3'b001:
                    alu_op = 4'b0010;


                // SLT
                3'b010:
                    alu_op = 4'b0011;


                // SLTU
                3'b011:
                    alu_op = 4'b0100;


                // XOR
                3'b100:
                    alu_op = 4'b0101;


                // SRL / SRA
                3'b101: begin

                    if (funct7 == 7'b0100000)
                        alu_op = 4'b0111;       // SRA
                    else
                        alu_op = 4'b0110;       // SRL

                end


                // OR
                3'b110:
                    alu_op = 4'b1000;


                // AND
                3'b111:
                    alu_op = 4'b1001;


                default:
                    alu_op = 4'b0000;

            endcase

        end

    end


    // ========================================================
    // PROGRAM COUNTER
    // ========================================================

    always @(posedge clk) begin

        if (reset)
            pc <= 32'd0;

        else
            pc <= pc + 32'd4;

    end

endmodule
```
2] For `testbench.sv`
 ``` text
  // ============================================================
// TESTBENCH FOR RV32I R-TYPE CPU
// ============================================================

module tb;

    reg clk;
    reg reset;


    // ========================================================
    // CREATE CPU
    // ========================================================

    rtype_core dut (

        .clk(clk),
        .reset(reset)

    );


    // ========================================================
    // CLOCK GENERATION
    // ========================================================

    initial begin

        clk = 1'b0;

        forever #5 clk = ~clk;

    end


    // ========================================================
    // TEST PROGRAM
    // ========================================================

    initial begin

        // ----------------------------------------------------
        // ADD x3, x1, x2
        // x3 = 10 + 3 = 13
        // ----------------------------------------------------

        dut.instruction_memory[0] =
            32'b0000000_00010_00001_000_00011_0110011;


        // ----------------------------------------------------
        // SUB x4, x1, x2
        // x4 = 10 - 3 = 7
        // ----------------------------------------------------

        dut.instruction_memory[1] =
            32'b0100000_00010_00001_000_00100_0110011;


        // ----------------------------------------------------
        // SLL x5, x1, x2
        // x5 = 10 << 3 = 80
        // ----------------------------------------------------

        dut.instruction_memory[2] =
            32'b0000000_00010_00001_001_00101_0110011;


        // ----------------------------------------------------
        // SLT x6, x1, x2
        // 10 < 3 = false
        // x6 = 0
        // ----------------------------------------------------

        dut.instruction_memory[3] =
            32'b0000000_00010_00001_010_00110_0110011;


        // ----------------------------------------------------
        // SLTU x7, x1, x2
        // ----------------------------------------------------

        dut.instruction_memory[4] =
            32'b0000000_00010_00001_011_00111_0110011;


        // ----------------------------------------------------
        // XOR x8, x1, x2
        // 10 XOR 3 = 9
        // ----------------------------------------------------

        dut.instruction_memory[5] =
            32'b0000000_00010_00001_100_01000_0110011;


        // ----------------------------------------------------
        // SRL x9, x1, x2
        // 10 >> 3 = 1
        // ----------------------------------------------------

        dut.instruction_memory[6] =
            32'b0000000_00010_00001_101_01001_0110011;


        // ----------------------------------------------------
        // SRA x10, x1, x2
        // ----------------------------------------------------

        dut.instruction_memory[7] =
            32'b0100000_00010_00001_101_01010_0110011;


        // ----------------------------------------------------
        // OR x11, x1, x2
        // 10 OR 3 = 11
        // ----------------------------------------------------

        dut.instruction_memory[8] =
            32'b0000000_00010_00001_110_01011_0110011;


        // ----------------------------------------------------
        // AND x12, x1, x2
        // 10 AND 3 = 2
        // ----------------------------------------------------

        dut.instruction_memory[9] =
            32'b0000000_00010_00001_111_01100_0110011;


        // ====================================================
        // RESET
        // ====================================================

        reset = 1'b1;

        #10;

        reset = 1'b0;


        // ====================================================
        // INITIAL REGISTER VALUES
        //
        // Simulation ke liye manually values de rahe hain.
        // ====================================================

        dut.regs.registers[1] = 32'd10;
        dut.regs.registers[2] = 32'd3;


        // ====================================================
        // RUN CPU
        // ====================================================

        #110;


        // ====================================================
        // DISPLAY RESULTS
        // ====================================================

        $display("");
        $display("========================================");
        $display("       RV32I R-TYPE CPU TEST");
        $display("========================================");

        $display("x1  = %0d", dut.regs.registers[1]);
        $display("x2  = %0d", dut.regs.registers[2]);

        $display("ADD result  x3  = %0d",
                 dut.regs.registers[3]);

        $display("SUB result  x4  = %0d",
                 dut.regs.registers[4]);

        $display("SLL result  x5  = %0d",
                 dut.regs.registers[5]);

        $display("SLT result  x6  = %0d",
                 dut.regs.registers[6]);

        $display("SLTU result x7  = %0d",
                 dut.regs.registers[7]);

        $display("XOR result  x8  = %0d",
                 dut.regs.registers[8]);

        $display("SRL result  x9  = %0d",
                 dut.regs.registers[9]);

        $display("SRA result  x10 = %0d",
                 dut.regs.registers[10]);

        $display("OR result   x11 = %0d",
                 dut.regs.registers[11]);

        $display("AND result  x12 = %0d",
                 dut.regs.registers[12]);

        $display("========================================");


        #10;

        $finish;

    end

endmodule
```
3]Expected output
```text
  ========================================
       RV32I R-TYPE CPU TEST
========================================
x1  = 10
x2  = 3
ADD result  x3  = 13
SUB result  x4  = 7
SLL result  x5  = 80
SLT result  x6  = 0
SLTU result x7  = 0
XOR result  x8  = 9
SRL result  x9  = 1
SRA result  x10 = 1
OR result   x11 = 11
AND result  x12 = 2
========================================
```
# Simulater

A **simulator** is a software tool that models the behavior of a hardware design on a computer without physically building the hardware.

In SystemVerilog, a simulator runs the **design code** together with the **testbench** and shows how the hardware behaves over time.

---

## Why Do We Need a Simulator?

A simulator is used to:

- Test a hardware design
- Verify that the design works correctly
- Find bugs and design errors
- Observe internal signals
- Check outputs for different inputs
- Analyze timing and signal changes
- View waveforms

This allows us to test a CPU or other digital circuit before implementing it on real hardware.

---

## Design + Testbench + Simulator

The basic flow is:

```text
        design.sv
            +
       testbench.sv
            |
            v
       +-----------+
       | Simulator |
       +-----------+
            |
            v
     Simulation Results
            |
       +----+----+
       |         |
    Output    Waveform
```
