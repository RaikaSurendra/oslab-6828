# Exercise 5: Link Address vs Load Address

## The Experiment
We modified the link address of the boot loader to test which instruction would break first. The boot loader is **loaded** at physical address `0x7c00` by the BIOS, but we told the linker it would run at `0x7c20`.

### Steps Taken
1.  Modified `boot/Makefrag` to change `-Ttext 0x7C00` to `-Ttext 0x7C20`.
2.  Recompiled the project.
3.  Traced execution with GDB.

### Findings

The first instruction to break is `lgdt gdtdesc` at offset `0x7c1e`.

#### Analysis
-   **The Instruction**: `lgdt gdtdesc` loads the Global Descriptor Table Register (GDTR) from memory.
-   **The Issue**: The instruction uses an **absolute memory address** to find `gdtdesc`.
-   **Linker Calculation**:
    -   Address of `gdtdesc` relative to start = `0x64`.
    -   Link Address = `0x7c20`.
    -   Target Address = `0x7c20 + 0x64 = 0x7c84`.
-   **Runtime Layout**:
    -   The code is actually loaded at `0x7c00`.
    -   The GDT descriptor is actually at `0x7c00 + 0x64 = 0x7c64`.
    -   Memory at `0x7c84` contains garbage (or code from `readsect`), not the GDT descriptor.

#### The Failure
When `lgdt` executes, it reads 6 bytes from `0x7c84` instead of the valid descriptor at `0x7c64`. This loads an invalid GDT into the CPU.
The CPU continues executing a few more instructions (`mov %cr0`, etc.) until it hits:
```assembly
ljmp    $PROT_MODE_CSEG, $protcseg
```
This jump attempts to load the Code Segment selector `0x8` from the GDT. Since the GDT is garbage, this triggers a CPU fault (likely a Triple Fault), causing the machine to reset.

### Conclusion
The mismatch between Link Address and Load Address breaks any instruction that uses **absolute addressing**. Relative jumps (`jmp label`) still work, but accessing global variables or descriptors fails.
