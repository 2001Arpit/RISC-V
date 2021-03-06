# RISC-V Microprocessor 
# Overview
Creating a RISC-V pipelined core, which has support of base integer RV32I instruction format using TL-Verilog on makerchip platform.
# Table of Contents
- [Introduction](#introduction)
- [GNU compiler toolchain](#gnu-compiler-toolchain)
- [Application Binary Interface](#application-binary-interface)
- [Digital Logic in TL-Verilog Using Makerchip](#digital-logic-in-tl-verilog-using-makerchip)
  - [Combinational Logic](#combinational-logic)
  - [Sequential Logic](#sequential-logic)
  - [Pipelined Logic](#pipelined-logic)
  - [Validity](#validity)
  - [Single Value Memory](#single-value-memory)
- [Basic RISC-V CPU Micro-Architecture](#basic-risc-v-cpu-micro-architecture)
  - [Fetch](#fetch)
  - [Decode](#decode)
  - [Register File Read and Write](#register-file-read-and-write)
  - [ALU](#alu)
  - [Branch Instructions](#branch-instructions)
- [Pipelining the RISC V CPU](#pipelining-the-risc-v-cpu)
  - [Hazards](#hazards) 
  - [3 Cycle Valid Signal](#3-cycle-valid-signal)
  - [Register File Bypass](#register-file-bypass)
  - [Load and Store Instructions](#load-and-store-instructions)
  - [Finalised RISC V CPU](#finalised-risc-v-cpu)
- [Acknowledgements](#acknowledgements)
    
# Introduction
![flow](https://user-images.githubusercontent.com/92947276/144841759-7f171938-2f64-4411-b059-5686dcbd872d.PNG)

Whenever you run an app on your computer, its function (high level program) is converted to its respective assembly language program using a compiler. It is then converted into machine level code using an assembler. Finally, this machine code (also known as a bitstream) is fed into the chip layout, which will perform its function accordingly.

This project aims to design a RISC V microprocessor core and understand its various functions. We will start with a simple c program then look into its assembly and machine language code. Before that, let us see some basic commands we need to know to implement this in a Linux based system.

# GNU Compiler Toolchain
* To view and edit your program file use the following command

    `leafpad <filename> &`
    Leafpad is a texteditor.

* To compile your program use the following command:
    
    `gcc <filename>`
    
* To view the output of the program use the following command:
    `./a.out`

* To use the risc-v gcc compiler use the following command:

    `riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o <object filename> <C filename>`

    More generic command with different options:

    `riscv64-unknown-elf-gcc <compiler option -O1 ; Ofast> <ABI specifier -lp64; -lp32; -ilp32> <architecture specifier -RV64 ; RV32> -o <object filename> <C      filename>`

    More details on compiler options can be obtained [here](https://www.sifive.com/blog/all-aboard-part-1-compiler-args)
  
 * To view assembly code use the below command,
    
    `riscv64-unknown-elf-objdump -d <object filename>`
    
  * To use SPIKE simualtor to run risc-v obj file use the below command,
  
    `spike pk <object filename>`
    
    To use SPIKE as debugger
    
    `spike -d pk <object Filename>` with degub command as `until pc 0 <pc of your choice>`
    
We will see the following commands in action with a program to compute the sum of numbers 1 to 9.

![sum_n code2](https://user-images.githubusercontent.com/92947276/144846309-21de62b2-0a00-459a-a3e1-227aa5a7439f.PNG)

Output:

![sum_n op](https://user-images.githubusercontent.com/92947276/144847376-d6e4fdee-c395-4e82-b39f-c81efdda95c6.PNG)

# Application Binary Interface

* The application program can directly access the registers of the RISC V architecture using something known as system calls. The ABI (also known as system call interface
  enables the application to access the hardware resources via registers.
* In RISC V architecture, the width of the register is defined as XLEN. For RV64 and RV32, the widths are 64 bits and 32 bits, respectively.
* RISC V belongs to the little endian memory addressing system, which means that the least significant byte of a word is stored in the smallest memory address.

![abi](https://user-images.githubusercontent.com/92947276/144853808-3407d29a-f9de-41e4-beeb-2459acdc7d5f.png)

* The following c program computes the sum of numbers 1 to 9. We also have its equivalent assembly language code.
* To get the sum replace value of int count with 9.

![1 to 9 code](https://user-images.githubusercontent.com/92947276/144854779-70b773b7-6ed4-4397-8765-0624619e06d0.PNG)

![load code](https://user-images.githubusercontent.com/92947276/144855226-7869f7ac-050c-42bd-88e2-c76f2728f3ff.PNG)

* Compile them using the gcc compiler comand mentioned above.
* The assembly code can be viewed using command mentioned above.
* `main` function will look as follows:

![main](https://user-images.githubusercontent.com/92947276/144857556-64d9b4c5-c059-4339-807a-9c6fe2546033.PNG)

* `load` function will look as follows:

![load](https://user-images.githubusercontent.com/92947276/144857691-ab31ecf4-76ef-4b48-aaa8-d814beb3c964.PNG)

* Output can viewed using spike:

![simulate 1 y 9 code](https://user-images.githubusercontent.com/92947276/144857923-ae4bc980-54d7-4525-8f96-1f43a1a8baa8.PNG)

* We can also view the hex file containing the bitstream to be fed into the chip layout.

![hexfile bitstream](https://user-images.githubusercontent.com/92947276/144858357-386c6fe6-80c6-48cd-ab1b-df2bc39e21cf.PNG)

* This hex file is then added to the test bench for our Verilog code.

![hexfile testbench](https://user-images.githubusercontent.com/92947276/144858694-45867cc7-b4fe-4c9a-9063-42cff2c2bf58.PNG)


# Digital Logic in TL-Verilog Using Makerchip  

* Makerchip is a  free online IDE for developing in Verilog or TL-Verilog. It can be used directly in your browser. You can code, simulate and view block diagrams and waveforms.
* It also features multiple examples and templates.

## Combinational Logic

* Here is the code for a combinational calculator:

![comb1](https://user-images.githubusercontent.com/92947276/144866068-49919aae-ff5d-447d-8839-c6c35d95718d.PNG)

* Unlike Verilog, there is no need to declare variables. Instead, TL-Verilog assigns random values to such variables, producing a warning.
* In this code we are assingning `$rand1[3:0]` and `$rand2[3:0]` to `$val1[31:0]` and `$val2[31:0]` respectively so that smaller values are generated making it easier to debug.

## Sequential Logic

* An example of a sequential circuit is the Fibonacci series circuit, where the next element is the sum of the previous two elements. This can be implemented in TL-Verilog as 
  `$num[31:0] = >>1$num[31:0] + >>2$num[31:0]`.
* Here `>>1` and `>>2` provides the output value from previous cycles.
* Here is the code for sequential calculator:

![comb](https://user-images.githubusercontent.com/92947276/144864452-fd4a86b9-a8f2-4391-9479-9ad4fa1cc8c5.PNG)

* For sequential logic, we need to store data from previous cycles. This is being implemented by using `$val1[31:0] = >>1$out[31:0]`.

## Pipelined Logic

* Pipelining is done to operate the circuit at a higher frequency. The pipeline is divided into stages, and each stage can execute its operation concurrently with the other 
  stages.
* In TL-Verilog we declare a pipeline as `|<pipline name>` and its stages as `<stage>`.
* We will implement the above calculator as 2 stage calculator:

![pipeline](https://user-images.githubusercontent.com/92947276/144869686-34abdd79-95fb-4805-abbe-c5408261d55c.PNG)

* Waveform:

![waveform pipe](https://user-images.githubusercontent.com/92947276/144869904-85d3d8fe-2bc8-48b2-83f4-0379e7e41e2f.PNG)

## Validity

* Validity is the notion of when values of signals are meaningful.
* It makes waveform easier to debug and provides clock gating.
* Clock gating is a technique used in many sequential circuits to reduce power dissipation. It removes the clock signal when the circuit is not in use. In our case, it removes 
  the clock signal when the cycle is not valid.
* After implementing the 2 stage calculator we will add a valid signal to it:

![valid](https://user-images.githubusercontent.com/92947276/144874338-e49519a0-edb1-456b-8ad0-ed9e4f4904a3.PNG)

## Single Value Memory

* The calculator now supports remember and recall function:

![memory](https://user-images.githubusercontent.com/92947276/144876370-38a7533d-337d-405c-a877-8bbef304ab50.PNG)

# Basic RISC-V CPU Micro-Architecture

* We will follow this block diagram to implement RISC V cpu:

![block](https://user-images.githubusercontent.com/92947276/144883029-9c58ee8d-cc7e-4aca-8db2-6cfaaed4ead8.PNG)

* We will implement this design using 4 stages: 
  * Fetch
  * Decode
  * Read and write
  * ALU 

## Fetch

* The shell already contains the instruction memory.
* We will now implement fetch logic :

![fetch](https://user-images.githubusercontent.com/92947276/144884649-253ad83e-3582-4f6c-b0f1-0e3709f5adef.PNG)

* PC(Program Counter) contains the address for the next instruction.
* IMEM contains the set of instructions.
* The processor will fetch the instruction whose address is stored in PC. 

## Decode

* There are 6 types of instructions :
  * I-type : Immediate
  * U-type : Upper Immediate
  * R-type : Register
  * S-type : Store
  * B-type : Branch
  * J-type : Jump

* We will decode `instr[6:2]` which is the opcode of the instruction, we are ignoring the first 2 bits since they are always 1.
* We will also consider immediate value, source address, destination address, funct7, funct3.
* Once we have all of the above fields we can now extract which type of instruction is being fetched.
* Here is our code after the decode stage:

![decode](https://user-images.githubusercontent.com/92947276/144889396-ef2ffbed-832b-4c65-8936-bbdfe4931b16.PNG)

## Register File Read and Write

* Register file in this cpu is capable of performing 2 reads and 1 write simultanously.
* Uncomment `//m4+rf(@1, @1)`, this macro will provide us with default values for the inputs.
* The register file has the following inputs and outputs:

![reg file](https://user-images.githubusercontent.com/92947276/144890850-8e6e3661-0c2b-4b0a-bd68-cfea9645863c.PNG)

* Inputs:
  * `$rf_rd_en1`        : enable signal for the first read operation.
  * `$rf_rd_en2`        : enable signal for the second read operation.
  * `$rf_rd_index1[4:0]`: first address from where data has to be read.
  * `$rf_rd_index2[4:0]`: second address from where data has to be read. 
  * `$rf_wr_en`         : enable signal for write operation.
  * `$rf_wr_index[4:0]` : address where data has to be written.
  * `$rf_wr_data[31:0]` : data to be written in the write address.
* Outputs:
  * `$rf_rd_data1[31:0]`: data from read index 1.
  * `$rf_rd_data2[31:0]`: data from read index 2.

* This is what our code will look like after implementing register file read and write:

![reg file code](https://user-images.githubusercontent.com/92947276/144892707-082331fa-25ab-4649-99c4-f4ae67d8354e.PNG)

## ALU

* After getting the values from register file read, we store `$rf_rd_data1[31:0]` and `$rf_rd_data2[31:0]` in `$src1_value[31:0]` and `$src1_value[31:0]` respectively.
* For now we will perform the arithematic operations 'add' and 'addi' on them. We will add more operations in the final code.
* add  : will perform addition operation on the two operands.
* addi : will add the first operand value with immediate value.
* Register file write will be performed after ALU operations. 
* The snapshot of the code is included in above image.

## Branch Instructions

* In RISC V branches are defined as conditional branches, i.e, they will only execute when certain conditions are met whereas jumps are unconditional.
* There are 6 types of branch instructions:

![branch](https://user-images.githubusercontent.com/92947276/144896344-9660a623-ce6d-4528-b8ee-69a315344250.PNG)

* Here is what our code will look like after implementing branches.

![branch code](https://user-images.githubusercontent.com/92947276/144897148-bbfdc959-de56-49a4-a8a3-cc328be18e10.PNG)

# Pipelining the RISC V CPU

## Hazards

![hazard](https://user-images.githubusercontent.com/92947276/144899161-8b3528d6-91f3-4499-a426-41f84b7972b1.PNG)

* Here we can see two types of hazards.
* First is related to the branch instruction, this is known as a control flow hazard. Here PC is expecting a value from branch target two cycles before that value is actually   
  calculated.
* Second is related to the register file write, this is known as a read after write hazard. We dont need to read a value that is written until stage 4 in the first instruction
  but we might need to read it for the second instruction in stage 2, which is a one cycle early.

## 3 Cycle Valid Signal

* To deal with these hazards we will will operate this machine every 3 cycles.

![sol](https://user-images.githubusercontent.com/92947276/144901360-c4e553e4-5516-4db9-8b01-a951bab7e9f2.PNG)

* We will use the following code to implement this:
```
$start = ((>>1$reset)&&(!$reset))? 1'b1: 1'b0;
$valid = $start ? 1'b1: >>3$valid ? 1'b1 : 1'b0;
```
* This will create the following waveform:
 
![wave](https://user-images.githubusercontent.com/92947276/144901925-284d76c1-a0bd-4062-a274-90ce5bc15bf5.PNG)

* We will modify this logic in the final code.

## Register File Bypass

* To avaoid the read after write hazard use the following code:
```
$src1_value[31:0] = ((>>1$rd == $rs1) && (>>1$rf_wr_en))? >>1$result : $rf_rd_data1[31:0];
$src2_value[31:0] = ((>>1$rd == $rs2) && (>>1$rf_wr_en))? >>1$result : $rf_rd_data2[31:0];
```

## Load and Store Instructions

* Inputs:
  * `$dmem_wr_en`        :Enable signal for write operation
  * `$dmem_rd_en`        :Enable signal for read operation
  * `$dmem_addr[3:0]`    :Address where we have to read/write
  * `$dmem_wr_data[31:0]`:Data to be written in address (store)
* Output:
  * `$dmem_rd_data[31:0]`Data to be read from address (load)

* Here is what the code will look like after we have implemented load and store logic.

![load](https://user-images.githubusercontent.com/92947276/144906303-e8c92547-4afd-4233-b00d-2768c4faf8a5.PNG)

## Finalised RISC V CPU

* After adding the remaining ALU operations and jumps instructions, this will be our final code for the RISC V CPU.

![FINAL CODE](https://user-images.githubusercontent.com/92947276/144907275-111ac656-bd51-4624-a8bd-07687417ad34.PNG)

![FINAL VIZ](https://user-images.githubusercontent.com/92947276/144907300-f876eadd-b5b2-4618-8ab8-b4eab1b7a391.PNG)

* Test case used:
```
*passed = |cpu/xreg[15]>>5$value == (1+2+3+4+5+6+7+8+9);
*failed = 1'b0;
```
* In VIZ we can see 45 in dmem as well as r15, this proves that our load and store instructions are working.

# Acknowledgements
- [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd.
- [Steve Hoover](https://github.com/stevehoover), Founder, Redwood EDA
- [Shivam Potdar](https://github.com/shivampotdar), TA | GSoC '20 @ FOSSi Foundation
- [Vineet Jain](https://github.com/vineetjain07), T.A. for MYTH workshop | GSoC'21
- [Shivani Shah](https://github.com/shivanishah269), TA - MYTH Workshop



















