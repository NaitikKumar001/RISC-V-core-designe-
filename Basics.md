# What happens?

 when I write a program what happens
 
  
          program
             |
     compiler/assembler
             |
    RISC-V Machine Instruction
             |
         CPU core
             |
          Result=a
      


 Basically what happens inside a core. 
 generally all the cotes are doing roughly
 same thing.
      
          Instruction Memory 
                  |
                Fetch
                  |
                Decode
                  |
                Execute
                  |
              write back
      

so,If we talk about Ibex.
- It is an open source core which is based  on RISC-V ISA
- It is based on two type of extensions
[1] RV32I- 32 Bit integer type Instruction
           It has 40 base instructions
[2] RV32E- 32 Bit embedded tpe Instruction
           It has 16 base Instruction

-Ibex has 2 stage pipeline,heavy 
 Architecture, more component
-Ibex have these components 
```text
pc
prefetch buffer
ALU
Decoder
Register File
compressed decoder
LSU
CSR
Controller
Mult/div
Mux
```
