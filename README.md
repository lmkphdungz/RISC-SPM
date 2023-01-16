# RISC-SPM
>A final project of the course "Digital Design using VHDL"
## **Overview**
Reduced instruction-set computers (RISC) are designed to have a small set of instructions that execute in short clock cycles, with a small number of cycles per instruction. RISC machines are optimized to achieve efficient pipelining of their instruction streams. The machine also serves as a starting point for developing architectural variants and a more robust instruction set. Designers make high-level tradeoffs in selecting an architecture that serves an
application. Once an architecture has been selected, a circuit that has sufficient performance (speed) must be synthesized. Hardware description languages (HDLs) play a key role in this process by modeling the system and serving as a descriptive medium
that can be used by a synthesis tool.

RISC-SPM or Reduced Instruction Set Computer Store Program Machine consists of three functional units :
1. Processing Unit ( Processor )
2. Controll Unit ( Controller )
3. Memory Unit ( RAM )

Program instructions and data are stored in memory

Instructions are fetched from memory synchronously, decoded and executed to : 
- Operate on data with ALU
- Change the contents of storage registers
- Change the content of the program counter (PC), instruction register (IR) and the address register (ADD_R)
- Change the content of memory
- Retrieve data and instructions from memory
- Control the movement of data on the system busses

The Program Counter (PC) contains the address of the next instruction to be executed

The Instruction Register (IR) contains the instruction that currently being executed

The address register (Add_R) contains the address of the memory location that will be addressed next by a read or write operation.

The Overall Architecture of a simple RISC-SPM is shown below

![Architecture RISC-SPM](https://github.com/canh25xp/RISC-SPM/blob/main/assets/RISC-SPM.png)

## **Processing Unit**
The processor includes *registers*, *buses*, *control lines*, and an *ALU* capable of performing arithmetic and logic operations on its operands depends on the opcode held in the instruction register.

There are 2 multiplexers in the Processing Unit : 
- Mux_1 : it's a 5-1 multiplexer
    - Output : Bus_1
    - Input : R0, R1, R2, R3, PC
- Mux_2 : it's a 3-1 multiplexer
    - Output : Bus_2
    - Input : ALU's output, Bus_1

An instruction can be fetched from memory, placed on Bus_2, and loaded into the instruction register. A word of data can be fetched from memory, and steered to a general-purpose register or to the operand register (Reg_Y) prior to an operation of the ALU. The result of an ALU operation can be placed on Bus_2, loaded into a register, and subsequently transferred to memory. A dedicated register (Reg_Z) holds a flag indicating that the result of an ALU operation is 0.

## **Arithmetic Logic Unit**
For the purposes of this example, the ALU has two operand datapaths, data_1 and data_2, and its instruction set is limited to only 4 instructions, that is :
Opcode|Action
-|-
ADD | Adds the datapaths to form data_1 + data_2
SUB | Subtracts the datapaths to form data_1 - data_2
AND | Takes the bitwise and of the datapaths data_1 & data_2
NOT | Takes the bitwise Boolean complement of data_1

## **Control Unit**
### Functions of the control unit:
1. Determine when to load registers
2. Select the path of data through the multiplexers
3. Determine when data should be written to memory
4. Control the three-state busses in the architecture.

### Control Signals :
Control Signal|Action
-|-
Load_Add_R|Loads the address register
Load_PC|Loads Bus_2to the program counter
Load_IR|Loads Bus_2 to the instruction register
Inc_PC|Increments the program counter
Sel_Bus_1_Mux | Selects among the Program_Counter, R0, R1, R2, and R3 to drive Bus_1
Sel_Bus_2_Mux|Selects among Alu_out, Bus_1, and memory to drive Bus_2
Load_R0|Loads general purpose register R0
Load_R1|Loads general purpose register R1
Load_R2|Loads general purpose register R2
Load_R3|Loads general purpose register R3
Load_Reg_Y|Loads Bus_2 to the register Reg_Y
Load_Reg_Z|Stores output of ALU in register Reg_Z
write|Loads Bus_1 into the SRAM memory

### Instruction Set
A machine language program consists of a stored sequence of 8-bit words (bytes). The format of an instruction of RISC_SPM can be long or short, depending on the operation : 
- Short instructions : requires 1 byte of memory to specifies 4-bit opcode, 2-bit source register address, a 2-bit destination register address.
- Long instruction : requires 2 bytes of memory. The first word of a long instruction contains a 4-bit opcode. The remaining 4 bits of the word can be used to specify addresses of a pair of source and destination registers, depending on the instruction. The second word contains the address of the memory word that holds an operand required by the instruction

Opcode|Src|Dst
-|-|-
0  0 1 0|0 1|1 0

Opcode|Src|Dst|Address
-|-|-|-
0  1 1 0|1 0|x x|0 0 0 1 1 1 0 1

The instruction mnemonics and their actions are listed below.

Short instruction|Action
-|-
NOP|No operation is performed; all registers retain their values. The addresses of the source and destination register are don't-cares, they have no effect.
ADD|Adds the contents of the source and destination registers and stores the result into the destination register.
SUB|Subtracts the content of the source register from the destination register and stores the result into the destination register.
AND|Forms the bitwise-and of the contents of the source and destination registers and stores the result into the destination register.
NOT|Forms the bitwise complement of the content of the source register and stores the result into the destination register.
HALT|Halts execution until reset

Long instruction|Action
-|-
RD|Fetches a memory word from the location specified by the second byte and loads the result into the destination register. The source register bits are don't-cares (i.e. unused).
WR|Writes the contents of the source register to the word in memory specified by the address held in the second byte. The destination register bits are don't-cares (i.e. unused).
BR|Branches the activity flow by loading the program counter with the word at the location (address) specified by the second byte of the instruction. The source and destination bits are don't-cares (i.e. unused).
BRZ|Branches the activity flow by loading the program counter with the word at the location (address) specified by the second byte of the instruction if the zero flag register is asserted.

The RISC_SPM instruction set is summarized below.

Ins|Opcode|Src|Dst|Action
-|-|-|-|-
NOP|0000|xx|xx|none
ADD|0001|src|dst|dst <= src + dst
SUB|0010|src|dst|dst <= dst - src
AND|0011|src|dst|dst <= src && dst
Nor|0100|src|dst|dst <= ~ src
RD|0101|xx|dst|dst <= memory [Add_R]
WR|0110|src|xx|memory[Add_R] < = src
BR|0111|xx|xx|PC<= memory[Add_R]
BRZ|1000|xx|xx|PC<= memory[Add_R]
HALT|1111|xx|xx|Halts execution until reset


### Controller States
Three phases of operation: fetch, decode, and execute.
- Fetching: Retrieves an instruction from memory (2 clock cycles)
- Decoding: Decodes the instruction, manipulates datapaths,and loads registers (1 clock cycle)
- Execution: Generates the results of the instruction (0, 1,or 2 clock cycles)

State|Action
-|-
idle|State entered after reset is asserted. No action.
fet1|Load the Add_R with the contents of the PC. (Note: PC is initialized to the starting address by the reset action.) The state is entered at the first active clock after reset is de-asserted, and is revisited after a NOP instruction is decoded.
fet2|Load the IR with the word addressed by the Add_R, and increment the program counter to point to the next location in memory, in anticipation of tlie next instruction or data fetch.
dec|Decode the IR and assert signals to control datapaths and register transfers.
ex1|Execute the ALU operation for a single-byte instruction, conditionally assert the zero flag, and load the destination register.
rd1|Load the Add_R with the second byte of a RD instruction, and increment the PC.
rd2|Load the destination register with the memory word addressed by the byte loaded in rd1.
wr1|Load the Add_R with the second byte of a WR instruction, and increment the PC.
wr2|Load the destination register with the memory word addressed by the byte loaded in wr1.
br1|Load the Add_R with the second byte of a BR instruction, and increment the PC.
br2|Load the program counter with the memory word addressed by the byte loaded in br1 .
halt|Default state to trap failure to decode a valid instruction.

### Controller ASM
...

## **Memory Unit**
For simplicity, the memory unit of the machine is modeled as an array of D flip-flops that form a **256 bytes** RAM 

## **Testbench**
To ensure the working of the machine, each module has it own testbench : Memory Unit, Control Unit, Register Unit, Arithmetic Logic Unit.