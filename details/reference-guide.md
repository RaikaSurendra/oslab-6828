# Lab 1 Reference Guide

This document provides a reading guide for the official Lab 1 instructions and the Intel 80386 Programmer's Reference Manual.

## 1. Analyzing the Lab 1 Instructions
(Source: [Lab 1: Booting a PC](https://pdos.csail.mit.edu/6.828/2018/labs/lab1/))

The lab is divided into three parts. Here is what to focus on in each:

### Part 1: PC Bootstrap
*   **Goal**: Understand how the hardware hands over control to software.
*   **Key Concept**: The **Reset Vector**. The CPU doesn't start at 0; it starts at `0xffff0`.
*   **Action details**: 
    *   The **BIOS** runs first. It is "firmware" (software burned into hardware).
    *   It performs self-tests (POST) and then loads the **Boot Sector** (512 bytes) from the disk to `0x7c00`.

### Part 2: The Boot Loader
*   **Goal**: Break the 1MB barrier and load the OS.
*   **Key Files**: `boot/boot.S` (Assembly) and `boot/main.c` (C).
*   **Key Concept**: **Real Mode vs. Protected Mode**.
    *   *Real Mode*: 16-bit, 1MB memory limit. Used by BIOS.
    *   *Protected Mode*: 32-bit, 4GB memory access. Used by OS.
    *   *The Switch*: Happens in `boot.S` using the Global Descriptor Table (GDT) and Control Register 0 (`CR0`).

### Part 3: The Kernel
*   **Goal**: Set up the environment for the rest of the OS.
*   **Key Files**: `kern/entry.S`, `kern/monitor.c`.
*   **Key Concept**: **Virtual Memory** and **Stack**.
    *   The kernel links at a high virtual address (`0xf0100000`) but loads at a low physical address (`0x00100000`).
    *   Paging allows this mapping.

---

## 2. Navigating the Intel 80386 Manual
(Source: [80386 Programmer's Reference Manual](https://pdos.csail.mit.edu/6.828/2018/readings/i386/toc.htm))

The manual is massive. Do not read it cover-to-cover. Focus on these chapters for Lab 1:

### Must-Read for Lab 1
1.  **[Chapter 2: Basic Programming Model](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c02.htm)**
    *   **2.3 Registers**: Learn `EAX`, `EBX`, `ESP`, `EIP`. You will see these in GDB constantly.
    *   **2.5 Data Types**: Understand bytes, words, and doublewords.

2.  **[Chapter 14: 80386 Real-Address Mode](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c14.htm)**
    *   This describes the state of the CPU when it powers on (BIOS mode).
    *   Crucial for understanding why `boot/boot.S` starts with `.code16`.

3.  **[Chapter 10: Initialization](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c10.htm)**
    *   **10.1 Processor State After Reset**: This lists the exact values of registers (like `CS` and `EIP`) when the power button is pressed. This explains the `0xffff0` entry point.

### Advanced / Reference (Skim for now)
4.  **[Chapter 5: Memory Management](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c05.htm)**
    *   Explains **segmentation** (translating logical -> linear addresses) and **paging** (translating linear -> physical addresses).
    *   Relevant for understanding the GDT in `boot.S` and the page tables in `kern/entrypgdir.c`.

5.  **[Chapter 17: Instruction Set](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c17.htm)**
    *   Use this as a dictionary when you see an assembly instruction you don't know (e.g., `LGDT`, `LMSW`, `CLI`).
