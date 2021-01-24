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
| FLAGS |
0	| CF | Carry flag	|
1	| Reserved | always 1 in EFLAGS	| 
2	|	PF | Parity flag |	
3	| Reserved |	 
4	| AF | Adjust flag |	
5	| Reserved | 
6	| ZF | Zero flag |	
7	| SF | Sign flag |	
8	| TF | Trap flag (single step) |	
9	| IF | Interrupt enable flag |	
10 | DF	| Direction flag |	
11 | OF	| Overflow flag |	
12-13 | IOPL | I/O privilege level (286+ only) |
14 | NT	| Nested task flag (286+ only) |
15 | Reserved. | Always 1 on 8086 and 186,always 0 on later models |	 
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
