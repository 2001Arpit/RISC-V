# RISC-V Microprocessor 
# Overview
Creating a RISC-V pipelined core, which has support of base interger RV32I instruction format using TL-Verilog on makerchip platform.
# Table of Contents
- [Introduction](#introduction)
- [GNU compiler toolchain](#gnu-compiler-toolchain)
- [Application Binary Interface](#application-binary-interface)
- [Digital Logic in TL-Verilog Using Makerchip](#digital-logic-in-tl-verilog-using-makerchip)
    -[Combinational Logic](##combinational-logic)
    -[Sequential Logic](##sequential-logic)
    -[Pipelined Logic](##pipelined-logic)
    -[Validity](##validity)
    -[Single Value Memory](##single-value-memory)
    
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

# 



