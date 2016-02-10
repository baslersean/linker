# linker
Linker implementation in Java

Implemenation of a two-pass linker in Java. Once compiled, it will ask for the name of the text file from the user as input.

Description:

The target machine is word addressable and has a memory of 600 words, each consisting of 4 decimal digits. The first (leftmost) digit is the opcode, which is unchanged by the linker. The remaining three digits (called the address field) are either:

(1) An immediate operand, which is unchanged.
(2) An absolute address, which is unchanged.
(3) A relative address, which is relocated.
(4) An external address, which is resolved.

The linker relocates relative addresses and resolves external references to produce a final output from input consisting of a series of object modules, each of which contains three parts: definition list, use list, and program text.

The linker processes the input twice. Pass one determines the base address for each module and the absolute address for each external symbol, storing the later in the symbol table it produces. Pass two uses the base addresses and the symbol table computed in pass one to generate the actual output by relocating relative addresses and resolving external references.

