# Overview of the ARM Architecture 

### About the ARM architecture  

The ARM architecture is a load-store architecture, with a 32-bit addressing range  

ARM, Thumb, and ThumbEE  三种arm指令

### **Processor modes, and privileged and unprivileged software execution**

Processor mode Architectures Mode number
User All 0b10000
FIQ All 0b10001
IRQ All 0b10010
Supervisor All 0b10011
Monitor Security Extensions only 0b10110
Abort All 0b10111
Hyp Virtualization Extensions only 0b11010
Undefined All 0b11011
System ARMv4 and later 0b11111  

User mode is an unprivileged mode, and has restricted access to system resources. All other modes have full access to system resources in the current security state, can change mode freely, and execute software as privileged  

### **ARM registers**  

the following registers are available and accessible in any processor mode  

• 13 general-purpose registers R0- R12.
• One Stack Pointer (SP).
• One Link Register (LR).
• One Program Counter (PC).
• One Application Program Status Register (APSR)  

![image-20221010162357479](F:\typora\image\image-20221010162357479.png)

一共32个寄存器（不算扩展）

除了PC APSR SPSR 寄存器，其他三十个寄存器当为通用寄存器

LR：一般为子程序的返回地址

APSR/CPSR: CPU上下文

SPSR_irq:一般保存CPSR的值

### Register accesses  

16-bit Thumb instructions can access only a limited set of registers.  Most 16-bit Thumb instructions can only access R0 to  .  Only a small number of these instructions can
access R8-R12, SP, LR, and PC.  

All 32-bit Thumb instructions can access R0 to R12, and LR.   

In ARM state, all instructions can access R0 to R12, SP, and LR, and most instructions can also access PC (R15).  

APSR & 通用寄存器的数据交换

 The MRS instructions can move the contents of a status register to a general-purpose register  

MSR instruction to move the contents of a general-purpose register to a status register.  

### Program Counter  

显式使用 mov pc,r0

隐式使用 bx lr 

pc 指向后两条指令  for ARM   指向后两条指令  for Thumb   

### Application Program Status Register （APSR）

It holds copies of the N, Z, C, and V condition flags. The processor uses them to determine whether or not to execute conditional instructions.  	 

### The Q flag  

when saturation has occurred in saturating arithmetic instructions, or when overflow has occurred in certain multiply instructions  .The Q flag is a sticky flag (不会自动清0,必须用读修改写的过程).   

### Current Program Status Register  

• The APSR flags.
• The processor mode.
• The interrupt disable flags.
• The instruction set state (ARM, Thumb, ThumbEE, or Jazelle).
• The endianness state (on ARMv4T and later).  字节序的状态
• The execution state bits for the IT block (on ARMv6T2 and later)   IT状态位  

### Saved Program Status Registers  

stores the current value of the CPSR when an exception is taken

### ARM and Thumb instruction set overview  

All ARM instructions are 32 bits long. Instructions are stored word-aligned  

Thumb instructions are either 16 or 32 bits long. Instructions are stored half-word aligned.  

#### Branch and control  

• Branch to subroutines.
• Branch backwards to form loops.
• Branch forward in conditional structures.
• Make following instructions conditional without branching.
• Change the processor between ARM state and Thumb state.  

#### Data processing   

#### Register load and store  

#### Multiple register load and store  

These instructions load or store any subset of the general-purpose registers from or to memory.  

#### Status register access   

msr mrs

#### Coprocessor  

# Structure of Assembly Language Modules  

### Syntax of source lines in assembly language  

{symbol} {instruction|directive|pseudo-instruction} {;comment}  

symbol :In instructions and pseudo-instructions it is always a label. In some directives it
is a symbol for a variable or a constant. 

Labels are symbolic representations of addresses ,Numeric local labels are a subclass of labels that begin with a number in the range 0-99. Unlike other labels, a numeric local label can be defined many times.   This makes them useful when generating labels with a macro.  

Directives: provide important information to the assembler that either affects the assembly process or affects the final output image.  

### Literals  文本

Assembly language source code can contain numeric, string, Boolean, and single character literals.  

### ELF sections and the AREA directive  

Object files produced by the assembler are divided into sections.  use the
AREA directive to mark the start of a section.  

Use the AREA directive to name the section and set its attributes.   

The following example defines a single read-only section called ARMex that contains code:

AREA ARMex, CODE, READONLY ; Name this block of code ARMex  

### An example ARM assembly language module  

An ARM assembly language module has several constituent parts.
These are:
• ELF sections (defined by the AREA directive).
• Application entry (defined by the ENTRY directive).
• Application execution.
• Application termination.
• Program end (defined by the END directive)  

```asm
AREA ARMex, CODE, READONLY
; Name this block of code ARMex
ENTRY ; Mark first instruction to execute
start
MOV r0, #10 ; Set up parameters
MOV r1, #3
ADD r0, r0, r1 ; r0 = r0 + r1
stop
MOV r0, #0x18 ; angel_SWIreason_ReportException
LDR r1, =0x20026 ; ADP_Stopped_ApplicationExit
SVC #0x123456 ; ARM semihosting (formerly SWI)
END ; Mark end of file
```

Application entry  : declares an entry point to the program.  

Application termination  :the application terminates by returning control to the debugger  

Program end :The END directive instructs the assembler to stop processing this source file.  

# Writing ARM Assembly Language

###  Register usage in subroutine calls  

By convention, you use registers R0 to R3 to pass arguments to subroutines, and R0 to pass a result back to the callers.   

To call subroutines, use a branch and link instruction. The syntax is:

```asm
BL destination ;destination is usually the label,can also be a PC-					   ;relative expression
;1.Places the return address in the link register lr
;Sets the PC to the address of the subroutine
;After the subroutine code has executed you can use a BX LR instruction ;to return.
```

Example  

```asm
    AREA subrout, CODE, READONLY ; Name this block of code
    ENTRY ; Mark first instruction to execute
start MOV r0, #10 ; Set up parameters
	MOV r1, #3
	BL doadd ; Call subroutine
stop MOV r0, #0x18 ; angel_SWIreason_ReportException
	LDR r1, =0x20026 ; ADP_Stopped_ApplicationExit
	SVC #0x123456 ; ARM semihosting (formerly SWI)
doadd ADD r0, r0, r1 ; Subroutine code
	BX lr ; Return from subroutine
	END ; Mark end of file
```

### Load immediate values  

ARM and Thumb instructions can only be 32 bits wide  

use a **MOV** or **MVN** instruction to load a register with an immediate value from a range that depends on the instruction set.  //小范围

load any 32-bit immediate value into a register with two instructions, a
**MOV** followed by a **MOVT**  (**mov32**(伪)= **mov**+**movt**)

use the **LDR** pseudo-instruction to load immediate values into a register.  

### Load immediate values using MOV32  

• A MOV instruction that can load any value in the range 0x00000000 to 0x0000FFFF into a register.
• A MOVT instruction that can load any value in the range 0x0000 to 0xFFFF into the most significant half of a register  

can also use the MOV32 instruction to load addresses into registers by using a label or any PC-relative expression in place of an immediate value.  

### Load immediate values using LDR Rd, =const  

• If the immediate value can be constructed with a single MOV or MVN instruction, the assembler
generates the appropriate instruction.
• If the immediate value cannot be constructed with a single MOV or MVN instruction, the assembler:
— Places the value in a literal pool (a portion of memory embedded in the code to hold constant values).
— Generates an LDR instruction with a PC-relative address that reads the constant from the literal pool.  

### Load addresses into registers  

can load an address into a register either:
• Using the instruction ADR.
• Using the pseudo-instruction ADRL.
• Using the pseudo-instruction MOV32.
• From a literal pool using the pseudo-instruction LDR Rd,=Label.  

### Load addresses to a register using ADR  

The label used with ADR must be within the same code section.   

**A32** Any value that can be produced by rotating an 8-bit value right by any even number of bits within a 32-bit word. The range is relative to the PC.

**32-bit T32 encoding**    ±4095 bytes to a byte, halfword, or word-aligned address.

**16-bit T32 encoding**    0 to 1020 bytes. label must be word-aligned. You can use the ALIGN directive to ensure this.  

```asm
    AREA Jump, CODE, READONLY ; Name this block of code
    ARM ; Following code is ARM code
num EQU 2 ; Number of entries in jump table
    ENTRY ; Mark first instruction to execute
start ; First instruction to call
    MOV r0, #0 ; Set up the three arguments
    MOV r1, #3
    MOV r2, #2
    BL arithfunc ; Call the function
stop
    MOV r0, #0x18 ; angel_SWIreason_ReportException
    LDR r1, =0x20026 ; ADP_Stopped_ApplicationExit
    SVC #0x123456 ; ARM semihosting (formerly SWI)
arithfunc ; Label the function
    CMP r0, #num ; Treat function code as unsigned
    ; integer
    BXHS lr ; If code is >= num then return
    ADR r3, JumpTable ; Load address of jump table  
    LDR pc, [r3,r0,LSL#2] ; Jump to the appropriate routine
JumpTable
    DCD DoAdd
    DCD DoSub
DoAdd
    ADD r0, r1, r2 ; Operation 0
    BX lr ; Return
DoSub
    SUB r0, r1, r2 ; Operation 1
    BX lr ; Return
	END ; Mark the end of this file
```

```asm
EQU ;You use it to give a value to a symbol. In this example, it assigns the value 2 to num. When num is used elsewhere in the code, the value 2 is substituted. 
DCD ;Declares one or more words of store. In this example, each DCD stores the address of a routine that handles a particular clause of the jump table.
LDR The LDR  ;PC,[R3,R0,LSL#2] instruction loads the address of the required clause of the jump table into the PC.
```

### Load addresses to a register using ADRL  

The label used with ADRL must be within the same code section  

A32
Any value that can be generated by two ADD or two SUB instructions. That is, any value that can be produced by the addition of two values, each of which is 8 bits rotated right by any even number of bits within a 32-bit word. The range is relative to the PC.
32-bit T32 encoding
±1MB to a byte, halfword, or word-aligned address.
16-bit T32 encoding
ADRL is not available.  



### Load addresses to a register using LDR Rd, =label  

可以使用不同分区的标号

```asm
LDR rn [pc, #offset_to_literal_pool]
    ; load register n with one word
    ; from the address [pc + offset]
```

```asm
    AREA LDRlabel, CODE, READONLY
    ENTRY ; Mark first instruction to execute
start
    BL func1 ; Branch to first subroutine
    BL func2 ; Branch to second subroutine
stop
    MOV r0, #0x18 ; angel_SWIreason_ReportException
    LDR r1, =0x20026 ; ADP_Stopped_ApplicationExit
    SVC #0x123456 ; ARM semihosting (formerly SWI)
func1
    LDR r0, =start ; => LDR r0,[PC, #offset into Literal Pool 1]
    LDR r1, =Darea + 12 ; => LDR r1,[PC, #offset into Literal Pool 1]
    LDR r2, =Darea + 6000 ; => LDR r2,[PC, #offset into Literal Pool 1]
    BX lr ; Return
    LTORG ; Literal Pool 1
func2
    LDR r3, =Darea + 6000 ; => LDR r3,[PC, #offset into Literal Pool 1]
    ; (sharing with previous literal)
    ; LDR r4, =Darea + 6004 ; If uncommented, produces an error because
    ; Literal Pool 2 is out of range.
    BX lr ; Return
    Darea SPACE 8000 ; Starting at the current location, clears
    ; a 8000 byte area of memory to zero.
    END ; Literal Pool 2 is automatically inserted
    ; after the END directive.
    ; It is out of range of all the LDR
    ; pseudo-instructions in this example.
```

```asm
DCB
;The DCB directive defines one or more bytes of store. In addition to integer values, DCB accepts quoted strings. Each character of the string is placed in a consecutive byte.
```

```asm
    AREA StrCopy, CODE, READONLY
    ENTRY ; Mark first instruction to execute
start
    LDR r1, =srcstr ; Pointer to first string
    LDR r0, =dststr ; Pointer to second string
    BL strcopy ; Call subroutine to do copy
stop
    MOV r0, #0x18 ; angel_SWIreason_ReportException
    LDR r1, =0x20026 ; ADP_Stopped_ApplicationExit
    SVC #0x123456 ; ARM semihosting (formerly SWI)
strcopy
    LDRB r2, [r1],#1 ; Load byte and update address
    STRB r2, [r0],#1 ; Store byte and update address
    CMP r2, #0 ; Check for zero terminator
    BNE strcopy ; Keep going if not
    MOV pc,lr ; Return
    AREA Strings, DATA, READWRITE
    srcstr DCB "First string - source",0
    dststr DCB "Second string - destination",0
    END
```

```asm
LDRB r2,[r1],#1 
    ;loads R2 with the contents of the address pointed to by R1 and then increments R1 by 1.
 
LDRB r2,[r1],#1    ;store
```

### MOV  

MOV32{cond} Rd, expr
where:
cond
is an optional condition code.
Rd
is the register to be loaded. Rd must not be SP or PC.
expr
can be any one of the following:
symbol
A label in this or another program area.
\#constant
Any 32-bit immediate value.
symbol + constant
A label plus a 32-bit immediate value.  

### Load and store multiple register instructions in ARM and Thumb  

寄存器和内存交互

```asm
LDR 

LDR R1, [R2,#1]  , *(r2+1)->r1
    
LDR R1, [R2,#1]! , *(r2+1)->r1 ,r2=r2+1
 
LDR R1, [R2],#1  , (*r2) ->r1 ,r2+1
```

### Load and store multiple register instructions  

```asm
LDM R0 , {R1-R4} ;  *(R0)->R1 , *(R0+4)->R2, ...
STM  ;mem->reg
PUSH ;多个寄存器入栈  
POP;    相当于 LDM SP! {R0,R3}
```

When the base register is updated to point to the next block in memory, this is called write back ,寄存器后加！实现

递减 full 型堆栈

###  Stack operations for nested subroutines  

```asm
subroutine PUSH {r5-r7,lr} ; Push work registers and lr
; code
BL somewhere_else
; code
POP {r5-r7,pc} ; Pop work registers and pc
```

 In ARMv4T systems, you cannot change state by popping directly into PC.  

### Block copy with LDM and STM  

#### Example of block copy without LDM and STM  

```asm
	AREA Word, CODE, READONLY ; name the block of code
num EQU 20 ; set number of words to be copied
	ENTRY ; mark the first instruction called
start
    LDR r0, =src ; r0 = pointer to source block
    LDR r1, =dst ; r1 = pointer to destination block
    MOV r2, #num ; r2 = number of words to copy
wordcopy
    LDR r3, [r0], #4 ; load a word from the source and
    STR r3, [r1], #4 ; store it to the destination
    SUBS r2, r2, #1 ; decrement the counter
    BNE wordcopy ; ... copy more
stop
    MOV r0, #0x18 ; angel_SWIreason_ReportException
    LDR r1, =0x20026 ; ADP_Stopped_ApplicationExit
    SVC #0x123456 ; ARM semihosting (formerly SWI)
    AREA BlockData, DATA, READWRITE
src DCD 1,2,3,4,5,6,7,8,1,2,3,4,5,6,7,8,1,2,3,4
dst DCD 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
	END
```

#### Example of block copy using LDM and STM  

```asm
MOVS r3 ,r2 ,LSR #3
ANDS r2, r2, #7
```

```asm
	AREA Block, CODE, READONLY ; name this block of code
num EQU 20 ; set number of words to be copied
	ENTRY ; mark the first instruction called
start
    LDR r0, =src ; r0 = pointer to source block
    LDR r1, =dst ; r1 = pointer to destination block
    MOV r2, #num ; r2 = number of words to copy
    MOV sp, #0x400 ; Set up stack pointer (sp)
blockcopy
    MOVS r3,r2, LSR #3 ; Number of eight word multiples
    BEQ copywords ; Fewer than eight words to move?
    PUSH {r4-r11} ; Save some working registers
octcopy
    LDM r0!, {r4-r11} ; Load 8 words from the source
    STM r1!, {r4-r11} ; and put them at the destination
    SUBS r3, r3, #1 ; Decrement the counter
    BNE octcopy ; ... copy more
    POP {r4-r11} ; Don't require these now - restore
    			 ; originals
copywords
    ANDS r2, r2, #7 ; Number of odd words to copy
    BEQ stop ; No words left to copy?
wordcopy
    LDR r3, [r0], #4 ; Load a word from the source and
    STR r3, [r1], #4 ; store it to the destination
    SUBS r2, r2, #1 ; Decrement the counter
    BNE wordcopy ; ... copy more
stop
	MOV r0, #0x18 ; angel_SWIreason_ReportException
	LDR r1, =0x20026 ; ADP_Stopped_ApplicationExit
	SVC #0x123456 ; ARM semihosting (formerly SWI)
	AREA BlockData, DATA, READWRITE
src DCD 1,2,3,4,5,6,7,8,1,2,3,4,5,6,7,8,1,2,3,4
dst DCD 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
	END
```

### The Read-Modify-Write operation  

``` asm
VMRS r10,FPSCR ; copy FPSCR into the general-purpose r10
BIC r10,r10,#0x00370000 ; clear STRIDE bits[21:20] and LEN bits[18:16] 
ORR r10,r10,#0x00030000 ; set bits[17:16] (STRIDE =1 and LEN = 4)
VMSR FPSCR,r10 ; copy r10 back into FPSCR
```

• BIC to clear to 0 only the bits that must be cleared. (读修改写操作)
• ORR to set to 1 only the bits that must be set   

### Use of macros  

```asm
	MACRO
$label TestAndBranch $dest, $reg, $cc  ;原型说明
$label CMP $reg, #0       ；实际展开
	B$cc $dest
	MEND
```

Test-and-branch macro example  

```asm
;This macro can be invoked as follows:
test TestAndBranch NonZero, r0, NE
...
...
NonZero

;After substitution this becomes:
test CMP r0, #0
BNE NonZero
...
...
NonZero
```

Unsigned integer division macro example  

```asm
	MACRO
$Lab DivMod $Div,$Top,$Bot,$Temp
    ASSERT $Top <> $Bot ; Produce an error message if the
    ASSERT $Top <> $Temp ; registers supplied are
    ASSERT $Bot <> $Temp ; not all different
    IF "$Div" <> ""
        ASSERT $Div <> $Top ; These three only matter if $Div
        ASSERT $Div <> $Bot ; is not null ("")
        ASSERT $Div <> $Temp ;
	ENDIF
$Lab
    MOV $Temp, $Bot ; Put divisor in $Temp
    CMP $Temp, $Top, LSR #1 ; double it until
90 MOVLS $Temp, $Temp, LSL #1 ; 2 * $Temp > $Top
    CMP $Temp, $Top, LSR #1
    BLS %b90 ; The b means search backwards
    IF "$Div" <> "" ; Omit next instruction if $Div
					; is null
		MOV $Div, #0 ; Initialize quotient
	ENDIF
91 CMP $Top, $Temp ; Can we subtract $Temp?
    SUBCS $Top, $Top,$Temp ; If we can, do so
    IF "$Div" <> "" ; Omit next instruction if $Div
					; is null
		ADC $Div, $Div, $Div ; Double $Div
	ENDIF
    MOV $Temp, $Temp, LSR #1 ; Halve $Temp,
    CMP $Temp, $Bot ; and loop until
    BHS %b91 ; less than divisor	
    MEND
```

```asm
ratio DivMod R0,R5,R4,R2
```

```asm
    ASSERT r5 <> r4 ; Produce an error if the
    ASSERT r5 <> r2 ; registers supplied are
    ASSERT r4 <> r2 ; not all different
    ASSERT r0 <> r5 ; These three only matter if $Div
    ASSERT r0 <> r4 ; is not null ("")
    ASSERT r0 <> r2 ;
ratio
    MOV r2, r4 ; Put divisor in $Temp
    CMP r2, r5, LSR #1 ; double it until
90 MOVLS r2, r2, LSL #1 ; 2 * r2 > r5
    CMP r2, r5, LSR #1
    BLS %b90 ; The b means search backwards
    MOV r0, #0 ; Initialize quotient
91 CMP r5, r2 ; Can we subtract r2?
    SUBCS r5, r5, r2 ; If we can, do so
    ADC r0, r0, r0 ; Double r0
    MOV r2, r2, LSR #1 ; Halve r2,
    CMP r2, r4 ; and loop until
    BHS %b91 ; less than divisor
```

### Instruction and directive relocations

 汇编器会在未知的label位置放一个重定位指示符，linker 会将label变为地址

The assembler emits a relocation for these instructions if the label used meets any of the following
requirements, as appropriate for the instruction type:
• The label is WEAK.
• The label is not in the same AREA.
• The label is external to the object (IMPORT or EXTERN).
For B, BL, and BX instructions, the assembler emits a relocation also if:
• The label is a function.
• The label is exported using EXPORT or GLOBAL.  

```asm
IMPORT sym ; sym is an external symbol
DCW sym ; Because DCW only outputs 16 bits, only the lower
; 16 bits of the address of sym are inserted at
; link-time.
```

### Conditional execution in ARM state 

Example **conditional instructions** to control execution  

```asm
; flags set by a previous instruction
LSLEQ r0, r0, #24 
ADDEQ r0, r0, #2
;…
```

Example **conditional branch** to control execution  

```asm
; flags set by a previous instruction  //更加高效
BNE over
LSL r0, r0, #24
ADD r0, r0, #2
over
;…
```

### Conditional execution in Thumb state  

```asm
;IT指令
; flags set by a previous instruction
ITT EQ
LSLEQ r0, r0, #24
ADDEQ r0, r0, #2
;…
```

### Updates to the condition flags  

Instructions with the optional S suffix update the flags.  

The instructions CMP, CMN, TEQ, and TST always update the flags.  

N
Set to 1 when the result of the operation is negative, cleared to 0 otherwise.
Z
Set to 1 when the result of the operation is zero, cleared to 0 otherwise.
C
Set to 1 when the operation results in a carry, or when a subtraction results in no borrow, cleared
to 0 otherwise.
V
Set to 1 when the operation causes overflow, cleared to 0 otherwise.

C is set in one of the following ways:
• For an addition, including the comparison instruction CMN, C is set to 1 if the addition produced a
carry (that is, an unsigned overflow), and to 0 otherwise.
• For a subtraction, including the comparison instruction CMP, C is set to 0 if the subtraction produced a
borrow (that is, an unsigned underflow), and to 1 otherwise.
• For non-addition/subtractions that incorporate a shift operation, C is set to the last bit shifted out of
the value by the shifter.
• For other non-addition/subtractions, C is normally left unchanged, but see the individual instruction
descriptions for any special cases.  

### Condition code suffixes and related flags  

Suffix Flags Meaning
EQ Z set Equal
NE Z clear Not equal
CS or HS C set Higher or same (unsigned >= )
CC or LO C clear Lower (unsigned < )
MI N set Negative
PL N clear Positive or zero
VS V set Overflow
VC V clear No overflow
HI C set and Z clear Higher (unsigned >)
LS C clear or Z set Lower or same (unsigned <=)
GE N and V the same Signed >=
LT N and V differ Signed <
GT Z clear, N and V the same Signed >
LE Z set, N and V differ Signed <=
AL Any Always. This suffix is normally omitted.  

```asm
ADD r0, r1, r2 ; r0 = r1 + r2, don't update flags
ADDS r0, r1, r2 ; r0 = r1 + r2, and update flags
ADDSCS r0, r1, r2 ; If C flag set then r0 = r1 + r2,
; and update flags
CMP r0, r1 ; update flags based on r0-r1.
```

