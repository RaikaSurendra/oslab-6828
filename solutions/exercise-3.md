# Exercise 3: Boot Loader Analysis

## Boot Loader Questions

**1. At what point does the processor start executing 32-bit code? What exactly causes the switch from 16- to 32-bit mode?**

The processor starts executing 32-bit code after the far jump instruction in `boot/boot.S`:
```assembly
ljmp    $PROT_MODE_CSEG, $protcseg
```
The switch from 16-bit real mode to 32-bit protected mode is enabled by setting the **Protection Enable (PE)** bit (bit 0) in the `CR0` control register:
```assembly
movl    %cr0, %eax
orl     $CR0_PE_ON, %eax
movl    %eax, %cr0
```

**2. What is the last instruction of the boot loader executed, and what is the first instruction of the kernel it just loaded?**

The last instruction of the boot loader executed is the function pointer call in `boot/main.c` that jumps to the kernel's entry point:
```c
((void (*)(void)) (ELFHDR->e_entry))();
```
(In assembly, this compiles into a `call` instruction to the address stored in `ELFHDR->e_entry`).

**3. Where is the first instruction of the kernel?**

The first instruction of the kernel is located at the memory address specified by `ELFHDR->e_entry`.
From previous `kerninfo` output, we know this address is **0x0010000c** (Physical Address).

**4. How does the boot loader decide how many sectors it must read in order to fetch the entire kernel from disk? Where does it find this information?**

The boot loader determines the size and location of the kernel by reading the **ELF Program Headers** (`struct Proghdr`).
1.  It first reads the first sector (which it assumes contains the ELF Header) to `0x10000`.
2.  It verifies the ELF Magic number.
3.  It finds the Program Headers array at offset `e_phoff`.
4.  It iterates through `e_phnum` program headers.
5.  For each header, it reads `p_memsz` bytes from the disk offset `p_offset` into the physical address `p_pa`.
```c
ph = (struct Proghdr *) ((uint8_t *) ELFHDR + ELFHDR->e_phoff);
eph = ph + ELFHDR->e_phnum;
for (; ph < eph; ph++)
    readseg(ph->p_pa, ph->p_memsz, ph->p_offset);
```
