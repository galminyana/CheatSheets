## Intel x64 CheatSheet
---
---
### Registers
---

| 64 bits | 32 bits | 16 bits | 8 bits | Description |
|---------|---------|---------|--------|-------------|
| rax | eax | ax | al | |
| rbx | ebx | bx | bl | |
| rcx | ecx | cx | cl | |
| rdx | edx | dx | dl | |
| rsi | esi | si | sil| Source Index (source for data copies) |
| rdi | edi | di | dil | Destination Index (destination for data copies) |
| rsp | esp | sp | spl | Stack Pointer |
| rip | eip | ip | XX | Instruction Pointer |
| rbp | ebp | bp | XX | <base Pointer (start of stack) |
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

#### Comparison and Test Instructions
---
| Instruction | Description | 
|-------------|-------------|
| cmp Op1, Op2 | Set condition codes according to Op1-Op2 |
| test Op1, Op2 | Set condition codes according to Op1 & Op2 |

#### Jump Instructions
---

| Instruction | Description | Condition Code | 
|-------------|-------------|----------------|
| jmp label | Jump to label |
| je / jz label | Jump if equal/zero | ZF |
| jne / jnz label | Jump if not equal/nonzero | ~ZF |
| js label | Jump if negative | SF |
| jns label | Jump if nonnegative | ~SF |
| jg / jnle label | Jump if greater (signed) |
| jge / jnl label | Jump if greater or equal (signed) |
| jl / jnge label | Jump if less (signed) |
| jle / jng label | Jump if less or equal |
| ja / jnbe label |Jump if above (unsigned) |
| jae / jnb label | Jump if above or equal (unsigned) | ~CF |
| jb / jnae label | Jump if below (unsigned) | CF |
| jbe / jna label | Jump if below or equal (unsigned) | CF \| ZF |

#### Procedure Call instructions
---
| Instruction | Description | 
|-------------|-------------|
| call label | Push return address and jump to label | 
| call @ |Push return address and jump to specified location |
| leave | Set RSP to RBP, then pop top of stack into RBP |
| ret | Pop return address from stack and jump there |


### System Calls
---
Values returned in RAX.
Registers:
- RAX : System Call Number
- RDI : Argument 1
- RSI : Argument 2
- RDX : Argument 3
- R10 : Argument 4
- R8 : Argument 5
- R9 : Argument 6







