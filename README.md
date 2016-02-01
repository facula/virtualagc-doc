# virtualagc-doc
A bit of translation of the assembler mnemonics, especially the ones that might be weird (which is a lot of them, because the AGC architecture isn't much like most modern architectures):
First, a general note that the registers are generally addressable as memory, and so instructions that operate on the accumulator and memory can also usually operate on the accumulator and another register (or even on the accumulator twice, as in CCS A).
- CA (clear and add), CAF (clear and add from fixed memory), CAE (clear and add from eraseable memory) = load a value into the accumulator. I've also seen it translated as "copy to accumulator", but given the existence of CS which loads the negative of a value into the accumulator, I think that "clear and add" and "clear and subtract" are the correct original meanings. Still, copying into the accumulator is what it does, so there's no harm in thinking of it that way.
- AD (add) = add a value in memory to the accumulator.
- SU (subtract) = subtract a value in memory from the accumulator.
- MASK = bitwise AND a value in memory with the accumulator.
- TS (transfer to storage) = write the accumulator to memory.
- ADS (add and store) = add the accumulator to a value in memory.
- XCH (exchange) = exchange the value in the accumulator with a value in memory.
- TC (transfer control) = call. Jump to the given address, and store the address of the instruction following the TC for a later RETURN. Note that there is no call stack, just a single return-address register, so if you want to TC from a routine that itself RETURNs, you need to store the return address somewhere and restore it before RETURN.
- TCF (transfer control to location in fixed memory) = jump. Doesn't store a return address.
- RETURN = jump to the address in the return address register.
- TCAA (transfer control to address in accumulator) = jump to the address in A, one way of doing an indirect jump. Actually sugar for TS Z (store the accumulator into the program counter).
- INHINT (inhibit interrupts) = disable interrupts.
- RELINT (release interrupts) = enable interrupts.
- BZF (branch if zero to location in fixed memory) = jump if the accumulator is zero.
- BZMF (branch if zero or minus to location in fixed memory) = jump if the accumulator is zero or negative.
- OVSK (overflow skip) = skip the following instruction if the accumulator contains an overflow.
- CCS (count, compare, and skip) = a general purpose loop / compare operation. Reads a value from memory, stores one less than that value to the accumulator, then jumps to one of four following instructions depending on whether the accumulator is positive, negative, +0, or -0. CCS A decrements the accumulator and jumps depending on whether it decremented to zero, akin to x86 LOOP.
- INDEX = read a value from memory and use it to modify the following instruction, by adding the value retrieved from memory to the instruction itself. Instructions are generally encoded with useful things in their lower bits, such that e.g. INDEX foo ; CA bar loads the value from bar+[foo] into A, and INDEX A ; TC foo jumps to foo+A, another way of accomplishing an indirect jump. In stranger cases, you can INDEX on a value with higher bits set, which changes the type of the instruction entirely, and that's okay.
- TC INTPRET = call the interpreter. The interpreter is a routine that executes a completely different instruction set, with instructions for trigonometry, matrix operations, and other complex mathematical operations on fixed-point quantities of various precision. Interpreter code is much smaller than assembly code to do the same thing (which is important, since the computer only had about 72kB of ROM), but also quite slow so it's only used where needed. The interpreter uses the return-address register set by TC not as a return address, but as the address to begin interpreting at, so that interpreted code can be threaded directly in with regular code. When the interpreter reads an EXIT instruction it jumps to the following instruction word, switching back to regular mode.
