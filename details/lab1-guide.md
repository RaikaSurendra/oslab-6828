# Lab 1: Comprehensive Guide & Understanding the Flow

## Introduction
Lab 1 is about the very first moments of a computer's life after you press the power button. It traces the handover of control from the hardware (BIOS) to the first software (Boot Loader) and finally to the Operating System (Kernel).

Understanding this lab requires grasping three main components:
1.  **The BIOS**: Hardwired code that starts the machine.
2.  **The Boot Loader**: A tiny program (512 bytes) that loads the bigger OS.
3.  **The Kernel**: The core of the operating system.

---

## 1. The Physical Sequence: What Exactly is Happening?

### Step 1: Power On & The BIOS (16-bit Real Mode)
*   **The State**: When the x86 PC boots, it starts in **Real Mode**. This simulates an ancient Intel 8088 processor (16-bit registers, 1MB addressable memory).
*   **The Reset Vector**: The CPU is hardwired to execute its first instruction at physical address `0xffff0`.
*   **The BIOS Role**: The code at `0xffff0` is the BIOS (Basic Input/Output System).
    *   It initializes the motherboard, checks RAM, and initializes video.
    *   It looks for a "bootable" device (floppy, hard disk, CD).
    *   **The Handover**: Once it finds a disk, it reads the **first 512 bytes** (Sector 0) of that disk into memory at address `0x7c00`.
    *   It jumps to `0x7c00`.

### Step 2: The Boot Loader (boot/boot.S and boot/main.c)
Now your code is running! But you are constrained:
*   You are still in **16-bit Real Mode**.
*   You only have 512 bytes of code.

**Key Tasks:**
1.  **Enable A20**: Unlocking memory above 1MB (a legacy quirk).
2.  **Switch to Protected Mode**:
    *   Load a Global Descriptor Table (GDT).
    *   Set the `PE` (Protection Enable) bit in `CR0`.
    *   Execute a far jump (`ljmp`) to reload the code segment.
    *   **Result**: You are now in **32-bit Protected Mode**. You can access all 4GB of memory.
3.  **Load the Kernel**:
    *   The boot loader acts as a disk driver. It reads the **ELF Header** of the kernel code from the disk.
    *   It copies the kernel's "segments" (code and data) into memory at physical address `0x00100000` (1MB mark).
4.  **Jump to Kernel**: Finally, it calls the entry point of the kernel.

### Step 3: The Kernel (kern/entry.S, kern/init.c, etc.)
The OS is now in charge.
*   **Virtual Memory**: The kernel initializes paging to map virtual addresses (like `0xf0100000`) to physical addresses (like `0x00100000`).
*   **Console**: It sets up the serial port and VGA monitor so `cprintf` works.
*   **Monitor**: It drops into a command-line interface (`K>`) waiting for your commands.

---

## 2. Key Concepts to Master

### Real Mode vs. Protected Mode
*   **Real Mode**: `Address = Segment * 16 + Offset`. Safe, simple, but limited to 1MB.
*   **Protected Mode**: `Address = Paging(Segment Selector + Offset)`. Allows virtual memory, permissions, and 4GB+ RAM.

### The Stack
The stack matches the memory model.
*   The BIOS sets up a tiny stack.
*   The Boot Loader moves it.
*   The Kernel sets up its own large stack.
*   *Crucially*, C functions rely on the stack to store local variables and return addresses. If `esp` (stack pointer) is wrong, the program crashes.

### ELF (Executable and Linkable Format)
The "Format" of the binary files.
*   **Header**: "I am an ELF file, my entry point is X."
*   **Program Headers**: "Load Z bytes from file offset A to memory address B."
The boot loader manually parses this to figure out where to put the kernel bytes in RAM.

---

## 3. How to Navigate and Debug

### Using the Tools
*   **`make qemu-nox`**: Runs the code. Good for seeing if it works.
*   **`make qemu-nox-gdb` + `make gdb`**: The microscope.
    *   `si` (Step Instruction): Execute one assembly instruction.
    *   `x/i $eip`: "Examine Instruction at Instruction Pointer". Shows you the *next* line of assembly code.
    *   `b *0x7c00`: Break at the boot sector load point.

### Reading Assembly
Don't fear the `.asm` files.
*   `obj/boot/boot.asm` and `obj/kern/kernel.asm` are your best friends.
*   They show the **C code** interleaved with the **Assembly** it generated and the **Memory Address** where it lives.
*   Use them to correlate what you see in GDB with your source code.

## Summary Checklist
- [ ] **BIOS**: Starts at `0xffff0`, jumps to `0x7c00`.
- [ ] **Boot Loader**: `0x7c00`. Real -> Protected Mode. Reads ELF. Jumps to Kernel.
- [ ] **Kernel**: Sets up virtual memory, stack, and runs `i386_init`.
