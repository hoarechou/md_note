### How is Bios ROM mapped into address space on PC


On the PC there's always some address decoding logic involved because there are a few "holes/windows" in the physical address space through which the BIOS ROM and I/O devices (e.g. video card) are accessible instead of the RAM. That's by design, for compatibility reasons, so older programs can still run on newer computers.

As for the initial address at which the CPU starts execution after a reset, if you look at the documentation, you will see that Pentium-class CPUs start with this:
<br>
**EIP=0xFFF0<br>
CS.Selector=0xF000<br>
CS.Base=0xFFFF0000<br>**

If you follow the normal real-mode addressing scheme, the physical address should be **CS.Selector*16+IP**, or, with the values substituted, 0xFFFF0. However, the CPU actually calculates the address using **CS.Base+(E)IP** (in the real and 16/32-bit protected mode, but not in virtual 8086 or 64-bit protected mode), hence the first address that the CPU requests from the memory is going to be **0xFFFFFFF0**. Your inability to use far jumps to code within the ROM at that high address may be due to the fact that loading into CS will reset CS.Base to 16 * the new value of CS.Selector. So, jumping to, say, 0xF000:0xFFF0 will transfer control to 0xFFFF0 instead of 0xFFFFFFF0 and unless the ROM is also mapped at that low location in the memory and the code in it is suited for running with CS(.Selector)=0xF000, it's not going to run.

Also, neither the CPU nor the circuitry around it has to support all 32 (or more) address lines if the PC is limited to have at most 16MB (as it was on i80286 and i80386SX) or 4GB (as it was on i80386DX/original i80386 and i80486) or 240-52 bytes (on 64-bit capable Pentium-class CPUs) and if that's the case, if a number of high bits in the physical address space are ignored, execution can be said to effectively start at an address lower than the theoretical maximum - 16, e.g. 0x00FFFFF0 (i80286/i80386SX).

If you need to resolve problems with your board, see its documentation and schematics to find out how the ROM is mapped into the physical address space on it.
