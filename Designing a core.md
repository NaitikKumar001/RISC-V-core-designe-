p# What is smallest core

 If we talk about smallest and simplest core 
 it doesn't have any pipelining,not a heavy 
 Architecture,and have less component unlike 
 lbex

 -A simple core may contain these components 
 ```text
 Pc
 Instruction Fetch
 Decoder
 Control Unit
 Immediate genrater
 Register file
 ALU
 LSU
 write back
 Pc update
```
# Support system 
This core Support follow type.
```text
R-TYPE- ADD,SUB,AND,OR,XOR
I-TYPE- ADDI,ANDI,ORI,XORI
S-TYPE- SW
B-TYPE- BEQ
J-TYPE- JAL
```
It only Support RV32I Extension 

```text
1] PC---》INSTRUCTION MEMORY
    tells which instruction address need

2] INSTRUCTION MEMORY---》FETCH
    Instruction Fetched

3] FETCH---》DECODE
    Decode decodes the instruction
    ex:-add x3,x8,x2 opcode = add
                        rs1 = x8
                        rs2 = x2
                         rd = x3

4] DECODER---》 CONTROL UNIT
    control unit tell ALU which operation
    perform

   DECODER---》 REGISTER FILE

   Decoder———》IMMEDIATE GENRATER———》ALU
    If any immediate value present then is
    in 12 bit so,immediate genrater extend
    it according to ALU bit

5] ALU———》perform all the operation

6] ALU ———》LSU
    It send the result to LSU
7] LSU ———》 DATA MEMORY

8] LSU ———》WRITE BACK ———》REGISTER FILE
    result go back in write back using MUX

9] PC UPDATE ———》 PC
   Pc update get control signals about
   next_address then
 if it is normal ——》 pc+4
 if it in branch
            |__ true—》target address
            |
            |__ false—》pc+4
```

# How Pc and Pc update works?
Pc update gives the next instruction to PC
- In case of normal instruction like add x5,x7,x2
  next_pc= pc+4

-But if branch/jump instructions are used 
then
```text
            FETCH
              |
.           DECODE
              |
          EXICUTE [check branch condition] 
              |
        ——————————————————
       |                  |
      YES.                NO
   then calculate.      next_pc = pc+4
   target address
target address = pc+offset

OFFSET——》it tells 'current address sa               kitna aaga ya peecha jaana hai'










  

