# RTL

**RTL → Register Transfer Level**

RTL describes the working of hardware at the register-transfer level.

Architecture tells us **what components are present** in the hardware, for example:
- ALU
- Registers
- Register File
- PC
- MUX

But RTL tells us **how these components work and how data moves between them**.

For example:

- Data transfer from Register → ALU
- Data transfer from ALU → Register
- PC value update
- Register File write operation
- MUX selection
- ALU result calculation

In simple words:

> **RTL describes what happens inside the hardware and how data transfers from one component to another.**

---

# What RTL Describes

RTL mainly describes:

- PC kaise update hoga?
- ALU ka result kaise calculate hoga?
- Register File kab write karega?
- Data Register se ALU tak kaise jayega?
- ALU se Register tak data kaise jayega?
- MUX ka selection kaise hoga?
- Clock ke according data kab transfer hoga?

So:

> **RTL mainly describes the data flow and operations between registers and hardware components.**

---

# RTL Language

RTL is not a single programming language.

RTL can be written using **Hardware Description Languages (HDLs)** such as:

- Verilog
- SystemVerilog
- VHDL

For example, SystemVerilog can be used to describe RTL for a CPU core.

---

# RTL Example

For example, a simple register transfer:

```systemverilog
always_ff @(posedge clk) begin
    q<=d;
  end
```
```Meaning
-》On every rising clock edge the value of
'd' store in 'q'

-》Here it explain the transfer data behavior
   between 'd' and 'q' in SystemVerilog.
   this is the actual work of RTL
```

# In Sequential logic
  current Output depends upon privious output 
  If we write in SystemVerilog 
  ```text
  always _ff @(posedge clk) begin
        pc<= next_pc;
  end
```
```Meaning
  cycle1 , PC = 1000
  cycle2 , PC = 1004
  cycle3 , PC = 1008
```
# In Combinational Logic 

 Output depends upon current input 
 so in SystemVerilog 
 ```
  always _comb begin
     result = a+b
  end
```
```Mwaning
  According to current input calculate
  result continuously
