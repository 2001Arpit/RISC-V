# RISC-V Microprocessor 
# Overview
Creating a RISC-V pipelined core, which has support of base interger RV32I instruction format using TL-Verilog on makerchip platform.
# Table of Contents
- [Introduction](#introduction)
- [GNU compiler tool chain](#gnu-compiler-toolchain)
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
    
We will see the following commands in action with a program to compute the sum of first 'n' numbers.

![sum_n code2](https://user-images.githubusercontent.com/92947276/144846309-21de62b2-0a00-459a-a3e1-227aa5a7439f.PNG)

Output:

![sum_n op](https://user-images.githubusercontent.com/92947276/144847071-0d7e275c-2201-4e26-a80c-5870988e3c4a.PNG)









    
    
    
