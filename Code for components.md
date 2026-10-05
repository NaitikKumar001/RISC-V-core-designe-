# Code to Design a Core

All the hardware code is written in **SystemVerilog**.

There are two main parts:

1. `design.sv` — contains the actual hardware design.
2. `testbench.sv` — tests and verifies the hardware design.

---

# 1. design.sv

The `design.sv` file contains the actual hardware modules used to build the CPU core.

For our basic RISC-V core, the design can be divided into different modules:

# 2. testbench.sv 
 The `testbench.sv` file used to test code written 
 in design.sv with appropriate values 

 # Code to create ALU 
 1] For`design.sv`
 
 ``` text
  module alu (
    input  logic [31:0] a,
    input  logic [31:0] b,
    input  logic [3:0]  alu_op,
    output logic [31:0] result
);

always_comb begin
    case (alu_op)
        4'b0000: result = a + b;                 // ADD
        4'b0001: result = a - b;                 // SUB
        4'b0010: result = a << b[4:0];           // SLL
        4'b0011: result = ($signed(a) < $signed(b)) ? 1 : 0; // SLT
        4'b0100: result = (a < b) ? 1 : 0;       // SLTU
        4'b0101: result = a ^ b;                 // XOR
        4'b0110: result = a >> b[4:0];           // SRL
        4'b0111: result = $signed(a) >>> b[4:0]; // SRA
        4'b1000: result = a | b;                 // OR
        4'b1001: result = a & b;                 // AND
        default: result = 32'b0;
    endcase
end

endmodule
```
2]For `testbench.sv`
``` text 
  module alu_tb;

    logic [31:0] a;
    logic [31:0] b;
    logic [3:0]  alu_op;
    logic [31:0] result;

    // Connect ALU
    alu dut (
        .a(a),
        .b(b),
        .alu_op(alu_op),
        .result(result)
    );

    initial begin

        // Input values
        a = 32'd10;
        b = 32'd3;

        // ADD
        alu_op = 4'b0000;
        #10;
        $display("ADD  : %0d + %0d = %0d", a, b, result);

        // SUB
        alu_op = 4'b0001;
        #10;
        $display("SUB  : %0d - %0d = %0d", a, b, result);

        // SLL
        alu_op = 4'b0010;
        #10;
        $display("SLL  : %0d << %0d = %0d", a, b, result);

        // SLT
        alu_op = 4'b0011;
        #10;
        $display("SLT  : %0d < %0d = %0d", a, b, result);

        // SLTU
        alu_op = 4'b0100;
        #10;
        $display("SLTU : %0d < %0d = %0d", a, b, result);

        // XOR
        alu_op = 4'b0101;
        #10;
        $display("XOR  : %0d ^ %0d = %0d", a, b, result);

        // SRL
        alu_op = 4'b0110;
        #10;
        $display("SRL  : %0d >> %0d = %0d", a, b, result);

        // SRA
        alu_op = 4'b0111;
        #10;
        $display("SRA  : %0d >>> %0d = %0d", a, b, result);

        // OR
        alu_op = 4'b1000;
        #10;
        $display("OR   : %0d | %0d = %0d", a, b, result);

        // AND
        alu_op = 4'b1001;
        #10;
        $display("AND  : %0d & %0d = %0d", a, b, result);

        $finish;
    end

endmodule
```
3] Expected output 
```text 
ADD  : 10 + 3 = 13
SUB  : 10 - 3 = 7
SLL  : 10 << 3 = 80
SLT  : 10 < 3 = 0
SLTU : 10 < 3 = 0
XOR  : 10 ^ 3 = 9
SRL  : 10 >> 3 = 1
SRA  : 10 >>> 3 = 1
OR   : 10 | 3 = 11
AND  : 10 & 3 = 2
```
# Code to create PC
1] For `design.sv`
```text 
module program_counter (
    input  logic        clk,
    input  logic        reset,
    input  logic [31:0] next_pc,
    output logic [31:0] pc
);

    always_ff @(posedge clk) begin
        if (reset)
            pc <= 32'h00000000;
        else
            pc <= next_pc;
    end

endmodule
```
2]For`testbench.sv`
```text 
module program_counter_tb;

    logic        clk;
    logic        reset;
    logic [31:0] next_pc;
    logic [31:0] pc;

    program_counter dut (
        .clk(clk),
        .reset(reset),
        .next_pc(next_pc),
        .pc(pc)
    );

    always #5 clk = ~clk;

    initial begin

        clk = 0;
        reset = 1;
        next_pc = 32'h00000000;

        #10;
        reset = 0;

        next_pc = 32'h00000004;
        #10;

        next_pc = 32'h00000008;
        #10;

        next_pc = 32'h0000000C;
        #10;

        $finish;
    end

endmodule
```

# Explanation 
```text
-clk → clock signal; PC updates on every rising edge.
-reset → puts PC back to 0.
-pc → current instruction address.
-next_pc → address that PC will take on the next clock cycle.
-always_ff @(posedge clk) → PC is a

-sequential element, so it changes on the clock edge.

Same as we can give hardware description of other components using SystemVerilog HDL
```





