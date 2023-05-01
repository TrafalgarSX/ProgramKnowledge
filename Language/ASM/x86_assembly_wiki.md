### Syntax
---
x86 assembly language has two main syntax branches: Intel syntax and AT&T syntax. Intel syntax is dominant in the DOS and Windows world, and AT&T syntax is dominant in the Unix world, since Unix was created at AT&T Bell Labs. 

![[assmebly_syntax.png]]

### Registers
---
-   AX multiply/divide, string load & store
-   BX index register for MOVE
-   CX count for string operations & shifts
-   DX [port](https://en.wikipedia.org/wiki/Computer_port_(hardware) "Computer port (hardware)") address for IN and OUT
-   SP points to top of [the stack](https://en.wikipedia.org/wiki/Stack_(abstract_data_type) "Stack (abstract data type)")
-   BP points to base of the stack frame
-   SI points to a source in stream operations
-   DI points to a destination in stream operations

Along with the **general registers** there are additionally the:

- IP instruction pointer
- [FLAGS](https://en.wikipedia.org/wiki/FLAGS_register_(computing) "FLAGS register (computing)")
- segment registers (CS, DS, ES, FS, GS, SS) which determine where a 64k segment starts (no FS & GS in [80286](https://en.wikipedia.org/wiki/Intel_80286 "Intel 80286") & earlier)
- extra extension registers ([MMX](https://en.wikipedia.org/wiki/MMX_(instruction_set) "MMX (instruction set)"), [3DNow!](https://en.wikipedia.org/wiki/3DNow! "3DNow!"), [SSE](https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions "Streaming SIMD Extensions"), etc.) (Pentium & later only).

### Segmented addressing
---
[[实模式-保护模式]]

There are some special combinations of segment registers and general registers that point to important addresses:
- CS:IP (CS is Code Segment, IP is Instruction Pointer) points to the address where the processor will fetch the next byte of code.
- SS:SP (SS is Stack Segment, SP is Stack Pointer) points to the address of the top of the stack, i.e. the most recently pushed byte.
- DS:SI (DS is Data Segment, SI is Source Index) is often used to point to string data that is about to be copied to ES:DI.
- ES:DI (ES is Extra Segment, DI is Destination Index) is typically used to point to the destination for a string copy, as mentioned above.

### Execution modes
---
The x86 processors support five modes of operation for x86 code, **Real Mode**, **Protected Mode**, **Long Mode**, **Virtual 86 Mode**, and **System Management Mode**, in which some instructions are available and others are not.

- [Real mode](https://en.wikipedia.org/wiki/Real_mode) (16-bit)   实模式（16 位）
   - 20-bit segmented memory address space (meaning that only 1 [MB](https://en.wikipedia.org/wiki/Megabyte "Megabyte") of memory can be addressed—actually, slightly more), direct software access to peripheral hardware, and no concept of [memory protection](https://en.wikipedia.org/wiki/Memory_protection "Memory protection") or [multitasking](https://en.wikipedia.org/wiki/Computer_multitasking "Computer multitasking") at the hardware level. Computers that use [BIOS](https://en.wikipedia.org/wiki/BIOS "BIOS") start up in this mode.
   
- [Protected mode](https://en.wikipedia.org/wiki/Protected_mode "Protected mode") (16-bit and 32-bit)
   - Expands addressable [physical memory](https://en.wikipedia.org/wiki/Physical_memory "Physical memory") to 16 [MB](https://en.wikipedia.org/wiki/Megabyte "Megabyte") and addressable [virtual memory](https://en.wikipedia.org/wiki/Virtual_memory "Virtual memory") to 1 [GB](https://en.wikipedia.org/wiki/Gigabyte "Gigabyte"). Provides privilege levels and [protected memory](https://en.wikipedia.org/wiki/Protected_memory "Protected memory"), which prevents programs from corrupting one another. 16-bit protected mode (used during the end of the [DOS](https://en.wikipedia.org/wiki/DOS "DOS") era) used a complex, multi-segmented memory model. 32-bit protected mode uses a simple, flat memory model.
   
- [Long mode](https://en.wikipedia.org/wiki/Long_mode "Long mode") (64-bit)
   - Mostly an extension of the 32-bit (protected mode) instruction set, but unlike the 16–to–32-bit transition, many instructions were dropped in the 64-bit mode. Pioneered by [AMD](https://en.wikipedia.org/wiki/Advanced_Micro_Devices "Advanced Micro Devices").

- [Virtual 8086 mode](https://en.wikipedia.org/wiki/Virtual_8086_mode) (16-bit)
   - A special hybrid operating mode that allows real mode programs and operating systems to run while under the control of a protected mode supervisor operating system
   
- System Management Mode (16-bit)


### Instruction types
---
##### Stack instruction

##### Integer ALU instructions

##### Floating-point instructions

##### SIMD instructions
SIMD(Single Instruction Multiple Data) 单指令流多数据流：是一种采用一个控制器来控制多个处理器，同时对一组数据（又称 数据向量）中的每一个分别执行相同的操作从而实现空间上的并行性的技术。

##### Memory instructions
