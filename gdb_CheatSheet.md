## GDB CheatSheet
---
---
### 

### Assembly
---
| (gdb$) | Description |
|--------|-------------|
| set disassembly-flavor intel | Intel sintax for code |
| disassemble | Prints ASM instructions |

### BreakPoints
---
| (gdb$) | Description |
|--------|-------------|
| break _function_ | Breakpoint at _function_ |
| break \*0x12345678 | Breakpoint at address 0x12345678 |
| info break | List breakpoints and watchpoints |
| clear \[breakpoint_id\] | Deletes one or all existing breakpoints |
| disable \[breakpoint_id\] | Disables one or all breakpoints |


### Controling Execution
---
| (gdb$) | Description |
|--------|-------------|
| run \[args\] | Execute program \[with parameters\] until breakpoint |
| stepi \[count\] | Step-into instruction (one or count forward) |
| step | Step into next instruction |
| nexti \[count\] | Step-over (one or count, stepping over function calls) |
| next | Next instruction, thought functions |
| continue | Continues normal execution until breakpoint |

### Memory Contents
---
- **\<n\>** : Number of locations to display
- **\<f\>** : Display Format
  - **d** : Signed decimal
  - **x** : Hex
  - **u** : Unsigned decimal
  - **c** : Charatcer
  - **s** : Null terminated string
  - **f** : Float
- **\<u\>** : Unit Size
  - **b** : Byte
  - **h** : Half Word
  - **w** : Word
  - **q** : Wuad
 
 
| (gdb$) | Description |
|--------|-------------|
| x/\<n\>\<f\>\<u\> \$rsp | Examine contents of stack |
| x/\<n\><<f\>\<u\> &_variable_ | Examine memory location _variable_, \<n\> number of locations |
| x/s 0x12345678 | Display NULL terminated string starting at address 0x12345678 |
| x/8xb 0x12345678 | Display 8 Hex bytes of memory starting at 0x12345678 |




### Logging
---
| (gdb$) | Description |
|--------|-------------|
| set logging file _filename_ | Default is _gdb.txt_ |
| set logging on/off | Enable or Disable logging |
  
  
