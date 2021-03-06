## Intel x64 CheatSheet
---
- [Program Template](#program-template)
- [Compile, Link, Shellcode](#compile-link-shellcode)
  * [View](#view)
  * [Generate Shellcode One Liner](#generate-shellcode-one-liner)
- [Registers](#registers)
  * [RFLAGS Register](#rflags-register)
- [Instructions](#instructions)
  * [Moving Data Instructions](#moving-data-instructions)
  * [Comparison and Test Instructions](#comparison-and-test-instructions)
  * [Arithmetic Instructions](#arithmetic-instructions)
  * [Scan / Compare / Copy Strings](#scan--compare--copy-strings)
  * [Jump Instructions](#jump-instructions)
  * [Loop Instructions](#loop-instructions)
  * [Procedure Call instructions](#procedure-call-instructions)
  * [Bit Shift Instructions](#bit-shift-instructions)
  * [Misc Instructions](#misc-instructions)
  * [Loop Instruction](#loop-instruction)
- [System Calls](#system-calls)
- [Referencing Variables](#referencing-variables)
  * [JMP-CALL-POP Technique](#jmp-call-pop-technique)
  * [Stack Technique](#stack-technique)
  * [Relative Addressing](#relative-addressing)
- [Linux x64 Syscalls](#linux-x64-syscalls)
- [Calling Assembly Function from C](#calling-assembly-function-from-c)
- [Calling C Function from Assembly](#calling-c-function-from-assembly)
---
### Program Template
---
```asm
global _start
section .text

_start:

   Code

section .data

   Initialized Variables

section .bss

   Uninitialized Variables
```
### Compile, Link, Shellcode
---
```bash
# nasm -f elf64 file.nasm -o file.o
# ld file.o -o file
# ld -N file.o -o file     <== Option -N allows program to read/write in .text
```
#### View 
---
```bash
# objdump -M intel -d FILE.o
```
#### Generate Shellcode One Liner
---
```bash
# echo “\"$(objdump -d FILE.o | grep '[0-9a-f]:' | 
              cut -d$'\t' -f2 | grep -v 'file' | tr -d " \n" | sed 's/../\\x&/g')\"""
```
### Registers
---

| 64 bits | 32 bits | 16 bits | 8 bits | Description |
|---------|---------|---------|--------|-------------|
| rax | eax | ax | al | Accumulator; used to store some calculation results |
| rbx | ebx | bx | bl | Base; index register for MOVE |
| rcx | ecx | cx | cl | Counter; count for string operations & shifts |
| rdx | edx | dx | dl | Datas; port address for IN and OUT |
| rsi | esi | si | sil| Source Index (source for data copies) |
| rdi | edi | di | dil | Destination Index (destination for data copies) |
| rsp | esp | sp | spl | Stack Pointer |
| rip | eip | ip | ipl | Instruction Pointer |
| rbp | ebp | bp | bpl | <base Pointer (start of stack) |
| r8 | r8d | r8w | r8b | General Purpose |
| ... | ... | ... | ... | General Purpose |
| r15 | r15d | r15w | r15b | General Purpose |

#### RFLAGS Register
---
| Bit #	| Abbreviation | Description	|
|-------|--------------|--------------|
| FLAGS |
| 0	| CF | Carry flag	|
| 1	| Reserved | always 1 in EFLAGS	| 
| 2	|	PF | Parity flag |	
| 3	| Reserved |	 
| 4	| AF | Adjust flag |	
| 5	| Reserved | 
| 6	| ZF | Zero flag |	
| 7	| SF | Sign flag |	
| 8	| TF | Trap flag (single step) |	
| 9	| IF | Interrupt enable flag |	
| 10 | DF	| Direction flag |	
| 11 | OF	| Overflow flag |	
| 12-13 | IOPL | I/O privilege level (286+ only) |
| 14 | NT	| Nested task flag (286+ only) |
| 15 | Reserved. | Always 1 on 8086 and 186,always 0 on later models |	 
| EFLAGS |
| 16 | RF	| Resume flag (386+ only) |	
| 17 | VM	| Virtual 8086 mode flag (386+ only) |	
| 18 | AC	| Alignment check (486SX+ only) |	
| 19 | VIF | Virtual interrupt flag (Pentium+) |	
| 20 | VI | 	Virtual interrupt pending (Pentium+) |	
| 21 | ID	| Able to use CPUID instruction (Pentium+) |	
| 22‑31 | Reserved |
| RFLAGS |
| 32‑63	| Reserved | 

### Instructions
---

#### Moving Data Instructions
---
| Instruction | Description | Opcode | Instruction Size |
|-------------|-------------|--------|------------------|
| mov _op1, op2_ | Mov _op2_ into _op1_ |
| lea _op1, op2_ | Load Effective Address of _op2_ into _op1_ |
| xchg _op1, op2_ | _op1_ <- _op2_ and _op2_ <- _op1_ |

#### Comparison and Test Instructions
---
| Instruction | Description | Opcode | Instruction Size |
|-------------|-------------|--------|------------------|
| cmp _Op1, Op2_ | Set condition codes according to Op1-Op2 |
| test _Op1, Op2_ | Set condition codes according to Op1 & Op2 |

#### Arithmetic Instructions
---
| Instruction | Description | Opcode | Instruction Size |
|-------------|-------------|--------|------------------|
| add _op1, op2_ | _op1_ <- (_op1_ + _op2_) |
| sub _op1, op2_ | _op1_ <- (_op1_ - _op2_) |
| inc _op1_ | _op1_ <- (_op1_ + 1) |
| dec _op1_ | _op1_ <- (_op1_ + 1) |

#### Scan / Compare / Copy Strings
---
| Instruction | Description | Opcode | Instruction Size |
|-------------|-------------|--------|------------------|
| scasb / scasw / scasd / scasq | Compares al/ax/eax/rax with memory pointed by rdi. If equals ZF = 1 |
| cmpsb / cmpsw / cmpsd / cmpsq | [rsi] <- [rdi] |
| lodsb / lodsw / lodsd / lodsq | rax <- [rsi] |
| movsb / movsw / movsd / movsq | [rdi] <- [rsi] |

#### Jump Instructions
---

| Instruction | Description | Condition Code | Opcode | Instruction Size |
|-------------|-------------|----------------|--------|------------------|
| jmp _label_ | Jump to label | 
| je / jz _label_ | Jump if equal/zero | ZF = 0 |
| jne / jnz _label_ | Jump if not equal/nonzero | ZF = 1 |
| js _label_ | Jump if negative | SF = 1 |
| jns _label_ | Jump if nonnegative | SF = 0 |
| jg / jnle _label_ | Jump if greater (signed) | ZF = 0 and SF = OF |
| jge / jnl _label_ | Jump if greater or equal (signed) | SF = OF |
| jl / jnge _label_ | Jump if less (signed) | SF <> OF |
| jle / jng _label_ | Jump if less or equal | ZF = 1 or SF <> OF |
| ja / jnbe _label_ |Jump if above (unsigned) | CF = 0 and ZF = 0|
| jae / jnb _label_ | Jump if above or equal (unsigned) | CF = 0 |
| jb / jnae _label_ | Jump if below (unsigned) | CF = 1 |
| jbe / jna _label_ | Jump if below or equal (unsigned) | CF = 1 or ZF = 1 |

#### Loop Instructions
---
| Instruction | Description | Opcode | Instruction Size |
|-------------|-------------|--------|------------------|
| loop _label_ | Decrements CX and jumps to label if CX <> 0 |
| loope / loopz _label_ | Decrements CX and jumps to label if CX <> 0 and ZF = 1 |
| loopne / loopnx _label_ |Decrements CX and jumps to label if CX <> 0 and ZF = 0

#### Procedure Call instructions
---
| Instruction | Description | Opcode | Instruction Size | 
|-------------|-------------|--------|------------------|
| call _label_ | Push return address and jump to label | 
| call _@_ |Push return address and jump to specified location |
| leave | Set RSP to RBP, then pop top of stack into RBP |
| ret | Pop return address from stack and jump there |

#### Bit Shift Instructions
---
| Instruction | Description | Opcode | Instruction Size | 
|-------------|-------------|--------|------------------|
| rol | Rotate Left. _bit_0_ <- _bit_64_ and CF <- _bit_64_ |
| ror | Rotate Right. _bit_64_ <- bit_0 and CF <- _bit_64_ |
| rcl | Shift Left. _bit_0_ <- CF, then CF <- _bit_64_ |
| rcr | Shift Right._ bit_64_ <- CF, then CF <- _bit_0_ |
| shl / sal |
| shr / sar |

#### Misc Instructions
---
| Instruction | Description | Opcode | Instruction Size | 
|-------------|-------------|--------|------------------|
| sti / cli | Sets IF to 1 or 0 |
| std / cld | Sets DF to 1 or 0 |
| stc / cls / cmc | Sets CF to 1, 0 or inverts it |

#### Loop Instruction
---
| Instruction | Description | Opcode | Instruction Size | 
|-------------|-------------|--------|------------------|
| loop _label_ | rcx -= rcx, if rcx != 0 jumps to _label_. If rcx == 0 continues |
| rep _instruction_ | if rcx != 0 execs _instruction_, then decrements rcx. If rcx == 0 continues |


### System Calls
---
Values returned in RAX.
Registers:
- **RAX** : System Call Number
- **RDI** : Argument 1
- **RSI** : Argument 2
- **RDX** : Argument 3
- **R10** : Argument 4
- **R8** : Argument 5
- **R9** : Argument 6

### Referencing Variables
---

#### JMP-CALL-POP Technique
```asm
global _start
section .text

_start:
   jmp call_shellcode

shellcode:
	pop rsi         ; RSI <- @ string 
	<CODE>

call_shellcode:
	call shellcode
	string db “Hello World”
```
#### Stack Technique
---
Stack String bytes in reverse order. Make **RSI** get **RSP** value, then **RSI** gets the value of @ string

#### Relative Addressing
---
```asm
global _start
section .text
_start:
       jmp real_start
       hello_world: db "Hello World",0xa

real_start:
       lea rsi, [rel hello_world]
```

### Linux x64 Syscalls
---
```c
exit( int)
fork( pointer)
read( uint, char*, int)
write( uint, char*, int)
open( char *, int, int)
``` 

### Calling Assembly Function from C
---
```c
#include <stdio.h>

int add(int op1, int op2);

void(main(void) {
	printf("%d", add(1,2));
}
```
```asm
global add
section data
section .text
add: 
  mov eax, [esp + 4]
  mov eax, [esp + 8]
  ret
```
```bash
# nasm -f elf add.nasm -o add.o
# gcc -Wall main.c add.o -o main
```
### Calling C Function from Assembly
---
```c
int add (int op1, int op2) {
	return a+b;
}
```
```asm
extern add
extern printf
extern exit

global start

section .data
	format: db "%d", 10, 0

section .text
	push 2
	push 1
	call add
	
	push eax
	push format
	call printf
	
	push 0
	call exit
```
```bash
# gcc -Wall -c add.c -o add
# nasm -f main.asm -o main.o
# ld main.o add.o -lc -I /lib/ld-linux.so.2
```



