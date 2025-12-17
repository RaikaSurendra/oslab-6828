# Lab 1 Reference Guide

This document provides a reading guide for the official [Lab 1: Booting a PC](https://pdos.csail.mit.edu/6.828/2018/labs/lab1/) instructions and the Intel 80386 Programmer's Reference Manual.

## 1. Analyzing the Lab 1 Instructions

The lab is divided into three parts. Here is what to focus on in each:

### Part 1: PC Bootstrap
> [!IMPORTANT]
> **Goal**: Understand how the hardware hands over control to software.

*   **Key Concept**: The **Reset Vector**. The CPU starts here:
    ```diff
    + CS:IP = 0xf000:fff0 (Physical Address: 0xffff0)
    ```

```mermaid
flowchart TD
    PowerOn[Power On] --> Reset[CPU Reset Vector<br/>0xffff0]
    Reset --> BIOS[BIOS<br/>(Initialize Hardware)]
    BIOS --> Search[Search for Bootable Device]
    Search --> Load[Load Boot Sector<br/>Drive 0 Sector 0]
    Load --> RAM[RAM 0x7c00]
    RAM --> Jump[Jump to 0x7c00<br/>Handover to Boot Loader]
    
    style Reset fill:#f9f,stroke:#333
    style BIOS fill:#bbf,stroke:#333
    style Jump fill:#bfb,stroke:#333
```

### Part 2: The Boot Loader
> [!NOTE]
> **Goal**: Break the 1MB barrier and load the OS.

*   **Key Files**: `boot/boot.S` (Assembly) and `boot/main.c` (C).
*   **Real vs. Protected Mode**:

```mermaid
graph LR
    subgraph RealMode [Real Mode (16-bit)]
        RM_Limit[Limit: 1MB]
        RM_Addr[Addr = Segment*16 + Offset]
    end
    
    subgraph ProtMode [Protected Mode (32-bit)]
        PM_Limit[Limit: 4GB]
        PM_Addr[Virtual Memory Paging]
    end

    RealMode -->|Set CR0 PE Bit<br/>ljmp instruction| ProtMode
    
    style RealMode fill:#ff9,stroke:#333
    style ProtMode fill:#9f9,stroke:#333
```

### Part 3: The Kernel
> [!TIP]
> **Goal**: Set up the heavy lifting (Virtual Memory, Stack).

*   **Key Files**: `kern/entry.S`, `kern/monitor.c`.
*   **Key Concept**: **Virtual Memory** (Link vs Load Address).
    ```diff
    - LOAD ADDRESS (Physical): 0x00100000
    + LINK ADDRESS (Virtual):  0xf0100000
    ```

---

## 2. Navigating the Intel 80386 Manual
(Source: [80386 Programmer's Reference Manual](https://pdos.csail.mit.edu/6.828/2018/readings/i386/toc.htm))

> [!WARNING]
> The manual is massive. **Do not read it cover-to-cover.**

### Must-Read for Lab 1

#### 1. [Chapter 2: Basic Programming Model](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c02.htm)
*   **2.3 Registers**: You will see these in GDB constantly.
    *   `EAX`, `EBX`, `ECX`, `EDX`: General Purpose.
    *   `ESP`: **Stack Pointer** (Critical!).
    *   `EIP`: **Instruction Pointer** (Where are we executing?).

#### 2. [Chapter 14: 80386 Real-Address Mode](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c14.htm)
*   Describes the "ancient" state of the CPU at power-on.
*   Explains why `boot/boot.S` starts with `.code16`.

#### 3. [Chapter 10: Initialization](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c10.htm)
> [!IMPORTANT]
> **10.1 Processor State After Reset**: This lists the exact values of registers when you press the power button.

### Advanced Reference (Skim)

*   **[Chapter 5: Memory Management](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c05.htm)**: Segmentation & Paging.
*   **[Chapter 17: Instruction Set](https://pdos.csail.mit.edu/6.828/2018/readings/i386/c17.htm)**: Dictionary for assembly instructions like `LGDT`, `LMSW`, `CLI`.
