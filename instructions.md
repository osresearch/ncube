## 4.7 Instruction Formats and Addressing Modes

The processor is designed to be as simple and symmetric as possible. Most
instructions work on all supported data types; the General registers
are interchangeable in all operations; all address modes work with
all instructions including Branches. An instruction consists of an
operation code (opcode) followed by zero, one or two address fields. The
representation of a two address instruction in memory is illustrated
below:

```
  | OPCODE | REFERENCE 1 | REFERENCE 2|
Low address                        High address
```

In the physical representation shown above REFERENCE 2 is both one of the
operands and the result. For example, if the OPCODE indicated Subtract
then the operation performed would be:

```
REFERENCE 1 - REFERENCE 2 -> REFERENCE 2
```

The assembly language operand ordering convention is the same. Thus,
if a Subtract operation is written

```
SUBW A,B
```

the operation performed is

```
(A-B) -> B
```

The order of address evaluation is from the low address so that the
address for A is evaluated before the address for B.

### 4.7.1 Opcode Formats

All opcodes are one byte long and each operation type group has at least
one reserved code for future expansion. The byte is divided into two
fields of four bits each. The first field, TP, specifies the length and
type of the operands (e.g. 8 bit integer, 32 bit real) and the second
field, OP, determines the operation and number of operands (e.g. Add--2
operands, Call--one operand). Each of the operations is described in
detail in chapter 4.8 but most are evident from their name in the opcode
table below. The first field is represented horizontally with the even
values above the odd values. The second field is displayed vertically
and is repeated twice.

```
 Opcode | OP TP |
        7       0
```

```
                OPCODE MAP
                    TP

    B    H    W        R    L
    0    2    4    6   8    A    C   E
  0 MOVB MOVH MOVW RES MOVR MOVL RES RES
  1 NEGB NEGH NEGW RES NEGR NEGL RES REP
  2 SBRB SBRH SBRW RES SBRR SBRL RES REPZ
  3 CMPB CMPH CMPW RES CMPR CMPL RES REPNZ
  4 ADDB ADDH ADDW RES ADDR ADDL RES TRAP
  5 ADCB ADCH ADCW RES SQTR SQTL RES RES
  6 SUBB SUBH SUBW RES SUBR SUBL RES RES           OP
  7 SBBB SBBH SBBW RES SGNR SGNL RES RES
  8 MULB MULH MULW RES MULR MULL RES RES
  9 DVRB DVRH DVRW RES DVRR DVRL RES RES
  A REMB REMH REMW RES REMR REML RES RES
  B DIVB DIVH DIVW RES DIVR DIVL RES RES
  C BITB BITH BITW RES RES  RES  RES RES
  D RES  RES  RES  RES RES  RES  RES RES
  E RES  RES  RES  RES RES  RES  RES RES
  F RES  RES  RES  RES ESC  ESC  ESC RES

    1    3    5    7   9    B    D   F
  0 SFTB SFTH SFTW RES CVBR NOP  RES BG
  1 SFAB SFAH SFAW RES CVHR CLC  RES BLE
  2 ROTB ROTH ROTW RES CVWR STC  RES BGU
  3 FFOB FFOH FFOW RES CVLR CMC  RES BLEU
  4 ANDB ANDH ANDW RES CVBL ERON RES BGE
  5 ORB  ORH  ORW  RES CVHL EROF RES BL
  6 XORB XORH XORW RES CVWL BKPT RES BGEU         OP
  7 NOTB NOTH NOTW RES CVRL RSET RES BLU
  8 ADCD RES  LDPR RES CVBW EI   RES BNE
  9 SBBD RES  STPR RES CVHW DI   RES BE
  A RES  RES  LCNT RES CVWB RES  RES BNV
  B RES  RES  LPTR RES CVWH RES  RES BV
  C RES  RES  BCNT RES CVRW RETI RES CALL
  D RES  RES  BPTR RES CVLW WAIT RES JMP
  E RES  RES  MOVA RES RES  RET  RES RETP
  F ESC  ESC  ESC  ESC ESC  ESC  ESC ESC
```

The Opcode Map illustrates a number of symmetries that are explained in
the table below.

```
                          OPERANDS
OPERATIONS                (#, TYPE)              COLUMNS
Special                   0                      11
Special                   1, Byte or Halfword    14
Branch                    1, Word (Address)      15
Conversion                2, Mixed               9
Byte                      2, Byte                0,1
Halfword                  2, Halfword            2,3
Word                      2, Word                4,5
Reserved for Double word  *,*                    6,7
Real                      2, Real                8
Longreal                  2, Longreal            10
Reserved for Tempreal     *,*                    12
Reserved (Arbitrary)      *,*                    13
```

### 4.7.2 Addressing Modes

If an instruction has operands, the address fields always have at
least one byte. The first byte, called the Mode Specifier, encodes the
addressing mode and for most of the instructions the first four bits
specify the general register to be used in the address evaluation while
the next four bits indicate the mode. The format is as shown below:

TODO: figure

The modes are listed below with their encodings and mnenomics.

```
Addressing Mode Table
Mode Name                          Encoding   Mnemonic
Literal                            0,1,2,3 literal #n
Register Direct                    C       Rn     Rn
Register Indirect                  4       Rn     (Rn)
Autodecrement                      D       Rn     -(Rn)
Autoincrement                      6       Rn     (Rn)+
Autoincrement Indirect             7       Rn     @(Rn)+
Autoskip                           5       Rn     (Rn)++
Offset+Register Indirect
  Byte Offset                      8       Rn     A(Rn)
  Halfword Offset                  9       Rn     A(Rn)
  Word Offset                      A       Rn     A(Rn)
  (Word Offset+Register) Indirect  B       Rn     @A(Rn)
RESERVED                           E

Special Modes: no General Register
Offset+PC
  Byte Offset+PC                   F       0      S(PC)
  Halfword Offset+PC               F       1      A(PC)
  Word Offset+PC                   F       2      S(PC)
  (Word Offset+PC)Indirect         F       3      @A(PC)
  Offset+SP Byte Offset+SP         F       4      S(SP)
  Halfword Offset+SP               F       5      S(SP)
  Word Offset+SP                   F       6      A(SP)
  (Word Offset+SP)Indirect         F       7      @A(SP)
  Direct Byte Offset               F       8      A
  Halfword Offset                  F       9      A
  Word Offset                      F       A      A
(Word)Indirect                     F       B      @A
Push/Pop                           F       C      STK
Immediate                          F       D      #n
RESERVED                           F       E
ESCAPE                             F       F
```

The assembler will chose the shortest reference form possible. The
addressing modes are described in detail below. First note the following:

* addresses of multibyte operands refer to the low order byte of
the operand.

* offsets are sign-extended to 32 bits before being used in effective
address calculation

* for Branch and Call instructions in Literal or Immediate mode the
value is added to the PC; for Register Direct mode the register contents
are added to the PC; for all other modes the address of the operand
simply replaces the PC.

```
LITERAL 00xxxxxx (Mode=0,1,2,3)
```

Since the encoding for literal includes modes 0,1,2,3, there are six
bits for the definition of the literal value. When an integer operand
is expected the six bits are treated as a standard 2's complement
integer between -32 and +31. And when the instruction indicates that
the literal is a real value, the integer value is converted implicitly
(without round off error) to the equivalent floating point value.

If a literal is used in a Branch, Call or Move Address instruction, the
literal is added to the PC (i.e. a relative Branch or Call results). If
a literal (or immediate) is used as a destination an Operand Error is
signaled.

```
Register Direct | C Rn |
```


In this mode the operand is contained in the indicated register. The
value is interpreted according to the instruction: real for floating
point instructions, integer for integer operations and bit string for
logical instructions. If a longreal operand is expected the low order
part is in Rn and the high order part in Rn+1. When a byte or halfword is
moved to a register it is sign-extended.

```
Register Indirect  | 4 Rn |
```

The indicated register contains the address of the low order byte of
the operand.

```
Autodecrement  | D Rn |
```

The indicated register is decremented by the length in bytes of the
operand and then the contents becomes the address of the operand. This
mode can be used to build a software stack or to access consecutive array
elements.

```
Autoincrement   | 6 Rn |
```

The data addressed by Rn is first accessed and then Rn is incremented
by the number of bytes in the operand. This mode is used to step through
arrays and, with Autodecrement, to build software stacks.

```
Autoincrement Indirect | 7 Rn |
```

The register Rn points to a 32 bit value that is the address of the
operand. After the operand is accessed Rn is incremented by four, since
addresses are four bytes long.

```
Autoskip   | 5 Rn |
```

After the operand addressed by the contents of Rn is fetched, the
value in Rn+1 is added to Rn. (If n=15 then n+1=0.) This mode allows
for automatically skipping through an array by an amount (in Rn+1) that
can be calculated during program execution. For example if a matrix is
stored by columns this mode permits automatic references to successive
row elements.

```
Offset + Register  | 8,9,A Rn |
A = byte, B = halfword, C = word
```


This mode calculates the address of the operand by adding the value in
Rn to the offset which is a signed integer whose length is determined by
the mode setting (A=byte, B=halfword, C=word). The offset immediately
follows the mode indicator and is sign- extended for the effective
address calculation. These modes are also available for the PC and SP
in place of a general register (see below).

```
(Offset + Register) Indirect   | B Rn |
Word offset
```

The contents of Rn are added to the offset (in this mode only a 32 bit
offset is allowed) and the 32 bit value at that address is the address
of the operand. This mode is also available with either PC or SP instead
of a general register (see below).

```
Offset + PC  | F 0,1,2 |
0 = byte, 1 = halfword, 2 = word
```

The address is calculated by adding the address of the instruction
(the value of PC before the current instruction is executed) to
the sign-extended value of the offset which can be a byte, halfword
or word. This mode is used to access operands relative to PC and with
branch instructions to jump relative to PC. (The Literal mode with branch
instructions also is relative to PC.) This permits compiling position
independent code.

```
(Offset + PC) Indirect   | F 3 | ????
Word offset
```

The address of the instruction (the contents of PC) is added to the
word offset and the 32 bit value at that address is the address to the
operand.

```
Offset + SP  | F 4,5,6 |
4 = byte, 5 = halfword, 6 = word
```

The address is calculated by adding the SP and the sign extended
offset. The offset can be a byte, halfword, or word. This mode is
often used to access local variables in an activation record on the
stack.

```
(Offset + SP) Indirect   | F 7 | Word offset
```

The SP and the word offset are added together and the 32 bit value at
that address is the address of the operand.


```
Direct  | F 8,9,A |
8 = byte, 9 = halfword, A = word
```

The address is the unsigned value of the offset (byte, halfword or word
depending on the mode) that follows the mode specifier.


```
Indirect  | F B | Word offset
```

The word that follows the mode specifier points to a 32 bit value that is the address of the operand.


```
Immediate   | F C | Value
```

In this mode the operand follows the mode specifier for arithmetic and
logical operators the length and type of the value is indicated by the
instruction. Thus, `ADDB` (Add Byte) will assume an 8 bit signed integer
while `MULL` (Multiply Longreal) will expect to find a 64 bit floating
point operand as the "value". An immediate operand used with a branch or
move address instruction causes an invalid operand fault. If this mode is
used as the destination (the second address in a two address instruction)
an Operand error is signaled.

```
Push Pop   | F D |
```

When this mode is the first specifier it takes the operand from the
top of the stack and the increments ("pops") SP by the length of the
operand. So the instruction

```
ADDR SP,mem
```

will use a 32 bit real value from the top of the stack as the first
operand, pop the stack and store the result as "mem". Similarly a

```
MOVH SP,mem
```

will move the halfword on the top of the stack to "mem" and pop the
stack. When used as the second specifier, the second operand and
the result come from the stack top. Thus with arithmetic and logical
instructions there is no change in SP. However,

```
MOVR mem,SP
```

will decrement SP by four (the length of the operand) and move the real
value at "mem" to the top of the stack. When this mode is used in both
specifiers then the classical stack operations result: both operands are
popped off the stack, the operation performed and the result is pushed
back on the stack. In the case of Divide and Subtract the operand at the
top of the stack is the dividend and subtrahend respectively. If both
specifiers are SP for a Move instruction, only the flags are affected.

* [Instruction set](instructions.md) (Section 4.8)
## 4.8 Instruction Set

### 4.8.1 Instruction Set Details

The instructions are listed alphabetically (by mnemonic) and are
grouped according to operation (e.g. all the Ad instructions are grouped
together).

The memory format of all of the instructions is shown below. The source
and destination specifiers are optional. While most instructions have
two addresses, there are a few with zero or one address.

```
| OPCODE  REFERENCE 1 (src)  REFERENCE 2 (dsrc, des) |
low address in memory
```

The source (src) address is always evaluated first and all addressing
operations (e.g. autodecrement) are performed before the destination
(dsrc, des) address is evaluated. (In the above notation "dsrc" refers
to the operand before the operation is performed and "des" refers to the
contents of that address after the operation.) This does not apply to
stack addressing modes where the SP at the beginning of the instruction
is always used. Any addressing mode that refers to the PC or (SP) uses
the value of the PC (opr SP) at the beginning of the instruction. The
source operand is never changed except when using the stack addressing
mode. If an instruction with byte or halfword operands references a
general register, the high order part of the data is ignored if it is
a source and if it is a destination the high order part is sign extended.

The unique exception conditions for each instruction are included in the
information below. There are a set of exceptions that are independent
of the particular instruction:

* (1) Memory error (ECC or Correctable ECC)
* (2) Timeout
* (3) Operand error (reserved addressing mode,literal or immediates the
destination)
* (4) Address error (address value greater than 2^17-1)
* (5) Stack overflow

The result stored at the destination of a floating point instruction is
described below. The result is stored before the exception is signaled
by an interrupt (except for Zero Divide and Invalid).

* (1) Inexact: the correctly rounded result
* (2) Underflow: the correctly rounded fraction but with the exponent
increased by the bias
* (3) Zero Divide: no result is stored; the destination is not changed
* (4) Overflow: the correctly rounded fraction but with the exponent
decreased by the bias
* (5) Invalid: no result is stored; the destination is not changed.

It is important to remember that the Negative (N) Flag is always set
according to the sign of the correct result. Thus on integer overflow,
the destination may appear positive even when N indicates negative.

### 4.8.2 Instruction Definitions

#### ADC - ADD WITH CARRY
The Carry and source values are added to the destination and the result
replaces the destination.

Opcodes:
```
  50     ADCB       ADd with Carry Byte
  52     ADCH       ADd with Carry Halfword
  54     ADCW       ADd with Carry Word
```
Assembler Syntax:
```
  ADC{B,H,W} src,des
```
Operation:
```
  src + dsrc + Carry → des
```
Flags:
  * `C` ← carry from most significant bit
  * `N` ← des < 0
  * `Z` ← des = 0
  * `V` ← Integer overflow
  * `U` ← 0

Exceptions:
  * Integer overflow


#### ADCD - ADD WITH CARRY DECIMAL
The Carry and source value (treated as a two decimal value) are added to
the destination (also considered as a two decimal value) and the result
replaces the destination. No check for invalid BCD encoding is made.

Opcode:
```
  81     ADCD       ADd with Carry Decimal
```
AssemblerSyntax:
```
  ADCD src,des
```
Operation:
```
  src + dsrc + Carry → des
```
Flags:
  * `C` ← Carry out of high order digit
  * `N` ← 0
  * `Z` ← des = 0
  * `V` ← 0
  * `U` ← 0

Exceptions:
  * none


#### ADD - ADD
The source is added to the destination and the result is stored at the
address of the destination.

Opcodes:
```
  40     ADDB       ADD Byte
  42     ADDH       ADD Halfword
  44     ADDW       ADD Word
  48     ADDR       ADD Real
  4A     ADDL       ADD Longreal
```
AssemblerSyntax:
```
  ADD{B,H,W,R,L} src,des
```
Operation:
```
  src + dsrc → des
```
Flags:  (Integer Operations: `ADDB,ADDH,ADDW`)
  * `C` ← carry from most significant bit
  * `N` ← des < 0
  * `Z` ← des = 0
  * `V` ← Integer overflow
  * `U` ← 0

Flags:  (Floating Point Operations: `ADDR,ADDL`)
  * `C` ← des < 0
  * `N` ← des < 0
  * `Z` ← des = 0
  * `V` ← 0
  * `U` ← 0
  * `IX` ← des rounded
  * `UF` ← des underflowed
  * `FZ` ← 0
  * `OF` ← des overflowed
  * `IN` ← dsrc or src = Nan

Exceptions:
  * Integer overflow
  * Inexact
  * Underflow
  * Overflow
  * Invalid


#### AND - AND
The destination operand is anded with the source and the result is stored
at the destination address.

Opcodes:
```
  41     ANDB       AND Byte
  43     ANDH       AND Halfword
  45     ANDW       AND Word
```
AssemblerSyntax:
```
  AND{B,H,W} src,des -
```
Operation:
```
  src AND dsrc → des
```
Flags:
 * `C` ← C
 * `N` ← des < 0
 * `Z` ← des = 0
 * `V` ← 0
 * `U` ← 0

Exceptions:
  * none


#### B - Branch
The Branch instructions are relative in the literal immediate
and register direct modes and use the value of the PC at the
beginning of the instruction. In all other modes the address of
the source operand replaces the PC. The Invalid exception results
when comparison accesses at least one Nan and a signed branch is
performed on the result. The unsigned branches should be used
for the predicates defined in the IEEE Floating Point Standard
that must not fault. The Repeat Mode is reset after decrementing
the counter and testing the termination condition so that if a
REPeat instruction precedes a branch they act together like a
"loop" instruction.

Opcodes:
```
  DF     JMP     Unconditional       JuMP unconditional
  BF     BV      V=1                 Branch on oVerflow
  AF     BNV     V=0                 Branch on Not oVerflow
  9F     BE      Z=1                 Branch on Equal
  8F     BNE     Z=0                 Branch on Not Equal
  0F     BG      (N or Z)=0          Branch on Greater
  4F     BGE     N=0                 Branch on Greater or Equal
  5F     BL      N=1                 Branch on Less
  1F     BLE     (N or Z)=1          Branch on Less or Equal
  2F     BGU     (C or Z)=0 or U=1   Branch on Greater Unsigned
  6F     BGEU    C=0 or U=1          Branch on Greater or Equal Unsigned
  7F     BLU     C=1 or U=1          Branch on Less Unsigned
  3F     BLEU    (C or Z)=1 or U=1   Branch on Less or Equal Unsigned
```
AssemblerSyntax:
```
 JMP src
 B{V,NV,E,NE,GU,GE,L,LE,G,GEU,LU,LEU} src
```
Operation:
```
 If Condition is True then
    Literal or Immmediate Mode: PC ← PC + src
    Register Direct mode: PC ← PC + content(reg)
    Other Modes: PC ← address of (src)
```
Flags:
  * No flags are changed except `IN` (INvalid exception). IN is set
  only when U=1 on BG, BGE, BL, BLE. The Repeat Mode (REP) is reset
  (REP ← 00) after decrementing the counter and checking the condition
  (see below).

Exceptions:
  * Invalid (BG,BGE,BL,BLE when U = 1);
  * Illegal Address (Immediate mode)


#### BCNT - BROADCAST COUNT
The Output Count registers whose numbers correspond to bit positions in
des that are set to one are loaded with the src value. The Output
Count registers are numbered 32,33. . .,41,63 so the bit positions
in des are understood to be offset by 32. Both src and des are
Word values.

Opcode:
```
  C5     BCNT       Broadcast CouNT
```
AssemblerSyntax:
```
 BCNT src,des
```
Operation:
```
 src → des MASK (All Output Count Register #'s)
```
Flags:
  * no changes

Exceptions:
  * none


#### BIT - BIT TEST
The Z Flag is set to 0 if all the bits of src that are masked by dsrc
are 0. Neither src nor dsrc is changed.

Opcodes:
```
  61     BITB       BIT test Byte
  63     BITH       BIT test Halfword
  65     BITW       BIT test Word
```
AssemblerSyntax:
```
  BIT{B,H,W} src,dsrc
```
Operation:
```
  src AND dsrc
```
Flags:
  * C ← C
  * N ← (src AND dsrc) < 0
  * Z ← (src AND dsrc) = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


#### BKPT - BREAKPOINT
This one byte instruction is used by a debugger to set breakpoints
in a user's program.

Opcode:
```
  6B     BKPT       BreaKPoinT
```
AssemblerSyntax:
```
 BKPT
```
Operation:
```
  generate interrupt 2:
    stack ← PS
    stack ← PC
    PC ← Word at location 16
    PS ← Word at location 20
```
Flags:
  * all flags set according to the new PS

Exceptions:
  * none

#### BPTR - BROADCAST POINTER
Opcode:
```
 D5      BPTR       Broadcast PoinTeR
```
AssemblerSyntax:
```
 BPTR src.des
```
Operation:
```
 src → des MASK (All Output Register #'s)
```
Flags:
  * no changes
Description:
   The Output Registers whose numbers correspond with the bit positions
   in des that are set are loaded with the src. This instruction sets
   up a group of Output Pointer registers to address a memory area
   containing a message to be broadcast. The Pointer registers should
   be set up before the Count registers (BCNT) are loaded. Both src
   and des are Word values.

Exceptions:
  * none


#### CALL - CALL
The current value of the Program Counter (PC) is pushed on the stack
and by loading the PC with a new value a branch to a subroutine is
taken. If the CALL is preceded by a REPEAT instruction the counter is
decremented and the termination condition is checked. The Repeat Mode is
reset (REP ← 00) and if termination is not reached then the return
address that is pushed on the stack points to the REPEAT instruction. If
termination is reached the CALL instruction is skipped. This enables
the processor to execute multiple CALLs. If there is no preceding
REPEAT then the saved return address points to the beginning of the
instruction following the CALL. If the addressing mode is Literal,
Immediate or Register Direct the call is relative and uses the value
of PC at the beginning of the CALL instruction.

Opcode:
```
  CF     CALL       CALL
```
AssemblerSyntax:
```
 CALL src
```
Operation:
```
  Literal or Immediate Mode:
    stack ← PC   PC ← PC + src
  Register Direct Mode:
    stack ← PC   PC ← PC + content (reg)
  Other Modes:
    stack ← PC   PC ← address of (src)
```
Flags:
  * no changes except REP ← 00 (see below)

Exceptions:
  * Illegal Address (Immediate mode)


#### CLC - CLEAR CARRY
The Carry Flag is set to zero.

Opcode:
```
  1B     CLC        CLear Carry
```
AssemblerSyntax:
```
 CLC
```
Operation:
```
  C ← 0
```
Flags:
  * C ← 0   no other changes

Exceptions:
  * none


#### CMC - COMPLEMENT CARRY
The Carry Flag is reversed.

Opcode:
```
  3B     CMC        CoMplement Carry
```
AssemblerSyntax:
```
 CMC
```
Operation:
```
  C ← not(C)
```
Flags:
  * C ← not(C)   no other changes

Exceptions:
  * none


#### CMP - COMPARE
The value src is compared to dsrc and the appropriate flags are set for
subsequent conditional branching. Neither src nor dsrc is changed. The
Carry flag is set by the Floating Point comparisons so that the Unsigned
branches can be used for the Unordered predicates defined in the
IEEE Floating Point Standard. Also if either src or dsrc is Nan the
appropriate Invalid exception is signaled by the branch instruction.

Opcodes:
```
  30     CMPB       CoMPare Byte
  32     CMPH       CoMPare Halfword
  34     CMPW       CoMPare Word
  38     CMPR       CoMPare Real
  3A     CMPL       CoMPare Longreal
```
AssemblerSyntax:
```
 CMP{B,H,W,R,L} src,dsrc
```
Operation:
```
  src - dsrc → tem
```
Flags:  (Integer Operations: CMPB,CMPH,CMPW)
  * C ← src < (unsigned) dsrc
  * N ← tem < 0
  * Z ← tem = 0
  * V ← 0
  * `U` ← 0

Flags:  (Floating Point Operations: CMPR,CMPL)
  * C ← tem < 0
  * N ← tem < 0
  * Z ← tem = 0
  * V ← 0
  * `U` ← src or dsrc = Nan
  * IX ← 0
  * UF ← 0
  * FZ ← 0
  * OF ← 0
  * IN ← 0

Exceptions:
  * none


#### CV - CONVERT
The source operand is converted to the type and length indicated by the
destination specifier and stored at the address of the destination.

Opcodes:
```
  09     CVBR     ConVert Byte to Real
  19     CVHR     ConVert Halfword to Real
  39     CVLR     ConVert Longreal to Real
  49     CVBL     ConVert Byte to Longreal
  59     CVHL     ConVert Halfword to Longreal
  69     CVWL     ConVert Word to Longreal
  79     CVRL     ConVert Real to Longreal
  89     CVBW     ConVert Byte to Word
  99     CVHW     ConVert Halfword to Word
  A9     CVWB     ConVert Word to Byte
  B9     CVWH     ConVert Word to Halfword
```
AssemblerSyntax:
```
 CV{BW,BR,BL,HW,HR,HL,WL,WB,WH,RL,LR} src, des
```
Operation:
```
  CONVERT (src) → des
```
Flags:  (All Operations)
  * C ← C  (when des is INTEGER)
  * C ← des < 0 (when des is FLOATING POINT)
  * N ← des < 0
  * Z ← des = 0
  * V ← Integer overflow (when des is INTEGER)
  * V ← 0  (when des is FLOATING POINT)
  * `U` ← 0
  * IX ← des rounded
  * UF ← des underflowed
  * FZ ← 0
  * OF ← des overflowed
  * IN ← src = Nan

Exceptions:
  * Integer overflow [CVWB,CVWH,CVRW,CVLW]
  * Inexact [CVLR]
  * Underflow [CVLR]
  * Overflow [CVLR]
  * Invalid [CVRL,CVLR]


#### DI - DISABLE INTERRUPTS
The Interrupt Enable (IE) flag in the Program Status register is set to
zero. This disables all interrupts that can be disabled.

Opcode:
```
   9B    DI         Disable Interrupts
```
AssemblerSyntax:
```
 DI
```
Operation:
```
  0 → IE (flag in Program Status register)
```
Flags:
  * IE ← 0   no other changes

Exceptions:
  * none


#### DIV - DIVIDE
The destination is divided by the source and the result is stored at
the destination address.

Opcodes:
```
  A0     DIVB       DIVide Byte
  A2     DIVH       DIVide Halfword
  A4     DIVW       DIVide Word
  A8     DIVR       DIVide Real
  AA     DIVL       DIVide Longreal
```
AssemblerSyntax:
```
 DIV{B,H,W,R,L} src,des
```
Operation:
```
  dsrc / src → des
```
Flags:  (Integer Operations: DIVB,DIVH,DIVW)
  * C ← C
  * N ← des < 0
  * Z ← des = 0
  * V ← Integer overflow
  * `U` ← 0

Flags:  (Floating Point Operations: DIVR,DIVL)
  * C ← des < 0
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0
  * IX ← des rounded
  * UF ← des underflowed
  * FZ ← (src = 0 and des <> 0)
  * OF ← des overflowed
  * IN ← (src of dsrc = Nan) or (src and dsrc = 0)

Exceptions:
  * Integer overflow (dsrc = largest negative value, src = -1)
  * Integer Zero Divide
  * Inexact
  * Underflow
  * Floating Zero Divide
  * Overflow
  * Invalid


#### DVR - DIVIDE REVERSE
The source operand is divided by the destination operand and the result
is stored at the address of the destination.

Opcodes:
```
  B0    DVRB       DIVide Reverse Byte
  B2     DVRH       DiVide Reverse Halfword
  B4     DVRW       DiVide Reverse Word
  B8     DVRR       DiVide Reverse Real
  BA     DVRL       DiVide Reverse Longreal
```
AssemblerSyntax:
```
 DVR{B,H,W,R,L} src,des
```
Operation:
```
  src / dsrc → des
```
Flags:  (Integer Operations: DVRB,DVRH,DVRW)
  * C ← C
  * N ← des < 0Z ← des = 0
  * V ← Integer overflow
  * `U` ← 0

Flags:  (Floating Point Operations: DVRR,DVRL)
  * C ← des < 0
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0
  * IX ← des rounded
  * UF ← des underflowed
  * FZ ← (des = 0 and src <> 0)
  * OF ← des overflowed
  * IN ← (src or dsrc = Nan) or (src and dsrc = 0)

Exceptions:
  * Integer overflow (src = largest negative value, dsrc = 1)
  * Integer Zero Divide
  * Inexact
  * Underflow
  * Floating Zero Divide
  * Overflow
  * Invalid


#### EI - ENABLE INTERRUPTS
The Interrupt Enable (IE) flag in the Program Status register is set to
one. This enables all interrupts that have not been otherwise disabled.

Opcode:
```
  8B     EI         Enable Interrupts
```
AssemblerSyntax:
```
 EI
```
Operation:
```
  1 → IE (Interrupt Enable flag in Program Status register)
```
Flags:
  * IE ←1   no other changes

Exceptions:
  * none


#### ER - ERROR
Error on and off are used to set a pin level in order to indicate a
potentially fatal condition (see 4.5).

Opcodes:
```
  4B     ERON       ERror ON
  5B     EROF       ERror OFf
```
AssemblerSyntax:
```
 ER{ON,OF}
```
Operation:
```
  ERROR pin ← 1 (ERON)
  ERROR pin ← p (EROF)
```
Flags:
  * no changes

Exceptions:
  * none


#### FFO - FIND FIRST ONE
If the source is zero the destination is set to 8 (FFOB), 16 (FFOH) or
32 (FFOW) and the Z Flag is set to one. Otherwise, Z is zero and the
destination is set to the bit position of the first one bit in the
source, scanning from the right (e.g. if the least significant bit is
one the destination is set to zero). The destination is a Byte even
though the source can be a Byte (FFOB), Halfword (FFOH) or Word (FFOW).

Opcodes:
```
  31     FFOB       Find First One Byte
  33     FFOH       Find First One Halfword
  35     FFOW       Find First One Word
```
AssemblerSyntax:
```
 FFO{B,H,W} src,des
```
Operation:
```
  location of first one (src) → des
```
Flags:
  * C ← C
  * N ← 0
  * Z ← src = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


#### LCNT - LOAD COUNT
The I/O Count Register designated by the destination is loaded with the
source operand, The Input Registers are numbered 0,1,. . .,9,31 and
the Output Registers are 32,33,. . .,41,63. The least significant bit
of the Count Register is always zero but no error is signaled if an
attempt is made to load an odd number. Also no error is signaled if
des is greater than 63 but the result is undefined. The source operand
is a Word and the destination is a Byte.

Opcode:
```
  A5     LCNT       Load CouNT
```
AssemblerSyntax:
```
 LCNT src,des
```
Operation:
```
  src → I/O Count Register #(des)
```
Flags:
  * no changes

Exceptions:
  * none


#### LDPR - LOAD PROCESSOR REGISTERS
The source value is loaded into the Processor Register designated
by the destination. The Processor Registers are listed below. No
operation is performed if a "read only" register is designated
by des. The source is a Word and the destination operand is a Byte
value indicating one of the Processor Registers.
```
  P0    SP     Stack Pointer
  P1    PS     Program Status
  P2    FR     Fault Register
  P3    CR     Configuration Register
  P4    PI     Processor I. D.
  P5    OR     Output Ready  (read only)
  P6    IR     Input Ready(read only)
  P7    OE     Output Enable
  P8    IE     Input Enable
  P9    IP     Input Pending(read only)
  P10   PE     Parity Error(read only)
  P11   IO     Input Overrun (read only)
```

Opcode:
```
  85     LDPR       LoaD Processor Register
```
AssemblerSyntax:
```
 LDPR src,des
```
Operation:
```
  src → Processor Register #(des)
```
Flags:
  * no changes

Exceptions:
  * none


#### LPTR - LOAD POINTER
The I/O Address Register designated by the destination is loaded with
the source operand. The Input Registers are numbered 0,1,. . .,9,31 and
the Output Registers are 32,33,. . .,41,63. The least significant bit
of the Address Register is always zero but no error is signaled if an
attempt is made to load an odd address. Both operands are Words.

Opcode:
```
  B5     LPTR       Load PoinTeR
```
AssemblerSyntax:
```
 LPTR src,des
```
Operation:
```
  src → I/O Address Register #(des)
```
Flags:
  * no changes

Exceptions:
  * none


#### MOV - MOVE
The source value is moved to the destination address.

Opcodes:
```
  00     MOVB       MOVe Byte
  02     MOVH       MOVe Halfword
  04     MOVW       MOVe Word
  08     MOVR       MOVe Real
  0A     MOVL       MOVe Longreal
```
AssemblerSyntax:
```
 MOV{B,H,W,R,L} src,des
```
Operation:
```
  src → des
```
Flags:
  * no changes

Exceptions:
  * none


#### MOVA - MOVE ADDRESS
The address specifier of the source operand is evaluated and stored at
the destination location. If the addressing mode of the source is
Literal, Immediate or Register Direct the PC is first added to the
source value. The value of PC used is that at the beginning of the
instruction. If the source addressing mode is Stack mode then the
contents of the Stack Pointer are moved to the destination.

Opcode:
```
  E5     MOVA       MOVe Address
```
AssemblerSyntax:
```
 MOVA src,des
```
Operation:
```
  Literal or Immediate Mode: src + PC → des
  Register Direct Mode: content (reg) + PC → des
  Stack Mode: content (SP) → des
  Other Modes: address of (src) → des
```
Flags:
  * no changes

Exceptions:
  * Illegal Address


#### MUL - MULTIPLY
The source and destination are multiplied and the result is stored
at the address of the destination. Integer overflow occurs when the
high order half of the product is not the sign extension of the low
order half. This is true even when the operands are bytes or halfwords
in registers.

Opcodes:
```
  80     MULB       MULtiply Byte
  82     MULH       MULtiply Halfword
  84     MULW       MULtiply Word
  88     MULR       MULtiply Real
  8A     MULL       MULtiply Longreal
```
AssemblerSyntax:
```
 MUL{B,H,W,R,L} src,des
```
Operation:
```
  src * dsrc → des
```
Flags:  (Integer Operations: MULB,MULH,MULW)
  * C ← C
  * N ← des < 0
  * Z ← des = 0
  * V ← Integer overflow
  * `U` ← 0

Flags:  (Floating Point Operatios: MULR,MULL)
  * C ← des < 0
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0
  * IX ← des rounded
  * UF ← des underflowed
  * FZ ← 0
  * OF ← des overflowed
  * IN ← dsrc or src = Nan

Exceptions:
  * Integer overflow
  * Inexact
  * Underflow
  * Overflow
  * Invalid


#### NEG - NEGATE
The source operand is negated and the result is stored at the address
of the destination. Integer overflow occurs when the source is the
largest negative number.

Opcodes:
```
  10     NEGB       NEGate Byte
  12     NEGH       NEGate Halfword
  14     NEGW       NEGate Word
  18     NEGR       NEGate Real
  1A     NEGL       NEGate Longreal
```
AssemblerSyntax:
```
 NEG{B,H,W,R,L} src,des
```
Operation:
```
  -(src) → des
```
Flags:  (Integer Operations: NEGB,NEGH,NEGW)
  * C ← borrow from most significant bit
  * N ← des ← 0    TODO - verify this
  * Z ← des = 0
  * V ← Integer overflow
  * `U` ← 0

Flags:  (Floating Point Operations: NEGR,NEGL)
  * C ← des < 0
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0
  * IX ← 0
  * UF ← 0
  * FZ ← 0
  * OF ← 0
  * IN ← src = Nan

Exceptions:
  * Integer overflow
  * Invalid


#### NOP - NO OPERATION
This instruction does nothing.

Opcode:
```
  0B     NOP        NO oPeration
```
AssemblerSyntax:
```
 NOP
```
Operation:
```
  nothing
```
Flags:
  * no changes

Exceptions:
  * none


#### NOT - NOT
The source is complemented and the result is stored at the destination
location.

Opcodes:
```
  71     NOTB       NOT Byte
  73     NOTH       NOT Halfword
  75     NOTW       NOT Word
```
AssemblerSyntax:
```
 NOT{B,H,W} src,des
```
Operation:
```
  NOT(src) → des
```
Flags:
  * C ← C
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


#### OR - OR
The destination and source are "ored" together and the result is stored
at the address of the destination.

Opcodes:
```
  51     ORB        OR Byte
  53     ORH        OR Halfword
  55     ORW        OR Word
```
AssemblerSyntax:
```
 OR{B,H,W} src,des
```
Operation:
```
  src OR dsrc → des
```
Flags:
  * C ← C
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


#### REM - REMAINDER
The remainder of the destination divided by the source replaces the
destination. The following point instruction is used for argument
reduction and is always exact. However, it is only a partial remainder;
the instruction must be repeated until Z becomes one (that is the
reason for the unusual definition of the Z flag).

Opcodes:
```
  90     REMB       REMainder Byte
  92     REMH       REMainder Halfword
  94     REMW       REMainder Word
  98     REMR       REMainder Real
  9A     REML       REMainder Longreal
```
AssemblerSyntax:
```
 REM{B,H,W,R,L} src,des
```
Operation:
```
  dsrc REM src → des
```
Flags:  (integer Operations: REMB,REMH,REMW)
  * C ← C
  * N ← des < 0
  * Z ←  des = 0
  * V ← 0
  * `U` ← 0

Flags:  (Floating Point Operations: REMR,REML)
  * C ← des < 0
  * N ← des < 0
  * Z ← abs(des) < abs(src)
  * V ← 0
  * `U` ← 0
  * IX ← 0
  * UF ← des underflowed
  * FZ ← 0
  * OF ← 0
  * IN ← (dsrc or src = Nan) or (src = 0)

Exceptions:
  * Integer Zero Divide
  * Underflow
  * Invalid


#### REP - REPEAT
A REPeat instruction may precede and other instruction. It causes bits
26 to 31 in the Program Status register to be set as shown above. The
instruction following the repeat is reexecuted and the indicated count
register (src must be a general register designator) is decremented
until the repeat condition is satisfied. One of the conditions for
all three instructions is that the count register becomes zero. But if
the Z flag becomes zero (REPZ) or one (REPNZ) then the condition is
also satisfied and the repeat is terminated by setting bits 30 and 31
in the PS register to 0. The Z flag is checked (for REPZ and RPNZ)
before the Count register is decremented so that it will correctly count
the number of times the following instruction is executed. If the
Count is initially zero the following instruction is skipped. If
a repeat is used with a branch instruction it has the effect of a
"loop" instruction. If an addressing mode other than register direct
is used, an address error is signaled. Also, if the designated Count
register is used in the following instruction in an addressing
mode or as an operand the results are undefined.

As examples of the use of Repeat assume that R4 and R5 point to two
vectors of real numbers, that R15 contains the length of the vectors
and that R10 is zero. The

```
  REP R15
  ADDR (R4)+,R10
```

will accumulate in R10 the summation of the vector elements pointed to
by R4 and

```
  L:  MOVR (R4)+,R9
      MULR (R5)+,R9
      ADDR R9,R10
      REP  R15
      JMP  L
```

will compute the inner product of the two vectors.

Opcodes:
```
  1E    REP       REPeat while Count not Zero
  2E    REPZ      REPeat while Zero flag set
  3E    REPNZ     REPeat while zero flag Not set
```
AssmeblerSyntax:
```
REP{,Z,NZ} src
```
Operation:
```
  REP: PS(30,31) ← 01; Count = REG#(src)
  REPZ: PS(30,31) ← 10; Count = REG#(src); Z = 1
  REPNZ: PS(30,31) ← 11; Count = REG#(src); Z = 0
    for all: PS(26,27,28,29) ← Count
      after repeat condition satisfied (on REPZ and RPNZ
      the Z flag is checked before the Count)
      PS(30,31) ← 00
```

Flags:
  * no changes

Exceptions:
  * address


#### RET - RETURN
The contents of the stack top (assumed to be a return address) are popped
into the Program Counter.

Opcode:
```
  EB     RET        RETurn
```
AssemblerSyntax:
```
 RET
```
Operation:
```
  PC ← stack
```
Flags:
  * no changes (the Repeat Mode is reset)

Exceptions:
  * none


#### RETI - RETURN FROM INTERRUPT
The top of stack (assumed to contain the PC in effect before the current
interrupt) is popped into the PC register and then the next value on
the stack is popped into the Program Status (PS) register.

Opcode:
```
  CB     RETI       RETurn from Interrupt
```
AssemblerSyntax:
```
 RETI
```
Operation:
```
  PC ← stack
  PS ← stack
```
Flags:
  * All flags set according to the new PS

Exceptions:
  * none


#### RETP - RETURN AND POP
The top of stack is popped into the Program Counter and then the source
(Word) value is added to the Stack Pointer in order to pop a set of
local variables off the stack.

Opcode:
```
  EF     RETP       RETurn and Pop
```
AssemblerSyntax:
```
 RETP src
```
Operation:
```
  PC ← stack
  SP ← SP + src
```
Flags:
  * no changes (the Repeat Mode is reset)

Exceptions:
  * none


#### ROT - ROTATE
If the source is zero the destination is not changed but the Carry
flag is set to the least significant bit of dsrc. Otherwise dsrc is
rotated (left if src < 0; right of src > 0) and the Carry flag is set
to the value of the last bit shifted out. The source is always a Byte
operand even though the destination can be a Byte (ROTB), Halfword
(ROTH) or Word (ROTW).

Opcodes:
```
  21     ROTB       ROTate Byte
  23     ROTH       ROTate Halfword
  25     ROTW       ROTate Word
```
AssemblerSyntax:
```
 ROT{B,H,W} src,des
```
Operation:
```
  dsrc ROTATE BY src → des
```
Flags:
  * C ← if src = 0 then the least significant bit of des,
    otherwise the last bit shifted out
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


#### RSET - RESET
RSET causes the Integer and Floating point Execution units to be
initialized and all pending interrupts to be reset. All I/O activity
is aborted. The serial channel "ready" flags are set to one (ready)
and all other I/O registers are cleared including error flags.

Opcode:
```
  7B     RSET       ReSET processor
```
AssemblerSyntax:
```
 RSET
```
Operation:
```
  The processor is initialized
```
Flags:
  * no changes

Exceptions:
  * none


#### SBB - SUBTRACT WITH BORROW
The Carry (borrow) and source values are subtracted from the destination
and the result replaces the destination.

Opcodes:
```
  70     SBBB     SuBtract with Borrow Byte
  72     SBBH     SuBtract with Borrow Halfword
  74     SBBW     SuBtract with Borrow Word
```
AssemblerSyntax:
```
 SBB{B,H,W} src,des
```
Operation:
```
  dsrc - src - Carry → des
```
Flags:
  * C ← borrow from most significant bit
  * N ← des < 0
  * Z ← des = 0
  * V ← Integer overflow
  * `U` ← 0

Exceptions:
  * Integer overflow


#### SBBD - SUBTRACT DECIMAL
The Carry value (borrow) and source (Byte) value treated as a two
BCD digit value are subtracted from the destination considered
similarly. The result replaces the destination. The operands are
not checked for invalid BCD format.

Opcode:
```
  91     SBBD     SuBtract with Borrow Decimal
```
AssemblerSyntax:
```
 SBBD src,des
```
Operation:
```
  dsrc - src - Carry → des
```
Flags:
  * C ← borrow from most significant digit
  * N ← 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


#### SBR - SUBTRACT REVERSE
The destination value is subtracted from the source and the result
replaces the destination.

Opcodes:
```
  20     SBRB       SuBtract Reverse Byte
  22     SBRH       SuBtract Reverse Halfword
  24     SBRW       SuBtract Reverse Word
  28     SBRR       SuBtract Reverse Real
  2A     SBRL       SuBtract Reverse Longreal
```
AssemblerSyntax:
```
 SBR{B,H,W,R,L} src,des
```
Operation:
```
  src - dsrc → des
```
Flags:  (Integer Operations: SBRB,SBRH,SBRW)
  * C ← borrow from most significant bit
  * N ← des < 0
  * Z ← des = 0
  * V ← Integer overflow
  * `U` ← 0

Flags:  (Floating Point Operations: SBRR,SBRL)
  * C ← des < 0
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0
  * IX ←  des rounded
  * UF ← des underflowed
  * FZ ← 0
  * OF ← des overflowed
  * IN ← src or dsrc = Nan

Exceptions:
  * Integer overflow
  * Inexact
  * Underflow
  * Overflow
  * Invalid


#### SFA - SHIFT ARITHMETIC
If the source is zero the destination is unchanged and the Carry flag
is set to the least significant bit of the destination. Otherwise,
the operand at the destination address is shifted by the number of
places equal to the value of the source. If the source is positive the
shift is to the left and if negative it is to the right. Left shifts
cause zero to be shifted in from the right and right shifts cause the
sign to be copied from the left. In both cases the Carry flag is set
to the last bit shifted out. If the shift is right Integer overflow
cannot occur but left shifts cause Integer overflow if the bits
shifted out are not all equal to the resulting sign bit. The source
operand is always a Byte operand even though the destination can be
a Byte (SFAB), Halfword (SFAH) or Word (SFAW).

Opcodes:
```
  11     SFAB       ShiFt Arithmetic Byte
  13     SFAH       ShiFt Arithmetic Halfword
  15     SFAW       ShiFt Arithmetic Word
```
AssemblerSyntax:
```
 SFA{B,H,W} src,des
```
Operation:
```
  dsrc SHIFT ARITHMETIC BY src ← des
```
Flags:
  * C ← if src = 0 then least significant bit (dsrc)
    otherwise last bit shifted out
  * N ← src < 0
  * Z ← des = 0
  * V ← Integer overflow
  * `U` ← 0

Exceptions:
  * Integer overflow


#### SFT - SHIFT LOGICAL
If the source is zero the destination is unchanged and the Carry flag
is set to the least significant bit of the destination. Otherwise, the
operand at the destination address is shifted by the number of places
equal to the value of the source. If the source is positive the shift
is to the left and if negative it is to the right. Left shifts cause
zero to be shifted in from the right and right shifts cause zero to be
shifted in from the left. In both cases the Carry flag is set to the
last bit shifted out. The source operand is always a Byte operand even
though the destination can be a Byte (SFTB), Halfword (SFTH) or Word
(SFTW).

Opcodes:
```
  01     SFTB       ShiFT logical Byte
  03     SFTH       ShiFT logical Halfword
  05     SFTW       ShiFT logical Word
```
AssemblerSyntax:
```
 SFT{B,H,W} src,des
```
Operation:
```
  dsrc SHIFT LOGICAL BY src → des
```
Flags:
  * C ← if src = 0 then least significant bit (dsrc)
    otherwise last bit shifted out
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


#### SGN - SET SIGN
The sign of the destination is set to the sign of the source.

Opcodes:
```
  78     SGNR       Set siGN Real
  7A     SGNL       Set siGN Longreal
```
AssemblerSyntax:
```
 SGN{R,L} src,des
```
Operation:
```
  SIGN (src) → SIGN (des)
```
Flags:
  * C ← des < 0
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0
  * IX ← 0
  * UF ←  0
  * FZ ← 0
  * OF ← 0
  * IN ← src or dsrc = Nan

Exceptions:
  * Invalid


#### SQT - SQUARE ROOT
The square root of the source replaces the destination. The square root
is correctly rounded and connot overflow or underflow.

Opcodes:
```
  58     SQRT       SQuare rooT Real
  5A     SQTL       SQuare rooT Longreal
```
AssemblerSyntax:
```
 SQT{R,L} src,des
```
Operation:
```
  SQUARE ROOT (src) → des
```
Flags:
  * C ← 0
  * N ← 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0
  * IX ← des rounded
  * UF ← 0
  * FZ ← 0
  * OF ← 0
  * IN ← (src < 0) or (src = Nan)

Exceptions:
  * Inexact
  * Invalid


#### STC - SET CARRY
The Carry flag is set to one.

Opcode:
```
  2B     STC        SeT Carry
```
AssemblerSyntax:
```
 STC
```
Operation:
```
  1 → Carry
```
Flags:
  * C ← 1   no other changes

Exceptions:
  * none


#### STPR - STORE PROCESSOR REGISTERS
The contents of the Processor Register whose number corresponds with
the value of the source replaces the destination. The destination
is a Word and the source is a Byte value designating a Processor
Register. The Processor Registers are listed below.
```
  P0 SP Stack Pointer
  P1 PS Program Status
  P2 FR Fault Register
  P3 CR Configuration Register
  P4 PI Processor I. D.
  P5 OR Output Ready (read only)
  P6 IR Input Ready(read only)
  P7 OE Output Enable
  P8 IE Input Enable
  P9 IP Input Pending(read only)
  P10 PE Parity Error(read only)
  P11 IOInput Overrun (read only)
```

Opcode:
```
  95     STPR       STore Processor Registers
```
AssemblerSyntax:
```
 STPR src,des
```
Operation:
```
  PROCESSOR REGISTER # (src) → des
```
Flags:
  * C ← C
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


#### SUB - SUBTRACT
The source is subtracted from the destination and the result is stored
at the address of the destination.

Opcodes:
```
  60     SUBB       SUBtract Byte
  62     SUBH       SUBtract Halfword
  64     SUBW       SUBtract Word
  68     SUBR       SUBtract Real
  6A     SUBL       SUBtract Longreal
```
AssemblerSyntax:
```
 SUB{B,H,W,R,L} src,des
```
Operation:
```
  dsrc - src → des
```
Flags:  (Integer Operations: SUBB,SUBH,SUBW)
  * C ← borrow from most significant bit
  * N ← des < 0
  * Z ← des = 0
  * V ← Integer overflow
  * `U` ← 0

Flags:  (Floating Point Operations: SUBR,SUBL)
  * C ← des < 0
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0
  * IX ← des rounded
  * UF ← des underflowed
  * FZ ← 0
  * OF ← des overflowed
  * IN ← src or dsrc = Nan

Exceptions:
  * Integer overflow
  * Inexact
  * Underflow
  * Overflow
  * Invalid


#### TRAP - TRAP
The current values of PS and PC are pushed on the stack and the value
at location (8 * src) replaces the PC while the value at location (8 *
src + 4) replaces the PS. The source operand is an unsigned Byte.

Opcode:
```
  1E     TRAP       TRAP
```
AssemblerSyntax:
```
 TRAP src
```
Operation:
```
  generate interrupt # (src):
    stack ← PS
    stack ← PC
    PC ← Word at location (8 * src)
    PS ← Word at location (8 * src + 4)
```
Flags:
  * all flags set according to the new PS value

Exceptions:
  * none


#### WAIT - WAIT
This instruction causes the processor to idle until it receives an
interrupt.

Opcode:
```
  DB     WAIT       WAIT
```
AssemblerSyntax:
```
 WAIT
```
Operation:
```
  wait for interrupt
```
Flags:
  * no changes

Exceptions:
  * none


#### XOR - EXCLUSIVE OR
The destination is set to the exclusive or of the source and the operand
at the destination location.

Opcodes:
```
  61     XOBR       eXclusive OR Byte
  63     XORH       eXclusive OR Halfword
  65     XORW       eXclusive OR Word
```
AssemblerSyntax:
```
 XOR{B,H,W} src,des
```
Operation:
```
  src XOR dsrc → des
```
Flags:
  * C ← C
  * N ← des < 0
  * Z ← des = 0
  * V ← 0
  * `U` ← 0

Exceptions:
  * none


