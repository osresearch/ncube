# 5. THE SOFTWARE

## 5.1 Introduction

There are two levels of operating software in the system: the Monitor
(in EPROM) and the Operating System. The monitor is a simple, single user
system that is in effect when the system is powered on. The Monitor
uses terminal 0 and provides extensive diagnostic and management
functions. The Operating System, IX™ (IX is a trademark of NCUBE
Corporation), is automatically invoked if the system is in Normal mode
and passes the diagnostic tests. IX™ is a fully protected multiuser,
multitasking operating system with complete resource management including
memory, main array, graphics and file system. The file system has a
hierarchical structure and is distributed across all the disk drives in
the system. Thus, a user can access his files regardless of which terminal
(or Peripheral Controller) he uses.

In may ways the Operating System is similar to UNIX™ (UNIX is a Bell Laboratories trademark), and therefore will not be described in detail herein. The IX™ System does, however, have additional facilities including:

* system temperature sensing
* distributed file system
* array management
* uniform file protection

The IX System is described in section 5.3.

## 5.2 The Monitor

### 5.2.1 Introduction

The Monitor is contained in the system EPROM and is invoked when the
system is powered on. The Monitor always communicates with Terminal 0
on Peripheral Controller 0 (the System Console) for displaying messages
and receiving commands. When the system mode switch on the front panel
is in the "Normal" position, the Monitor runs the diagnostics and boots
the Operating System (if the diagnostics run successfully). If the mode
switch is set to "Diagnostic", the Monitor goes into a single user system
after successfully running the diagnostics. The Monitor system provides
a large range of offline diagnostic and backup facilities.

The Monitor consists of two parts: the ROM Monitor and the RAM
Monitor. They are both in the system EPROM but the ROM Monitor uses no
RAM even for stack space while the RAM Monitor, when invoked, is copied to
RAM and uses RAM for data. The ROM Monitor starts the system and executes
the diagnostics up to the memory test phase. If memory test passes, the
RAM Monitor is automatically invoked; but if it fails, the system stays
in the ROM Monitor and a few simple commands are available (see 5.2.3).

### 5.2.2 Monitor Diagnostics

The facilities tested by the Monitor diagnostics are listed below in order.

1) The two front panel LEDs are turned on and the ROM Monitor is started
2) The EPROM contents are verified (a checksum is computed)
3) All I/O devices except the disk controller are initialized.
4) The Serial Channel for Terminal 0 is tested

5) If (2) or (4) fail, the LEDs remain on and the system hangs indicating
that Peripheral Controller board 0 is bad.

6) If (2) or (4) pass, then the LED labeled "STATUS 2" is turned off,
appropriate characteristics are set for Terminal 0 (19200 baud rate,
etc) and the system startup message, " Parallel Processor Peripheral
Subsystem", is displayed.

7) System memory (RAM) is tested and any errors (including corrected ECC
errors) are displayed. If there are memory errors or the Diagnostic Mode
is on, the system stays in the ROM Monitor, prints "ROM-Only Diagnostic
Monitor" followed by a "$" prompt and waits for user commands. If there
are no memory errors and the system is in Normal Mode, the RAM Monitor
is invoked and the diagnostics proceed.

8) The Disk Controller is tested if any disks are connected.

9) Power Supply status ad control signals are checked.

10) The Printer state is checked.

11) The System ID and slot numbers are checked.

12) All Temperature Sensors are tested and the temperature is displayed. If the temperature is above 38 degrees C. the system is shut down.

13) The Real Time Clock state and operation are checked.

14) All Interrupt Controllers are tested for state and response.

15) The remaining Serial Channels are tested.

16) The Timer (8253) is tested.

17) The Floating Point Processor (80287) is tested.

18) The DMA controller is checked.

19) Any SBX Modules connected to the system (as indicated by an EPROM
table) are tested.

If the system is in Normal Mode and a disk is connected then

20) The disks are started and the controller is tested.

21) The disks are checked and fixed if a system crash was the cause of
the last shutdown.

22) The Operating System is booted.

Otherwise

20) The system stays int eh RAM Monitor, a ">" prompt is displayed and
the system waits for a command.

### 5.2.3 ROM Monitor Commands

Since the ROM Monitor does not use RAM, its commands are few and
simple. They are listed below and are invoked by typing the first letter
in the command name. A "return" causes a new "$" prompt to be displayed. A
"t" can be typed at any time and whatever is happening will be aborted
and a new prompt displayed. The operand specifications for the commands
are defined as follows

ADDR consists of two 4 digit hexadecimal numbers separated by a
colon. The first number is the segment selector and the second is the
offset.

```
LENGTH  \
IOADDR   } are 4 digit hexadecimal numbres
VALUE   /
```

SEG MAX is the number of 64 Kbyte segments of memory to be tested
(starting from memory address 0).

#### COMMANDS

##### `continue`

Restarts the disk operating system after a "debug" stop.

##### `display <ADDR>,<LENGTH>`

A section of memory from ADDR to ADDR+LENGTH-1 is displayed in the
following format:
```
ADDR hhhh hhhh hhhh hhhh hhhh hhhh hhhh hhhh <asci>
```
where ADDR is the beginning address, "`hhhh`" represents a 16 bit
word in hex and the ASCII equivalent of the 8 words is also displayed
("." represents unprintable characters).

##### `goto ram monitor`

The RAM Monitor is booted.

##### `help`

The list of ROM Monitor commands and operands is displayed.

##### `input <IOADDR>`

The value at I/O address IOADDR is displayed. Typing a "line feed"
repeats the command with the same operand and a "return" terminates it.

##### `memory test <SEG MAX>`

SEG MAX 64 Kbyte segments of memory are tested starting at memory address 0.

##### `output <IOADDR>,<VALUE>`

VALUE is written to I/O address IOADDR. A "line feed" repeats the command at the same address but allows a different VALUE to be typed and "return" terminates it.

##### `power down`

The system is powered down.

##### `set <ADDR>`

The value in memory at ADDR is displayed and can be altered by typing a
new value. A "line feed" advances to the next word in memory and repeats
the command. A "return" terminates it.

### 5.2.4 RAM Monitor Commands

The RAM Monitor is invoked automatically if the diagnostics pass the
memory test or explicitly by typing "`g`" in response to the ROM Monitor
prompt. The RAM Monitor Commands are of four types: general, debugging,
disk control or tape control. The general commands are invoked by typing
the first letter of the command name. The debugging, disk control and tape
control command are invoked by first typing "`y`", "`x`" or "`t`" respectively,
followed by the first letter of the specific command name. If "return"
is the first character typed, a new monitor prompt, "`>`", is printed and
the command analyzer is restarted. A "`c`" can be typed at any time and
regardless of what is happening, it will be aborted and a new prompt
will be displayed.

The operand specifications are the same as the ROM Monitor's (see 5.2.3)
but with several additions.

## 5.3 The Operating System

### 5.3.1 Overview

The operating system, IX™, is a high performance UNIX-style interface to
the hardware. It supports multiple users, including password and billing,
and multitasking. The editor, NMACS, is screen oriented and is similar
to a simplified version of EMACS. The file system is the most prominent
feature of the operating software because nearly every system resource is
treated as a type of file. The file system is hierarchical like UNIX but
has extensive mechanisms for file protection and sharing. The operating
system treats memory as a collection of segments that can be allocated
and shared. Processes are created and scheduled (priority, round robin)
by the system and provide part of the protection facility. There is a
debugger and a linking loader. One of the unique facilities of the IX™
system is the management of the main processing array. It is managed as a
device and each process requests subsets of the array which are allocated
according to availability. Fault tolerance is supported by the system it
periodically runs diagnostics on the array and if any nodes fail, they
are mapped out of the allocatable resource and the operator is informed
of the fault. Only the facilities listed above which are essential to an
understanding of the present invention are described in more detail below.

### 5.3.2 File System
The file system is the user's uniform interface to almost all of
the system resources. The two main entities in the file system are
directories which provide the structure and files which contain the
data. Most resources (e.g. printers, terminals, processing array) are
treated as devices which are simply one type of file. A file has a name
which both uniquely identifies it and indicates its position in the file
structure. Files have a set of operations defined that can be performed
by a user having the requisite privileges.

### 5.3.3 Editing
There are three editors in the IX™ system. One is a line editor called
"ed". It is compatible with the "ed" line editor in UNIX. Another is
a stream editor whose name is "sed". Sed is also compatible with the
UNIX stream editor of the same name. For detailed information see the
extensive literature on standard UNIX systems (e.g. B. W. Kernighan's
books: "A Tutorial Introduction to the ED Text Editor" and "Advanced
editing on UNIX").

The third editor is a screen editor called "nm" (NMACS). It is similar
to the widely used screen editor EMACS.

### 5.3.4 Memory Management
The system of the present invention provides a segmented virtual memory
environment. The virtual address space is 230 bytes. Main memory is
treated as a set of segments on 256 byte boundaries. The operating
system provides allocation, deallocation, extension (segments can grow
to 64 Kbytes), compaction and swapping functions. The system relies
on the Intel 80286 memory management hardware. Memory is allocated and
deallocated with the system call "core".

### 5.3.5 Process Management
Processes are managed by the operating system as the fundamental units of
computation. They are created, scheduled, dispatched and killed by the
system in a uniform way for all processes. When the operating system
is booted the primary, highest priority system process, called the
MCP (Master Control Program), is dispatched. It initializes the system
including dispatching background system processes (like a print spooler)
that it gets from a system initialization file, watches terminals and
creates processes. It also cleans up and shuts down the system when
power failure or overheating is detected.

Whenever a user logs on the system, the MCP checks his name and
password. If he is an authorized user and the password is correct, the
MCP creates a process for him. The parameters of the process are taken
from his "log on" file that is created by the system administrator. These
parameters include the priority, the initial program (usually the shell),
the preface (user's root directory) and billing information. The logon
file for "`user1`" is named `/sys/acct/user1`.

A process is represented by a data structure in memory. This structure,
called a process object, has the following entries:

* state: This is the area where register values including segment pointers
are saved when the process is not executing.

* condition: The conditions that a process can be in are
  * runnable
  * waiting for memory allocation
  * waiting for array allocation
  * waiting for a message
  * waiting for error handling
  * etc

* code and data: These entries point to the code and data for the
process program.

* preface: This is the name of the root directory of the process.

* directory: This is the name of the current working directory

* priority: A number between 1 and 255 indicating the relative priority
of the process (255 is the highest priority).

* time: The maximum number of clock ticks this process can run before it
must be rescheduled.

* rights: A process can be granted (1) or denied (0) various rights
according to the setting of the flags listed below
  * create links
  * delete links
  * create processes
  * kill processes
  * superuser

* owner: Name of the user who created the process.

* open files: This is a table of descriptors for each or the open files of
the process. When a process is created the first three entries (channel
numbers 0,1 and 2) are initialized to the following:
  * `0` standard input file
  * `1` standard output file
  * `2` standard error file

When a new process is created, its owner, priority and rights are
either initialized by the logon file or are inherited from the creating
process. The priority and rights can be reduced or restricted but not
increased or expanded.

All processes in the system are linked together in the process list. When
it is time to dispatch a new process the list is searched starting
from the process that was most recently running. The search finds and
dispatches the highest priority process that is in runnable condition. If
there is more than one the last one found is dispatched. The process
runs until one of three events occurs:

1) the process time slice is exhausted
2) the process must wait for some event such as a message or disk operation
3) another higher priority process becomes runnable.

Thus, the process management system implements preemptive, priority,
round robin scheduling.

Process System Calls

There are a set of operations for process management. These system calls are:

* `frun`: run a file
* `getpcs`: get priority, rights, time, condition, owner, etc
* `chprot`: change protection or rights
* `alarm`: set process alarm clock
* `endpcs`: terminate a process
* `endump`: terminate and dump
* `pause`: suspend a process
* `psend`: send a message or signal
* `vector`: set interrupt vector

### 5.3.6 Device Management
The system treats almost all resources as devices which are simply a
special type of file. The devices include disk drives, tape drives,
printers, graphics hardware, interboard bus, SBX interfaces and the
hypercube array. Devices are managed as are other files with open, close,
read and write calls. For special operations that do not fall easily in
those categories, the operating system supports a "special operation"
call. These special operations are things such as setting terminal
parameters and printer fonts.

#### 5.3.6.1 Hypercube Array
The system treats the hypercube array as a device type file. Consequently,
it is allocated with an "open" command, deallocated with "close" and
messages are sent and received with "write" and "read" respectively. One
of the powerful features of the hypercube is that it is defined
recursively and so all orders of cube are logically equivalent. When
allocation is requested the user specifies in the "open" call the
subcode order (N) he needs. If a subcode of that order is available,
it is initialized and the nodes are numbered from 0 to 2N-1. The subcube
is allocated as close as possible to the Peripheral Controller that the
user's terminal is connected to. If no subcube of that size is available,
the "open" returns an error condition. This allows the user to either
wait for a subcube of order N to become available or to request a smaller
one. Once allocated the user owns the subcude until his process terminates
or he explicitly deallocates (closes) it. A degree of fault tolerance is
achieved in the system because the operating system periodically runs
diagnostics on the Hypercube Array and if a node fails, it is mapped
out of the allocatable resource. However, the rest of the nodes are
available for use. (A faulted node also causes the LED attached to its
Array board to be turned off indicating a condition requiring service.)

#### 5.3.6.2 Graphics System
The graphics boards are also treated as device files and are allocated
and managed by each user with file system calls. The special operations
that are defined for the graphics devices are the graphics operations
that the hardware itself supports such as line and circle drawing,
fill-in, panning, etc.

#### 5.3.6.3 SBX Interface
Each System Control board in a system has three SBX connectors. One
is used for the cartridge tape controller and another is dedicated to
providing the Interboard Bus (a bus for moving data between Peripheral
Controllers). The last SBX connector is available for custom parallel
I/O applications. There are many potential uses for the SBX Interface
including networking, 9 track tape drive controller, etc. Regardless of
whit it is used for, it will be treated as a device by the operating
system. Consequently, it is only necessary to write the appropriate
device driver in order to use the standard file system calls for device
management.

### 5.3.7 Initialization
The first level initialization is accomplished by simply turning on the
system in Normal mode. When the operating system is booted, it looks
for a configuration file called `/sys/startup`

If the "startup" file exits, a shell is created that runs it as a command
file. One example of a command that would very likely be found in the
startup file is

```
/sys/bin/spool > /sys/spool.log
```

and which causes the print spooler to be run as a parallel process.

In addition, the system administrator must perform certain functions
such as creating logon files for each user.

In addition to initializing the operating system, the hypercube array
must be initialized. The initialization of individual processors is
discussed in section 4.9. In this section an algorithm for initializing
the system is described. The algorithm is based on a tree structure and
can be more easily illustrated than described. The diagram below shows
the initialization responsibility for each processor assuming there are
16 processors. The binary numbers are the processor ID's and the decimal
numbers represent the stage in time of the initialization.

TODO - figure with binary tree?

The assembly language code that implements this algorithm is listed below.

```
       MOVW ID,R1        ;ID is memory location
                         ;containing the processor
       IDLDPR R1,IDREG   ;the ID is loaded into the ID
                         ;processor register
       FFO R1,R2         ;R2 = # of trailing zeros i
       IDSUBB #1,R2      ;
       JL END            ;no trailing zeros => this
                         ;processor is a leaf on the graph
LOOP:  MOVW #1,R3        ;compute ID of neighbor by
                         ;complementing one of the
       SFTW R2,R3        ;trailing zeros
       MOVW R1,R4        ;
       XORW R3,R4        ; R4 = new ID{send message length to port #(R2)}
                         ; {receive status; use timeout}
                         ;     a. dead (timed out)
                         ;     b. failed self test
                         ;     c. parity errord. alive and well
                         ; {if alive MOVW R4,ID;put new ID in memory}
                         ; {send copy of code and new ID to R2}
       REPC R2           ;
       JMP LOOP          ;
END:                     ;{look for responses and EROF}
```

### 5.3.8 Operating System Commands
This section specifies the commands in alphabetic order that are
implemented in the operating system:

* `ADB`:           debugger
* `AS`:            assembler (80286)
* `ASN`:           assembler ( )
* `AT`:            later execution
* `CAT`:           catenate and print
* `CD`:            change directory
* `CHMOD`:         change protection
* `CMP`:           file compare
* `CN`:            change name
* `CP`:            copy
* `DATE`:          print date
* `DC`:            desk calculator
* `DF`:            disk free space
* `DIFF`:          diff. file compare
* `DU`:            disk usage
* `ECHO`:          echo arguments
* `ED`:            line editor
* `ET`:            terminal emulation
* `F77`:           Fortran 77 (80286)
* `F77N`:          Fortran 77 ( )
* `GREP`:          pattern search
* `HELP`:          help
* `HD`:            hex dump
* `KILL`:          kill process
* `LN`:            make a link
* `LS`:            list directory
* `MAIL`:          local mail
* `MAN`:           print manual
* `MESG`:          messages (yes/no)
* `MORE`:          paged display
* `MOUNT`:         mount file system
* `NM`:            screen editor (NMACS)
* `NSH`:           shell (see SH)
* `PASSWD`:        change password
* `PR`:            print file
* `PS`:            process status
* `PSTAT`:         system status
* `PWD`:           working directory
* `RM`:            remove file
* `RMLN`:          remove link
* `ROFF`:          text formatter
* `SA`:            system accounting
* `SED`:           stream editor
* `SH`:            shell
* `SHUT`:          invoke RAM Monitor
* `SLEEP`:         suspend process
* `SORT`:          sort or merge
* `SPLIT`:         split a file
* `STTY`:          set terminal
* `TEE`:           pipe with file save
* `WAIT`:          wait for completion
* `WALL`:          write to all users
* `WHO`:           display system users
* `WRITE`:         send text

### 5.3.9 File Formats and Conventions
In this section the data structures that are used in the operating system
are specified. Most of the structures are used for managing
```
  processes
    procobj
  files
    cactab
    dir
    file
    opntab?
  system
    sysdata
    sysdev
```

To fully understand some of these structures it is necessary to have
a working knowledge of the 80286 (see iAPX 286 Programmer's Reference
Manual from Intel for details). Some of the important characteristics
of the 80286 are:

* Memory is treated as a set of variable length (up to 64 Kbytes) segments.
* Each segment has a virtual address that consists of two parts (each
part is two bytes)
  *  an index (segment selector) into one of two tables of segment
  descriptors: the Global Descriptor Table (GDT) and the Local Descriptor
  Table (LDT).
* The hardware recognizes some special segments and has support for fast
task switching. These include the GDT, the LDT and the Task State Segment
(TSS).

In the specifications below the abbreviations have the following meanings:
C=Constant, B=Byte, H=Halfword, W=Word, D=Double Word. If a Word is an
address in memory then it consists of the two parts described above. If
a Word is a disk address then it has three parts that designate cylinder,
head and sector.

```
CACTAB(5)
NAME
cactab - format of the sector buffer cache table
```

DESCRIPTION
The file system maintains a cache of buffers for disk sectors to
minimize the actual disk traffic. The number of buffers is set by the
system variable "caccnt". When the buffers are all full and a sector
must be read that is not in a buffer, the least recently used buffer is
used for the new sector. Therefore, the buffers are arranged in a linked
list with a system variable "lruptr" pointing to the least recently used
buffer. The entries in the sector buffer cache table (which is located
at "cactab") are called sector buffer descriptors ("sucbufdes") and are
specified below.


```
secbufdes -- format of a sector buffer descriptorH       caclruf  ;least recently used link forwardH       caclrup  ;least recently used link backwardB       cacst    ;buffer status (see below)/access countB       cacmod   ;buffer modifiedH       cacchn   ;lock chain for bufferH       cacchne  ;H       cacdev   ;device number (*2) for bufferW       cacadr   ;disk address of bufferbufstC       unchanged = 0               ;buffer not saved on swap ????C       modified = 1               ;buffer modified (saved on swap)SEE ALSOsysdata(5)DIR(5)                      DIR(5)NAMEdir -- format of a directory
```

DESCRIPTION
Each node in the file system hierarchy is a directory. A directory contains pointers to files or other directories. The first name in every directory is "." and refers to itself. Names of files and directories can have at most 24 characters from the set (a-z,0-9,$,--,.). A directory is made of one or more directory sectors ("dirsec"). A directory sector contains up to 32 entries, each of which is 32 bytes. The first entry contains defining information about the directory. The rest of the entries, called directory pointers ("dirptr"), are pointers to files or other directories. The structure of directory sectors and directory pointers are specified below.

```
______________________________________dirsec -- format of a directory sectorH        dirid    ;non-ASCII magic number (F4F1) that is             ;checked on every reference to the dirsecB        level    ;level of directory in hierarchyW        nxtdir   ;disk address of next dirsec in this nodeW        dirdate  ;creation date of directoryH        dirown   ;directory owner numberB(18)    res      ;reservedW(8)     dirptr   ;first of variable number (up to 31) of             ;pointers to files or directoriesdirptr -- format of a directory pointerB(24)    name     ;24 character name of file or directoryH        rights   ;rights associated with name (see below)H        ddirdev  ;device for ddir pointer (if not null             ;dirptr points to device root)W        nodptr   ;disk address of next node or fileDIR(5)                      DIR(5)rights --    rights (and type) associated with name    (0 = granted, 1 = denied) bit 0:         type of object named (00 = file/device,bit 1:         01 = link, 10 = ddir, 11 = directory)bit 2:   change rightsbit 3:   reservedbit 4:   delete filebit 5:   execute filebit 6:   write filebit 7:   read filebit 8:   change names in directorybit 9:   create and change linksbit 10:  use directory for file lookupbit 11:  delete entry in directorybit 12:  delete directorybit 13:  execute from directorybit 14:  create file in directorybit 15:  read contents of directorySEE ALSOfile(5)FILE(5)                     FILE(5)NAMEfile -- format of a data file______________________________________
```

DESCRIPTION
A data file consists of one fails descriptor sector ("fildessec") and as many file pointer sectors ("filptrsec") as necessary. A file descriptor sector contains a 32 byte header and up to 248 pointers to sectors containing data. A file pointer sector contains a 12 byte header and up to 252 pointers to data.

```

______________________________________fildessec -- format of a file descriptor sectorH      fildid  ;non-ASCII value (F9F1) used for validationB      filtyp  ;file type (see below)B      subtyp  ;file sub type (not interpreted by system)W      nxtptr  ;disk address of the next pointer sectorH      filver  ;file version numberH      filock  ;file lock (1=read, 2=write)W      fildate ;file creation dateW      altered ;date file last alteredW      acssed  ;date file last accessedW      filsiz  ;file size (0 to 4294967295 bytes)H      filown  ;file owner numberH      acccnt  ;file access countW      fdatptr ;first of up to 248 disk addresses of          ;sectors containing data (fdatptr = 0 is          ;an invalid pointer)filtyp -- file type definitions (0 to 15 reserved for)C        nulfil  = 0 ;C        devfil  = 1 ;a device type fileC        sysfil  = 2 ;a system fileC        binfil  = 3 ;a binary fileC        relfil  = 4 ;a relocatable object file typeC        exefil  = 5 ;and executable fileC        txtfil  = 6 ;a text (ASCII) fileFILE(5)                     FILE (5)filptrsec -- format of a file pointer sectorH      filpid  ;non-ASCII value (FAF1) used for validationH      secbas  ;sector count base (number of data sectors          ;in file behind those pointed to here)W      nxtptr  ;disk address of next pointer sectorB(8)   res     ;reservedW      fdatptr ;first of up to 252 disk addresses of          ;sectors containing dataSEE ALSOed(1), dir(5), opntab(5)OPNTAB(5)                 OPNTAB(5)NAMEopntab -- format of an open file table______________________________________
```

DESCRIPTION
Whenever a file is opened an open file descriptor ("opfildes") is created and entered into the open file table ("opntab") of the process that invoked that "open file" call. The call returns the index, called the channel number or "fildes", of the descriptor in the open file table. Thereafter, file operations refer to the file through this channel number. There can be up to ??? open files in a process at one time. An open file table consists entirely of open file descriptors so it suffices to specify the format of the descriptors.

```

______________________________________opfildes -- format of an open file descriptorB     opnst    ;open file status (see below)B     opntyp   ;type of file (see below)H     opndev   ;device table index for fileW     opnptr   ;disk address of first pointer sector for          ;fileW     opncpt   ;disk address of pointer sector for current          ;byte pointerW     opnpos   ;current byte position in fileW     opnsec   ;disk address of sector for current byte          ;positionH     opnrgt   ;access rights for open fileH     opninx   ;index into pointer sector for current          ;sectorH     opndir   ;pointer to directory sector for fileH     opntem   ;temporary areaH     opntem2  ;temporary areaB     opnstps  ;count of number of link jumpsB     opndpt   ;current depth of name searchopnst -- definition of open file statusC       open    = 1 ;C       altered = 2 ;opntyp -- definition of open file typeC       file    = 0 ;C       device  = 1 ;C       pipe    = 2 ;SEE ALSOfile(5), open(2)PROCOBJ(5)                PROCOBJ(5)NAMEprocobj -- format of a process object______________________________________
```

DESCRIPTION
Each process is the system is represented by a data structure called a process object. The process object is represented by four entries in the Global Descriptor Table (GDT). These four entries are collectively called the process descriptor and all process descriptors are chained together. The first entry is an "invalid" segment descriptor that contains process information: a link to the next process descriptor in the chain, process id, priority and status. The other three entries are valid segment descriptors. The process descriptor and process object formats are defined below.

```

______________________________________process descriptorH      proc link  ;offset in the GDT to next process             ;descriptorH      proc id    ;unique identifier for the processB      proc priority             ;scheduling priority (1 is highest)B      proc null  ;=0 for invalid segment descriptorH      proc status             ;see belowW(2)   TSS desc   ;descriptor for Task State SegmentW(2)   LDT desc   ;descriptor for Local Descriptor             ;TableW(2)   procobj desc             ;descriptor for process objectproc statusC     run      = 0    ;process is runnableC     newpcs   = 1    ;new processC     interr   = 2    ;process is stopped by errorC     bufwat   = 20   ;waiting for memory bufferC     dacwat   = 21   ;waiting for device allocationC     secwat   = 22   ;waiting for sector bufferC     dskwat   = 23   ;waiting for disk operationC     endwat   = 24   ;waiting for process terminationC     trdwat   = 25   ;waiting for tty readC     twrwat   = 26   ;waiting for tty writeC     mcpwat   = 27   ;MCP idleC     ptrwat   = 28   ;waiting on printerC     cacwat   = 29   ;waiting for disk cacheC     diverr   = 30   ;divide or overflow stopC     trderr   = 31   ;trace stopC     brkerr   = 32   ;breakpoint stopC     ovrerr   = 33   ;integer overflow stopC     ptrerr   = 34   ;protection error stopPROCOBJ(5)                PROCOBJ(5)procobj -- format of a processor objectB(44)   TSS      ;Task State SegmentW       strtim   ;process start timeH       cpustt   ;time at start of time sliceH       newsflg  ;new sector allocation flagW       cputim   ;execution time (.01 sec)                                 ProcessW       dskrds   ;disk read count     StatisticsW       dskwrs   ;disk write countW       iocnt    ;other I/O count (bytes)W       corsiz   ;memory size (Kbytes)W       kcorsec  ;kilo-core-secW       cortim   ;mem size start timeH       pcsown   ;process ownerH       parent   ;process parentH       filmax   ;max open files (*32)H       segmax   ;LDT size            ProcessH       privlg   ;privilege bits (see below)                                 Param-H       ldtadr   ;offset of LDT       etersB(24)   pcsnam   ;process program nameB(256)  pcsdir   ;process prefaceB(256)  insdir   ;current working directoryH       devchn   ;chain for device waitH       alrmchn  ;chain for alarm wait                                 SystemW       alrmtim  ;alarm time          WorkH       lckchn   ;chain for locks     Area inH       pcswatid ;process being waited on                                 ProcessB(36)   devsav   ;device I/O data areaB(24)   namsav   ;name work areaW       bkptvec  ;breakpointW       trcvec   ;traceW       fpvec    ;floating point errorW       intovec  ;integer error       ProcessW       abrtvec  ;abort ( C)          InterruptW       killvec  ;kill process        VectorsW       protvec  ;protection error    (can beW       pipvec   ;pipe error          set byW       msgvec   ;message from process                                 "signal"W       intvec   ;interrupt           systemW       alrmvec  ;alarm               call)W       illvec   ;illegal instructionW       res0vec  ;W       res1vec  ;W       res2vec  ;W       res3vec  ;B(80)   stk287   ;save area for 80287 stackB(16)   stat287  ;save area for 80287 statusB(??)   opntab   ;open file tableB(??)   LDT      ;Local Descriptor TablePROCOBJ(5)                PROCOBJ(5)privlg (process privilege bits: 0=granted 1=denied)bit 0:bit 1: : :bit 12:   change memory sizebit 13:   create directoriesbit 14:   create linksbit 15:   superuser (all rights)______________________________________
```

The process statistics can be used for billing and are accessed with the "getpcs" system call. The process parameters are set at process creation from the log on file or they are inherited from the creating process. The open file table contains open file descriptors. Whenever a file is opened a descriptor for it is entered in this table and its index (channel number) is returned. Whenever a new segment is allocated to a process, a descriptor for it is entered in the LDT.

There are several system variables that are used in process management. These include

system state

last process: a pointer to the last dispatched process; it is used to implement round robin scheduling

process object: the segment selector for the current process object

time left: the number of clock ticks left in the time slice of the current process.

```

______________________________________SEE ALSOalarm(2), frun(2), endpcs(2), endump(2), getpcs(2), psend(2),pause(2), vector(2), sysdata(5), files(5)SYSDATA(5)               SYSDATA(5)NAMEsysdata -- format of system data file______________________________________
```

DESCRIPTION
The system data ("sysdata") defines the parameters statistics and variables of the system. The data can be read by invoking the system call "getsys". The format of the system data is given below.

```

______________________________________sysdata -- format of the system data fileH    sysid    ;sector id (F0F1) for validationH    basyr    ;base year for system date (1981)W    sysdate  ;system creation dateH    memsize  ;main memory size (/256)H    timzon   ;time zone correction factor                                  SystemH    crashf   ;system crash flag       Param-H    maxtmp   ;maximum system temperature                                  etersW    systemid ;system and board id numberH    sysbrd   ;bits for active system boardsH    grpbrd   ;bits for active graphics boardsH    acctflg  ;accounting enable flagH    shut     ;system shutdown flagB(??)res      ;reservedW    lastim   ;last system startup timeW    stime    ;system startup timeW    systim   ;system overhead timeW    idltim   ;system idle timeW    ecccnt   ;ecc error countW    arytim   ;array use timeH    badcnt   ;number unknown interrupts en-         ;counteredW    totsys   ;total system overhead time                                  SystemW    totidl   ;total system idle time  Statis-W    totecc   ;total ecc error count   ticsW    totary   ;total array use timeH    totbad   ;total unknown interrupts en-         ;counteredW(2) totup    ;total system up timeW    crhcnt   ;system crash countH    tmpin    ;current temp into systemH    tmpout   ;current temp out of systemB(??)res      ;reservedH    state    ;current system stateH    curpcs   ;pointer into GDT for current         ;processH    pcsobj   ;current process obj. seg. selector                                  SystemH    bufptr   ;pointer to start of memory                                  Vari-         :buffers                 ablesH    lruptr   ;pointer to sector buffer lru chainW    pcsptr   ;branch pointer for process switchH    pcsctr   ;id for next process to be createdB(??)res      ;reservedSYSDEV(5)                 SYSDEV(5)NAMEsysdev -- device index definition______________________________________
```

DESCRIPTION
Each device supported by the operating system has a set of device drivers to support it. These routines are accessed through call tables that are indexed by the a unique number for each device (see below). The basic system supports the disks, 8530's, 8259's, 8254, array interface, printer and an sbx connected to a 3M tape drive. Additional drivers may be added if other sbx interfaces are installed in the system.

The device calls are standardized for all devices. They are: init, open, read, write, alloc, special and seek.

The device index definitions for the system are

```

______________________________________ 0:         Null device 1:         sbx0 (tape drive if any) 2:         sbx1 (interboard bus if any) 3:         sbx2 (not defined) 4:         disk controller drive 0 5:         disk controller drive 1 6:         disk controller drive 2 7:         disk controller drive 3 8:         tty0 9:         tty110:         tty211:         tty312:         tty413:         tty514:         tty615:         tty716:         terminal broadcast17:         printer18:         hypercube array______________________________________
```

## 5.4 Node Nucleus
There is a small nucleus that runs in each node of the hypercube array. The main function of the nucleus is to provide communication and synchronization facilities. However, there is also a simple debugger and a program loader and scheduler.

### 5.4.1 Communication and Synchronization
The model of computation assumed by the system is that the user will explicitly separate a program and data into parts that run on separate processors. It is also assumed that any synchronization that is necessary will be accomplished by waiting on communication. Therefore, the key functions are the communication routines. Communication at the user level is done with two simple system calls: "send" and "receive". They both have three arguments: a set of nodes, a message and a message length. A message is a string of bytes and the set of nodes is the destination or origination of the message. Both routines are "blocking" functions and do not return until the message is sent or received. To avoid waiting when synchronization is unnecessary, there is a "test for message" function which returns immediately with a flag indicating whether there is a message waiting for reception. There are also timed versions of send and receive that have an additional "time-out" parameter. They return after the message is received (or sent) or time-out is reached, whichever comes first. They return a flag indicating which condition caused the return.

The underlying system level handshaking and buffer management breaks a message up into small blocks and sends (receives) one block at a time. For messages that must be routed through more than one node, this is much more efficient than trying to handle the whole message at once. Also, it prevents a "waiting for buffer" type of deadlock.

### 5.4.2 Debugging
There is a simple debugger that runs in each node. In response to messages from the Peripheral Controller that is managing the subcube, a node can set breakpoints and read and set memory and registers.

### 5.4.3 Program Loading and Scheduling
The node nucleus has system calls that, in response to messages from the Peripheral Controller currently managing the node, allows a node to load a program and its data and schedule it for execution.

### 5.4.4 Nucleus System Calls
This section specifies the calls that a program running in a node can make on the nucleus. It also shows how a program running in the Peripheral Controller that is managing a node can, through sending and receiving messages, access some of the system calls. The list of system calls includes ##STR47##

# 6 SYSTEM MANAGEMENT
In this section a method for initializing the system is presented, especially how to propagate the initializing software through the array. There is more than one acceptable algorithm. The one we present here is a very simple one with high efficiency. The algorithm is based on a tree structure. The diagram below shows the initialization responsibility for each processor assuming there are 16 processors. The binary numbers are the processor ID's and the decimal numbers represent the stage (in time) of the initialization. ##STR48##

The assembly language code that implements this algorithm is:

```

__________________________________________________________________________ MOVW ID,R1 ;ID is memory location containing the            ;processor ID LDPR R1,IDREG            ;the ID is loaded into the ID processor            ;register FFO  R1,R2 ;R2 = # of trailing zeros in ID SUBB #1,R2 ; JL   END   ;no trailing zeros → this processor is            ;a leaf on the graphLOOP: MOVW #1,R3 ;compute ID of neighbor by complementing SFTW R2,R3 ;one of the trailing zeros MOVW R1,R4 ; XORW R3,R4 ;R4 = new ID{send message length to port #(R2)}{receive status; use timeout}a.      dead (timed out)b.      failed self testc.      parity errord.      alive and well{if alive MOVW R4,ID;put new ID in memory}{send copy of code and new ID to R2}REPC       R2    ;JMP        LOOP  ;END:{look for responses and EROF}__________________________________________________________________________
```

