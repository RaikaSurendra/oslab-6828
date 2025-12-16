# Exercise 2: BIOS Tracing

## Tracing with GDB

I traced the execution of the BIOS from the reset vector (`0xffff0`).

### Observations

1.  **Reset Vector**: The first instruction executed is at `0xffff0` (CS=0xf000, IP=0xfff0).
    ```assembly
    [f000:fff0]    0xffff0: ljmp   $0x3630,$0xf000e05b
    ```
    This jumps to the beginning of the BIOS initialization code.

2.  **Initialization**:
    The BIOS proceeds to initialize the processor state. My trace showed the setup of the stack:

    ```assembly
    [f000:e05b]    0xfe05b: cmpw   $0x68,%cs:(%esi)
    [f000:e062]    0xfe062: jne    0xd241d0b5
    [f000:e066]    0xfe066: xor    %edx,%edx
    [f000:e068]    0xfe068: mov    %edx,%ss
    [f000:e06a]    0xfe06a: mov    $0x7000,%sp
    ```

    - It clears register `DX` (`xor %edx,%edx`).
    - It sets the **Stack Segment (SS)** to 0 (`mov %edx,%ss`).
    - It sets the **Stack Pointer (SP)** to `0x7000` (`mov $0x7000,%sp`).

    This establishes a stack for the BIOS to use during further initialization (testing hardware, POST, finding boot device, etc.).
