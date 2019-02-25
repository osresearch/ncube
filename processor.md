# 4. THE PROCESSOR

## 4.1 Introduction

The processor array is made up of 2N nodes where N is 6,7,8,9 or 10.
Each processing node (FIG. 4) consists of a general purpose 32
bit processor (including 32 and 64 bit floating point instructions),
128K bytes of ECC memory and 11 communication channels to support the
hypercube interconnection scheme and the 8 system I/O channels.

## 4.2 Architecture Overview

### 4.2.1 Data Representation

The processor recognizes two main classes of data: integers and
reals. Integers are represented in standard 2's complement form and
come in three types: byte (B-8 bits), halfword (H-16 bits) and word
(W-32 bits). There two types of reals. The 32 bit format, called real
(R), has an 8 bit exponent and 24 bits of significance. The longreal (L)
format is 64 bits with 11 in the exponent and 53 in the significand. The
longreal format is used for computations that need high accuracy and for
intermediate computations with real variables when the computation is
particular sensitive to roundoff error. Both of these formats conform
to the IEEE Binary Floating Point Standard (P754).

In addition to the various data formats, the processor recognizes and
manipulates addresses. Addresses are simply 32 bit unsigned values that
point to individual bytes in a linear address space.

### 4.2.2 Registers, Interrupts and Communication

The processor's instructions operate on data in main memory (as described
above) or on data in 32 bit registers. The processor contains three
types of registers: the general registers, the processor registers and
the communication control registers. The 16 general registers are 32 bits
long and are used for both operands and addresses. Since they are general
they can be used interchangeably in all operations and addressing modes.

The processor registers are special purpose and can only be read from
or written onto by Load Processor register (LDPR) and Store processor
Register (STPR) instructions respectively. The exact formats and detailed
descriptions of these registers are given in section 4.4.3. The processor
registers are shown in FIGS. 7, 9A and 9B and include:

* 0 Stack Pointer (SP)--points to the top of the stack

* 1 Program Status (PS)--contains flags, interrupt controls and other
status information

* 2 Fault Register (FR)--the fault codes are stored here

* 3 Configuration Register (CR)--the model number (read only) and memory
interface parameters are stored here

* 4 Processor Identification(PI)--contains a number that identifies the
processor's location in the array

* 5 Time Out (TO--contains a counter that is decremented approximately
every 100 microseconds and generates an interrupt (if enabled) when it
reaches zero

Processor registers 6 through 12 are used to signal "ready" and error
conditions for the I/O channels.

The I/O ports on the processor are unidirectional Direct Memory Access
(DMA) channels and each channel has two 32 bit write only registers: an
address register for the buffer location and a count register indicating
the number of bytes left to send or receive. Communication is performed by
setting the registers of the desired channel to the appropriate address
and data length and then the DMA channel takes over and communicates a
message without processor intervention. Interrupts can be used to signal
when a channel is available (i.e. when the count reaches zero the channel
is "ready"). A separate interrupt vector is provided to indicate to a
receiver that an error occurred during the data transmission.

In addition to communication synchronization and error reporting the
processor uses vectored interrupts for:

1. hardware errors (e.g. multibit memory errors)
2. program exceptions (e.g. real overflow)
3. software facilities (e.g. trace, timeout)

When an interrupt occurs the current program status (PS) and program
counter (PC) are pushed on the stack. Then PS and PC are loaded with
new values from the appropriated entry (indexed by the interrupt number)
in the interrupt vector table in low memory.

### 4.2.3 Instruction Formats and Addressing Modes

An instruction consists of an operation code followed by zero and one
or two data references:

```
Opcode | Reference 1 | Reference 2
Low address                High address
```


All instruction operation codes (opcodes) in the processor are one
byte long. The first four bits indicate the operation and number of
operands (e.g. `ADD`: 2 operands, `BRANCH`: 1 operand) while the other four
bits denote the operand size and type (e.g. Halfword (integer), Real
(floating point). This symmetry makes an opcode map easy to read and
code generation easier for a compiler.

All of the standard instructions are available for each data type
including arithmetic, logical, conversion, comparison, branch, call and
trap instructions. Instructions can be preceded by a `REPEAT` prefix that
causes them to be executed repeatedly until a termination condition
is satisfied. This is a very powerful facility for vector and string
operations. Repeats can also be used with both branches and calls in
order to execute a block of code repeatedly. (i.e. a `REPEAT BRANCH` is
equivalent to a loop instruction). And for future extension each operand
type has a reserved "excap" code.

A few instructions have no operands (e.g. `BREAKPOINT`) and some have
only one (e.g. `CALL`) but most have two address fields. All address
fields begin with a one byte mode selector. For all modes involving the
general registers the first four bits indicate the mode and the remaining
four determine which register to use. If there is an offset indicated
it follows the mode selector. Some of the modes provided are literal,
immediate, direct and indirect with no registers involved; and register
direct, register indirect with and without offset, autoincrement and
autodecrement and offset addressing with both the program counter (PC)
and the stack pointer (SP). As with instructions there is a reserved
"escape" code defined for the mode selector field.

## 4.3 Data Representation

The processor recognizes two classes of data: integrers and reals
(floating point number). There are three types of integers and two types
of reals.

### 4.3.1 Integers

The three integer data types are all represented in standard 2's
complement. They are called Byte (B), Halfword (H) and Word (W) and are
8, 16 and 32 bits long respectively. The ranges for the three integer
formates are specified as follows:

* Byte (B): -128 to 127
* Halfward (H): -32,768 to 32,767
* Word (W): -2,147,483,648 to 2,147,483,647

Most instructions treat integers as signed numbers but the
logical operations (e.g. AND, OR) view their operands as unsigned
quantities. Addresses are also treated by the processor as unsigned
values. The address space is logically a linear set of bytes from address
0 to 2^32-1; thus addresses are unsigned 32 bit integers (Words).

### 4.3.2 Reals

The floating point implementation in the processor conforms to the IEEE
Binary Floating Point Standard (P754). With the floating point arithmetic
not only are the rounded results as accurate as possible but it is
feasible to compute guaranteed bounds on the errors using the special
directed rounding modes. Also because to the high accuracy of Real (32
bits) computations and the availability of Longreal (64 bits) to back
them up at crucial points, it will be possible to run many more programs
in Real precision instead of automatically using Longreal everywhere.

The representations for the two floating point type are illustrated below
including the formulas for the value represented. In the formulas "s"
is the sign, "e" is the exponent, "f" is the fraction and "b" is the
bias in the exponent.

```
Real  | s | e | f |   b = 127
     31      23   0
```

```
Longreal | s | e | f |   b = 1024 
        63      51   0
```

The two formats are closely related; the distinguishing characteristics
being the exponent range (defined by the parameter b) and the fraction
precision. The Real format has 24 bits of precision (about 7 digits)
with a range of approximately 10^(-38) to 10^(38). The Longreal
format has a much wider range--about 10^(-308) to 10^(308) -- and more
than twice the precision of Real at 53 bits or about 15 digits. Thus
Longreal, besides being a powerful standalone computational format,
makes an excellent backup facility for Real calculations at points in
a program where the results are very sensitive to roundoff error.

This implementation conforms to the IEEE Floating Point Standard which
was carefully designed to provide accurate and reliable arithmetic. The
following properties are a result of the standard.

1. Denormalized numbers (e=0) fill the space between zero and the
smallest normalized number. They provide a far superior way of dealing
with underflow than the typical "flush to zero" response.

2. The implicit bit yields the greatest possible accuracy and is one of
the two reasons for choosing radix 2. The other is speed; for a given
amount of hardware binary will always be fastest.

3. The offset (b) was chosen to ensure that all normalized numbers have
representable reciprocals.

4. The format was organized to permit very fast comparisons.

5. Infinities (e=11...1, f=0) were explicity represented to allow
for handling zero divide and overflow exceptions.

6. When e=11...1 and f<>0 the representation is treated as Not a
Number (Nan) and instead of producing a numeric result when used as
an operand the processor generates an exception. Nan's were provided
to allow for software extensions including runtime diagnostics like
"uninitialized variable" and to permit potentially flawed computation
like 0/0 to continue in order to observe the effect, if any, on the
final results.

7. Longreal has greater range and more than double the precision of Real
to permit exact Real multiply with no threat of overflow or underflow
and generally to allow for Longreal accumulations of Real computations.

The floating point architecture of the processor implemented in accordance
with the principles of the present invention includes much more than the
data representations. All of the IEEE Standard requirements are either met
in the hardware or are facilitated in software. Among these requirements
is the provision of rounding modes. In the Program Status (PS) register
are two bits that control the rounding mode in effect. The modes are:

* `00` Round to Nearest Even: in this mode the closest possible result is
returned. If there are two then the even one is generated. This removes
the bias that exists in the more typical "round up in the half case"
rounding.

* `01` Round Up: the larger of the two numbers that bracket the exact result
is returned.

* `10` Round Down: this returns the smaller of the two possibilities.

* `11` Round Toward Zero: the result is the one that is the smaller in
magnitude.

Another important facility in the floating point architecture is exception handling. The following required faults are recognized

* Inexact Result: when the result of an operation is not exact but must
be rounded.

* Underflow: the result is nonzero and less in magnitude than the
smallest normalized number.

* Zero Divide: the denominator is zero while the numerator is nonzero.

* Overflow: the rounded result is larger in magnitude than the largest
representable number.

* Invalid Operation: this includes indeterminate operations like 0/0,
0 * INFINITY, etc. and the use of a Nan as an operand.

All of these exceptions have an associated flag (and Inexact has an
interrupt enable) in the PS register. If an exception occurs and its
interrupt is enabled, the processor produces enough information for
recovery. If the interrupt is disabled the flag is set and the processor
takes predefined action:

* Inexact Result: store the rounded result and continue (In this
implementation only Inexact Result may be disabled.) The exceptions and
responses are defined in detail in Section 4.5.

The floating point architecture also provides all the standard
instructions for all formats: add, subtract, multiply, divide,
compare and conversion. But in addition there are some unusual but
crucial instructions. Square root is correctly rounded and as fast as
divided. Remainder is an exact operation and permits argument reduction
for periodic functions with no roundoff error.

## 4.4 Registers

The following sections describe three types of registers in the processor:
the General registers, the Input/Output registers and the Processor
registers.

### 4.4.1 General Registers

The 16 General registers (128), shown in FIG. 9A, are labeled R0 to
R15. They are 32 bits wide and are used for data and addresses. They are
consistently symmetrical with no special designations or uses for any
of them. When integer data shorter than 32 bits is moved to a General
register it is sign-extended to 32 bits. When data longer than 32 bits
are stored in the registers, the low order part of the data goes in the
designated register, Ri, and the high order part resides in Ri+1. The
numbers "wrap around" so that if a Longreal is moved to R15 the high
order section is found in R0.

### 4.4.2 Input/Output Registers

In a processor, each of the 11 input and output ports (48), shown in
FIG. 5, is an independent Direct Memory Access (DMA) channel and has two
32 bit registers: an address register and a count register. The address
register contains a pointer to the least significant byte of the next
halfword to be transferred. If it is an output port the data is moved
from memory out to the port. If it is an input port the data is moved
to memory that has been received from the output port of the sending
processor. In both cases the count register is set to indicate the
number of bytes to be sent or received. As data is sent or received, the
appropriate address and count registers are incremented and decremented
respectively by the number of bytes transferred. When the count reaches
zero the ready flag in the Input or Output Status register (see below)
is set and an interrupt is generated if an interrupt has been enabled.

The DMA channels operate independently of instruction processing. They
being functioning whenever a count register is set to a nonzero
value. All of the ports are general except one input and one output port
are designated "host" (H) and are normally used to communicate over the
I/O bus to the System Control Boards.

### 4.4.3 Processor Registers

The Processor registers are the third type of register in the
processor. All Processor registers are 32 bits wide. They contain all the
special purpose and miscellaneous information and can only be loaded or
stored by the Load Processor Register (`LDPR`) and Store Processor Register
(`STPR`) instructions, respectively. These registers are labeled P0 to
P11 but they also have unique names that denote their purpose:

#### Stack Pointer (P0, SP)
The `SP` contains a pointer to the current top of stack. The stack grows
toward low memory.

#### Program Status (P1, PS)
This register contains the information that defines the current state
of a program. The format of the `PS` is shown below:

```
 | REP | REP REG | RC | R R T IE IG H TO CE |
31    29        26   24                     0

 | IV R R R R HX IN OF FZ UF IX U N Z V C |
15                                        0
```

All of the fields are one bit except REP (2 bits), REP REG (4 bits),
and RDC (2 bits). The meanings of the fields are defined below (R is
"Reserved"):

FLAGS

* `C` -- Carry is set on integer operations when there is a carry out
of the most significant position. It is also set by floating point
instructions and integer multiply and divide to indicate that the result
is negative. This allows the use of the Unsigned Branches to implement
the "unordered" branches required by the IEEE Floating Point Standard.

* `V` -- Integer Overflow is set when the integer result is too large
in magnitude for the format.

* `Z` -- The Zero flag is set when the integer or floating point result
is zero.

* `N` -- Negative is set when the integer or floating point result is
negative. If there is an Integer Overflow the Negative flag will not
agree with the sign bit of the stored result because the Negative flag
is set according to the actual result before Overflow is determined.

* `U` -- The Not Comparable flag is set when floating point values are
compared and one or both of the operands is Not-a-number (Nan).

FLOATING POINT EXCEPTIONS

The indicated flag is set when the associated exception occurs and if not
disabled the corresponding interrupt is generated. (In present embodiment
of the invention only the Inexact Result interrupt can be disabled. The
exceptions are defined in Section 4.5.

* `IX` -- Inexact Result
* `UF` -- Underflow
* `FZ` -- Floating Zero Divide
* `OF` -- Overflow
* `IN` -- Invalid Operation

INTERRUPT ENABLE FLAGS

If a flag is set and the associated exception or event occurs an
interrupt is generated. If the bit is zero the interrupt is suppressed
until the interrupt condition is cleared or the interrupt is enabled. The
floating point interrupt conditions are cleared as soon as the subsequent
instruction begins execution.

* `IIX` -- Inexact Result Enable
* `R` -- Reserved for Underflow Enable
* `R` -- Reserved for Zero Divide Enable
* `R` -- Reserved for Invalid Operation Enable
* `IV` -- Integer Overflow Enable
* `CE` -- Correctable ECC (when a memory error is corrected by the processor's
ECC logic and this flag is set an interrupt is generated; this permits
logging the number of memory errors.)
* `TO` -- Timeout Enable (if this flag is zero the interrupt that would be
generated by a zero value in the Timeout Register is suppressed.)
* `II` -- Input Enable (if this flag is zero any interrupt associated with
an input channel is suppressed.)
* `IO` -- Output Enable (if this flag is zero all output channel interrupts
are suppressed.)
* `IE` -- Interrupt Enable (if this flag is zero then all interrupts that
can be disabled by other flags are disabled)
* `T` -- Trace (this flag, if set to one, causes an interrupt as soon as the
current instruction finishes; this is used for "single step" debugging.)

CONTROL FIELDS

* `RC` -- Round Control (this field controls the rounding mode for real
operations.)
  * `00` Round to Nearest Even
  * `01` Round Up
  * `10` Round Down
  * `11` Round Toward Zero

* `REP` -- Repeat Mode (this field indicates the repeat mode in effect
for the instruction following one of the REPEAT operation codes.)
  * `00` No Repeat
  * `01` Repeat white REG is not zero
  * `10` Repeat while REG is not zero and the Z flap is one.
  * `11` Repeat while REG is not zero and the Z flag is zero.

* `REP REG` -- Repeat Register (if the repeat mode is not 00 then every
time the instruction the following repeat-type operation code is executed
the value in REG is decremented; REG can be any of the General registers.)

#### Fault Register (P2, FR)
When the processor takes an interrupt generated by an exception this
register contains information to aid recovery. The format of the Fault
Register is shown below.

```
 | R R R R R R R R R R R R |
31                        20

 | R R R R R S2 I2 E2 F2 S1 I1 E1 F1 GR RS RL |
15                                            0
```

* RU -- Round up (1 means round up)
* RS -- Round OR stick bit
* GR -- Guard bit
* F1 -- Fraction (F1 = 1 means fraction = 0)   \
* E1 -- Exponent (E1 = 1 means exponent = 0)    } First Operand
* I1 -- Invalid (I1 = 1 means exponent = ?)     |
* S1 -- Sign (S1 = sign of first operand)      /
* F2 -- Fraction (F2 = 1 means fraction = 0)   \
* E2 -- Exponent (E2 = 1 means exponent = 0)    } Second Operand
* I2 -- Invalid (I2 = 1 means exponent = ?)     |
* S2 -- Sign (S2 = sign of second operand)     /

The Guard, Round and Sticky bits are the hardware bits that are used
for rounding in floating point operations as defined in the IEEE Binary
Floating Point Standard. The Fraction, Exponent, Invalid and Sign bits
for each operand allow an interrupt handler to determine if the operand
is Nan, infinity, denormal, zero or "ordinary" (valid, nonzero) and its
sign without decoding the instruction.

#### Configuration Register (P3, CR)
This register is used to set various configuration parameters including
the Model Number which is a Read-Only field. The format of the `CR` is:

```
 | MODEL NUMBER | RESERVED |
31             24         16

 | RESERVED | TYPE | CYC | REFR |
15         12     11    987     0
```

* REFR -- Indicates the refresh rate. With a processor cycle time of
10 Megahertz, rate = (REFR) * (?? microseconds).  The typical refresh
rate is about 15.625 microseconds which requires a REFR = 19.
REFR is set to 4 by the initialization microcode.

* TYPE -- The allows for different memory types.
  (TODO: table is unclear)

* CYC -- This field specifies the memory speed.
  (TODO: table is unclear)

* RESERVED -- These bits are reserved for future use.

* MODEL NUMBER -- This field is set by the manufacturing process and is
read only. It is used to distinguish different versions of the processor.

#### Processor Identification Register (P4, PI)
The `PI` is set by the operating system at initialization and allows
processors to identify themselves. The high order bit (31) indicates
whether the processor is in the hypercube array (0) or on a System Control
Board (1). The rest of the bits indicate the address or position of the
processor in the array or on an Interface Board.

#### Timeout Register (P5, TR)
Approximately every 100 microseconds the unsigned value in this
register is decremented. Thus it can count for about 5.1 days. If
the Timeout Register is zero an interrupt is generated whenever it is
enabled. Decrementing stops when the value in the Timeout reaches zero.

#### Output Ready (P6, OR)
There is a Ready flag for each output channel. When the flag is set to
one it indicates that the count register for that channel is zero and
the channel is ready to transmit more data. The format of the register is

```
 | OH R R R R R R R R R R R R R R R |
31                                 16

 | R R R R R R O9 O8 O7 O6 O5 O4 O3 O2 O1 O0 |
15                                           0
```

where `OH` means Output Host, `R` is Reserved for future expansion and `Oi`
is the Output port number i. The `OR` register is read only.

#### Input Ready (P7, IR)
For each input port there is a flag which when set indicates that the
corresponding count register has gone to zero, the channel has completed
its DMA function and is now ready to receive more data. The format of
the register is the same as the Output Ready register except I (Input)
is substituted for O (Output). The IR register is read only.

#### Output Enable (P8, OE)
This register has the same format as the Output Ready register but the
meaning of the flag is different. If a flag is set to one an interrupt is
generated when the corresponding output channel is ready to transmit. The
interrupt is suppressed if the flag is zero or if the Output enable (`OI`)
flag in the Program Status register is zero.

#### Input Enable (P9, IE)
When an input count register has become zero and the channel is ready
to receive, an interrupt is generated if the corresponding flag in this
register is set to one. If the flag is zero or the Input enable (II)
flag in the Program Status register is zero the interrupt is suppressed.

#### Input Pending (P10, IP)
If the count register of an input port is zero but there is a halfword
in the port that has not been stored in memory, the corresponding bit
in this register is set to one. This register is read only.

#### Input Parity Error (P11, PE)
Every halfword received is checked for parity. If an error is detected
then after the transmission is complete (the count register becomes zero)
instead of generating a "ready" interrupt, the corresponding flag in
this register is set and an "input error" interrupt is generated. This
register is read only.

#### Input Overrun Error (P12, IO)
If a halfword is received and overwrites a previously received halfword
before is can be stored in memory an error is noted. After the count goes
to zero instead of signaling a "ready" interrupt, the corresponding flag
is set to one and an "input error" interrupt is generated. This register
is read only.


## 4.5 Interrupts and Exceptions

The processor has a powerful vectored interrupt facility and generates
several kinds of interrupts: program exceptions, software facilities,
I/O signals and hardware errors. The program exceptions include integer
overflow and zero divide, the floating point exceptions, stack overflow
and address and reserved opcode faults. The software facility interrupts
are trap, breakpoint and trace. The Input Ready, Output Ready, Input
Parity and Input Overrun interrupts are the I/O signals. And the hardware
errors are Corrected and Uncorrectable memory errors and Processor Self
Test errors.

All interrupts (including the TRAP and breakpoint (BKPT) instructions)
have the same convention. There is an unsigned number associated with the
new interrupt (the argument of the trap instruction) that is multiplied
by eight to give the absolute location in low memory of the interrupt
vector. Each vector is eight bytes; the first four bytes contain the
absolute address vector (VA) of the interrupt handing routine and the next
four bytes are a new Program Status (NPS) value. When an interrupt is
generated the processor pushes the Program Counter (PC) and the Program
Status (PS) on the stack, sets the Program Status register to NPS and
the Program Counter register to VA. If the interrupt is signaling a
program exception (interrupts 3 through 12, see below) instead of saving
the PC, the processor pushes the address of the offending instruction
("previous PC") on the stack so that the exception handler can decode
the instruction. One reason decoding may be necessary is because
the IEEE Floating Point Standard requires the ability to construct a
result, store it where the instruction would have and then continue the
computation. When the interrupt handler is finished it executes a Return
from Interrupt (REI) instruction that pops the old PS and PC values
off the stack and into their respective registers. A TRAP instruction
with the appropriate number as its argument can simulate any interrupt
(except that the PC is always pushed on the stack with TRAP regardless
of its argument).

All interrupts are defined below. The number at the left is the interrupt
number.

### 4.5.1 Interrupt Definitions

* 0 RESERVED

* 1 T: Trace-at the end of the current instruction interrupt 1 is
generated in order to facilitate single step debugging.

* 2 BK: Breakpoint--when the one byte instruction BKPT is executed an
interrupt 2 is generated: this is used for breakpoint debugging.

* 3 IV: Integer Overflow--when the result is too large in magnitude for
the destination format.

  * add/substract: when the carry does not equal the sign bit
  * multiply: when the high order half of the product is not equal to the
sign extension of the result
  * divide: when the most negative number is divided by -1

The interrupt can be disabled but in either case the result stored in
the destination is the low order part of the result (in divide it is
the divident).

* 4 IZ: Integer Zero Divide--when the denomintor of an integer divide
or remainder is zero this interrupt is generated and no result is stored.

* 5 IX: Inexact Result--when a real result must be rounded the flag is set
in the PS and if not disabled the interrupt is generated. In either case
the correctly rounded result is first stored at the destination. Inexact
Result may occur at the same time as either Overflow or Underflow. If
this occurs the Inexact flag is set but the interrupt is suppressed and
either the Overflow or the Underflow interrupt is generated.

* 6 Underflow--if a real result is not zero but is smaller in magnitude
then the format's smallest normalized number then the UFflagisset in the
PS and an interrupt generated. However, an encoded result (the offset
is added to the exponent) is first stored at the destination.

* 7 FZ: Floating Zero Divide--when division of a nonzero real number by
zero is attempted no result is stored, the FZ flag is set in the PS and
an interrupt is generated.

* 8 OF: Overflow--if a real result is larger in magnitude than the largest
normalized number then an encoded result (the offset is subtracted from
the exponent) is stored and an interrupt is generated.

* 9 IN: Invalid Operation--the conditions that cause the IN flag in the
PS to be set and interrupt 9 to be generated are:
  * if a real operand is a Nan (except for branch on unsigned comparison
or branch on equal or unequal)
  * if both operands of a floating point divide are zero
  * if the "divisor" in a floating point remainder operation is zero
  * if the operand of square root is negative

* 10 UC: Unimplemented Opcode--when one of the reserved opcodes is used
this interrupt is generated.

* 11 AE: Address Error--if an address is larger than 2^17-1 then interrupt
11 is signaled.

* 12 SO: Stack Overflow--when the stack pointer becomes less than 2048
this interrupt is generated. This keeps the stack from growing into the
interrupt vector area in low memory.

* 13 TO: Time Out--when the Time Out Register is decremented to zero
interrupt 13 is generated.

* 14 CE: Corrected Memory (ECC) Error--if a memory error is corrected
during the execution of an instruction, at the end of the instruction
this interrupt is generated. This is useful for logging memory errors.

* 15 UE: Uncorrectable Memory (ECC) Error--if a memory error occurs that
cannot be corrected this interrupt is generated. Since this could occur
at many points during the execution of an instruction, the state of
the machine is undefined after this error. If this error recurs before
the previous one is handled then the internal ERROR flag is set and the
ERROR pin is set high. This is to warn of a potentially fatal condition.

* 16 OE: Operand Error--if a literal or immediate is used as the
destination of a result or any mode other than Register Direct is used
with a Repeat instruction then interrupt 16 is generated.

* 17-31 RESERVED

* 32-42 OR: Output Ready--when the count register of an output port
has gone to zero and the channel is ready to send another message, the
corresponding bit in the OR register is set and an interrupt generated
if it is not suppressed either in the PS (Program Status) register or
the OE (Output Enable) register.

* 43-62 RESERVED

* 63 ORH: Output Ready Host--this is the interrupt that is used with
the output port that is normally used for communicating with the host
(i.e. the various interface boards).

* 64-74 IR: Input Ready--these are the interrupts used to signal that an
input channel is ready to receive a message.

* 75-94 RESERVED

* 95 IRH: Input Ready Host--this interrupt is used with the input channel
that is usually used for communicating with the host.

* 96-106 IE: Input Error--if either a parity or an overrun error is
detected while receiving a message, after the completion the appropriate
one of these interrupts is generated.

* 107-126 RESERVED

* 127 IEH: Input Error Host--if an error is detected on the channel used
for host communication this interrupt is generated.

### 4.5.2 Error Flag

There is an internal Error flag that is tied to the Error pin that
indicates that the processor is in an unknown, inconsistent or failure
state. On resetting the processor the Error flag is initialized to one and
if the on-chip initialization sequence and subsequent diagnostic software
run successfully it can be cleared by software (EROF). It is also set by
consecutive unserviced Uncorrectable ECC errors. The Error flag and pin
can also be set and reset by the ERON and EROF instructions respectively.

## 4.6 Communication

There are 22 unidirectional direct memory access (DMA) I/O channels
on each processor, 11 for input and 11 for output. The Input ports are
numbered 0,1,...,9 and 31; while the Output ports are numbers 32,33,...,41
and 63. The input and output ports are normally used in pairs
to form 11 full duplex I/O channels are shown below:

```
{(0,32),(1,33),...,(9,41),(31,63)}
```

Ports 31 and 63 are normally used for communicating with the Host (on
any System Control Board). Ports 0 to 9 and 32 to 41 are used to build
the hypercube interconnection network. Numbers 10 to 30 and 42 to 62
are reserved for future expansion.

Each of the I/O channels has an address register, a count register, a
"ready" flag and an interrupt enable flag. In addition each input channel
has a parity error flag, an overrun error flag and a "DMA pending"
flag. Besides the enable for each channel there are two global enable
flags in the Program Status (PS) register. The II flag disables all
input interrupts (including errors) even if the corresponding channel
flag is enabled and the IO flag disables all output interrupts.

In order to send a message from a memory buffer on a given output channel
one first either checks its ready flag or enables its interrupt and waits
for a "ready" interrupt. As soon as the channel indicates that it is ready
(idle), the address register is set to point to the first (low) byte of
the message, which must begin on an even boundary, by executing a LPTR
(Load Pointer) instruction. The source operand of this instruction is the
address of the message buffer and the destination operand is an integer
whose value determines which of the channel registers is to be loaded:

* 0,1,...,9,31 are input channels (10,11,...30 are reserved)
* 32,33,..., 41,53 are output channels (42,43,..., 62 are reserved).

In order to start the automatic message output, the corresponding count
register must be set to the number of bytes in the message. (In this
version of the processor the low order bit is forced to zero in both
the address and the count registers; thus the message buffer must start
on an even byte boundary and be an even number of bytes long. No error
is signaled is a program violates this requirement.) This is done by
executing a LCNT (Load Count) instruction. The destination operand
indicates the register to be loaded as explained above for the LPTR
instruction and the source operand is the count value (an unsigned 32
bit integer). The LCNT instruction also resets the parity and overrun
error flags when setting up an input port. The message transmission
is automatic and as data is sent the address register is incremented
and the count is decremented by the number of bytes transferred. When
the count becomes zero the output stops, the ready flat is set and if
enabled the ready interrupt is generated.

In addition to sending a message on a single channel, the processor
has a powerful BROADCAST facility. In order to send a message over
several channels at once, one must first ensure that the desired output
channels are ready. Then a BPTR (Broadcast Pointer) instruction is
executed. Its source operand is the address of the message as in LPTR but
its destination operand is a 32 bit mask. Every bit position that is set
to one will cause the corresponding output channel address register to be
loaded. (Bit position 0 corresponds to output channel 32, position 1 to
channel 33, etc.) The message broadcast is started by executing a BCNT
(Broadcast Count) instruction whose destination operand is a mask as
explained above for the BPTR instruction and whose source operand is an
unsigned 32 bit integer equal to the number of bytes in the message. The
major advantage of broadcasting is that the sending processor only has to
access each transmitted datum once thus reducing the memory bandwidth used
by the DMA facility. The processor can only handle one broadcast at a time
so if a subsequent broadcast is attempted, even on different channels,
before the current one is finished the results will be undefined.

In order for a message to be transmitted successfully the corresponding
input channel of the receiving processor must first be set up with an
address to an input buffer and the same count as the output channel. One
way this can be accomplished is by using a software protocol that always
sends a single halfword as the length of the desired message and waiting
for the receiving processor to respond with a halfword code that indicates
"ok to send message". This protocol will work because the last halfword
that is sent remains available for DMA even if the receiving processor's
input channel is uninitialized (count=zero). The presence of this data
in the input channel is indicated by the corresponding bit in the INPUT
DATA PENDING register (which can be tested by software) being set. Thus
as soon as the count register is set to one, the halfword (either the
length or on "ok to send") is stored in memory.

Before attempting to DMA the data to memory that is in an uninitialized
input port the error (Overrun and Parity) flags must first be checked
or they will be lost. This is because the Load Count instruction clears
the error flags.

The processor recognizes two types of errors in communication. Each
halfword is sent with a parity bit and on reception a parity check
is made. Also if a halfword is received into a DMA channel before
the precious one is stored in memory an input overrun error is
detected. (Overrun can occur when the input count goes to zero before
the output count--a software error, or when too many messages are being
sent to the processor at the same time.) If either type of error occurs
the corresponding flat is set and when the input count reaches zero
instead of "ready", an "input error" interrupt is generated (if II is
set). A software error that is not detected by the processor occurs
when the output count is smaller than the input. In that case, after
the message is sent the input channel will simply hang. This condition
can be avoided by correct software or by setting up timeout conditions
using the Timeout Register.

## 4.7
## 4.8

* [Instruction set](instructions.md) (Section 4.7 and 4.8)

## 4.9 Processor Initialization
A processor can be initialized by either asserting the reset pin
or by executing a RSET instruction. The resulting initialization is
significantly different in the two cases. They are both described below.

### Hardware Initialization
Hardware initialization is done by asserting the reset pin and proceeds
in several steps:

* External requests are ignored.

* The ERROR/ pin is latched into bit 31 (the mode flag) of the ID
processor register which indicates whether it is an I/O processor (0)
or an array processor (1). If the ERROR/ pin is grounded, bit 31 is set
to 1 indicating that the processor is on an I/O board. A floating ERROR/
pin will cause bit 31 to become 0 which implies that the processor is
on an array board. The mode flag is latched when the reset goes away.

* The processor performs self tests on the various internal units and
sets memory locations 4 to 2048 to zero. Location 0 is set to 1 (Word)
if the processor passed its tests and -1 if it failed.

* The processor state is set to zero except for

  *  bit 31 in the ID register (see above)

  *  bits 24-31 of the Configuration register are set by the manufacturing
process

  *  the stack pointer (SP) and the fault register (FR) which are undefined

  *  input and output ready bits are set to 1 and interrupts are disabled.

* The "shadow" ROM on the processor is activated and the procedure
listed below is executed. Its function is to

  *  determine whether it is an I/O or array processor

  *  if it is an array processor then

    *  it waits to receive a value (halfword) which is the length of the
actual message (see 3)

    *  it replies with a status message indicating that it is ready to receive the full message

    *  it receives the message (the full software initialization software), loads it starting at location 0 and jumps to location 1024

  *  if it is an I/O processor it waits until the central processor writes a nonzero value in location 0 and then jumps to location 1024 where the I/O initialization software would have been placed.

  *  a JMP (jump) to the initialization software is executed (the jmp disables the shadow ROM until it is enabled by another reset signal). The functions performed by the initialization software should include a full set of diagnostics.

### 4.9.2 Initialization Procedure (shadow ROM)

The code in the on-chip shadow ROM is listed below with comments.

```
! During shadow ROM execution all interrupts are disabled including
! interrupts that are not normally maskable;
RSET ;

! The RAM chips need 8 refresh cycles to initialize themselves. The
! refresh rate starts at one refresh every 8 cycles since the Configuration
! register is set to zero on reset. We idle for the required 64 cycles by
! looping on RSET 10 times. Each loop takes 7 cycles (3 for the RSET and
! 4 for the REP);

MOVW #11,R0;
REP R0;
RSET;

! The refresh rate is lowered to every 40 cycles by writing a 4 in the
! Configuration register. This is conservatively high but the operating
! system can lower it further if the processor clock rate justifies it;

LDPR #4,#CONFIG;

! Memory is now initialized with correct ECC bits by writing zero to
! every location. Since the Configuration register is initialized to
! assume 16k×4 memories, only the first quarter of memory is initialized
! by writing 8191 words. If the operating system changes the Configuration
! to 64k×4, then it should initialize the last 3/4 of memory;

MOVW #8191,R0;
MOVW #O,R1;
REP R0;
MOVW #O,(R1)+

! A self test belongs here. The result is encoded and stored in memory
! at location 4. A -1 means everything is fine;

MOVH #-1,4;

! Bit 31 of the ID Register is initialized when the reset pin is asserted
! with a one if the processor is an I/O processor or a zero is the processor
! is an array processor. I/O processors are initialized from memory while
! array processors are initialized by the serial ports;

STGPR #IDREG,R0;
BL IOINIT;

! Array processor initialization waits for a port to receive a
! message. The code below assumes that only one port will try to initialize
! the processor. If messages come in at two ports exactly at the same time,
! the code may not work;

PROCINIT: STPR #INPEND, R0  ! Are any incoming messages pending?
          BF PROCINIT       ! No, try again
          FFOW R0, R1       ! Yes, R1 gets the port number

! Initialize the port so DMA transfer of a two byte message to location
! 2 will occur;

LPTR #2,R1;
LCNT #2,R1;

! Compute in R3 the corresponding output port for a reply;

MOVW R1,R3;
ADDW #R2,R3;

! Wait for incoming message DMA to complete;

INWAIT1: STPR #INRDY, R2   ! Store input ready flags in R2
         BITW R2, R0       ! Test the appropriate flag
         BE INWAIT1        ! Loop until port is ready

! Start the output port DMA. The message will be the two byte self test
! status in location 4;

LPTR #4,R3;
LCNT #2,R3;

! Reinitialize the same input port to receive the contents of memory;

         LPTR #8, R1      ! The message will start at location 8
         LCNT 2, R1       ! for number of bytes indicate by the first message

INWAIT2: STPR #INRDY, R2  ! Wait for input DMA to complete by
         BITW R2, R0      ! testing the appropriate ready flag
         BE INWAIT2       ! and looping back until ready (done)

! Jump to a preset location (1024) to begin execution from memory. The
! JMP resets the "shadow ROM active" flag;

JMP 1024;

! I/O processor initialization. Wait for memory location 0 or 1 to go
! nonzero. The external processor that loads the memory image must wait
! at least xxx cycles after the RESET signal has gone away;

IOINIT: BITH #-1,0   ! Test halfword at location 0
        BE IONIT     ! Loop back until it becomes non-zero

! Jump to a preset location (1024) to begin execution from memory. The
! JMP resets the "shadow ROM active: flag;

JMP 1024;

! End of shadow Rom code;
```

```
MEMORY                      ADDRESS
0                           0
Initial message length      2
test results                4
reserved                    6
DMA (int? ? vector)         8
etc
```

