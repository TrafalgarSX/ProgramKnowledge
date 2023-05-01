### guide to reading (x86)assembly language
---
Understanding assembly language is essential to understanding how things really work, and can give you better insights into how things work, whether you're building systems up or tearing systems down. Reading assembly language is not a replacement for proper reverse engineering tools like **Ghidra or IDA**, but it is a necessary complimentary skill.

### reading is easier than writing
---
On a modern Intel CPU instructions number is close to 1000. You could imagine trying to find the right instruction for a specific situation would be difficult. In reality, the number of instructions you need to learn to read is quite small. 

In one binary I looked at, 83% of the instructions used were the 10 most frequent instructions, and many of the top 30 instructions are just slight variations (like `AND` and `OR`).

![[frequently-instructions.png]]

### Two flavors: AT&T and Intel syntax
---
For historical reasons, there are two “flavors” of disassembly syntax for x86. One is called “Intel” and the other is called “AT&T”. If you live in the Windows world, you may never see AT&T syntax, but some open source tools will default to AT&T syntax so it’s good to recognize when you’re dealing with it.

The biggest difference you’ll see between the two flavors is that the order of operands is **reversed**! Here’s an example of AT&T syntax:

```
addl $4, %eax
```

And here’s an example of Intel syntax:

```
add eax, 4
```

Besides the order being swapped, constants also get prefixed with `$`, and registers are prefixed with `%`. Some mnemonics also have a letter appended to indicate the size of the operands, such as `l` for 32-bit operands.

[[x86汇编语言语法#x86汇编语言语法]]

**WinDbg** only supports Intel syntax, for instance. Many **open source tools** will default to AT&T syntax, but will have an option to enable Intel syntax. For objdump, you can use -M intel. For instance:

```
objdump -d -M intel ./a.out
```

While I’m sure someone has an argument in favor of the AT&T syntax, I’d suggest avoiding it for one simple reason: **The Intel manual (SDM) uses Intel syntax, and is a crucial resource to understand how an instruction works.** So to avoid confusion, stick with Intel syntax.

### The parts of an instruction: mnemonic, operands, and prefixes
---
A single assembly language unit is an "instruction", and it has three parts.

- mnemonic 助记符
The “mnemonic” is the name of the instruction, like “ADD” or “MOV”, which **tells you what the instruction does.** Most mnemonics are an **abbreviated** version of a word, like MOV (move), SUB (subtract), or INC (increment). 

Other operations have an **abbreviated phrase** as the mneumoic, such as LEA (Load Effective Address), or SAL (Shift Arithmetic Left). 

There are complex instructions that operate on “vectors” (also known as SIMD or Single Instruction Multiple Data) which tend to have long and complicated names, but luckily you won’t have to read things like PMADDUBSW very frequently (Multiply signed and unsigned bytes, add horizontal pair of signed words, pack saturated signed-words).

- operands 操作数
Each instruction can **take 0 to 3 operands**, although 2 operands is the most common. **Operands could be registers, constants, or memory locations**, but not every type and combination of operand type is available for every instruction. For instance, there is no version of `MOV` that takes two memory locations, but `MOV reg, mem` and `MOV mem, reg` are both available for moving data around.

For instructions that take at least one operand, the first operand is often a destination for data to be written and is usually a _source_ as well. `INC EAX`

Any operands after the first are generally only read and not written. One notable exception to that rule is the **`XCHG` instruction which can swap two registers or a register and a memory location.**

- prefixes
Instructions can also have “prefixes” which modify the behavior of the instruction. The two types you are most likely to encounter are `LOCK` for making certain read/modify/write operations atomic, and `REP`/`REPZ`/`REPNZ` which are used for “string” operations for copying/comparing a sequence of bytes.

### Memory operands
---
Intel has a **flexible (and complex) set of memory addressing modes.** Luckily, when reading the code **you don’t need to know** most of this because with Intel syntax the address expression will always be described as a simple expression such as:
```
mov     rbp,[rsp-170h]
```

The most complex expressions can contain two registers, a constant, and a “scale factor” for one of the registers. For instance:
```
mov     rax,dword ptr [rbp+rcx*4+1234h]
```

One special instruction that takes a memory address is the `LEA` instruction, which stands for Load Effective Address. Unlike every other instruction that takes a memory operand, this one does not read or write to the address, but only calculates the _effective address_ where the memory load or store would be.

[[LEA-有效地址加载]]

### Registers
---
The x64 register set has an instruction pointer (RIP) and 16 general purpose registers (RAX, RCX, RDX, RBX, RSP, RBP, RSI, RDI, R8-R15)3. By “General Purpose” we mean that **they can be generally used for storing whatever you want in them.**

For instance, a `MOV` instruction can be used to move values into/out of any of the general purpose registers, but cannot directly access the instruction pointer, RIP.

Although they can be used for general purpose, some of the registers have special purposes and are used implicitly by certain instructions. For example 
1. the RSP register is always used with any operation that touches the stack, like the `CALL` and `PUSH` instructions. 
2. The RSI and RDI registers are used implicitly for any of the “string operations” like `MOVS`. 

Some registers are simply more efficient to use in certain cases, but when reading assembly code you generally don’t need to worry about this.

While all of these registers are **64 bits**, x86 can use smaller parts of most registers, and we use a different name when talking about the smaller parts of the registers. **EAX is the low 32 bits of RAX. AX is the low 16 bits of RAX. AL is the low 8 bits of RAX.** And we also have AH that can be used to describe the high 8 bits of AX.


![[register-Segs.png]]