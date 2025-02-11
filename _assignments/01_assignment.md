---
type: assignment
date: 2024-01-11T8:00:00+4:30
enable: yes
title: 'Lab #1 - Booting a PC'
due_event: 
    type: due
    enable: yes
    date: 2024-01-25T23:59:00+3:30
    description: 'Lab #1 due'
---

# Introduction

> #### Useful References:
> - [at&t_asm](https://www.cs.yale.edu/flint/cs421/papers/x86-asm/asm.html)
> - [GDB tutorial1](http://www.cs.toronto.edu/~krueger/csc209h/tut/gdb_tutorial.html)
> - [GDB tutorial2](http://beej.us/guide/bggdb/)
> - [GDB chect-sheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf)

This lab is split into three parts. The first part concentrates on
getting familiarized with x86 assembly language, the QEMU x86 emulator,
and the PC's power-on bootstrap procedure. The second part examines the
boot loader for our kernel, which resides in the ``boot/``
directory of the ``jos/`` tree. Finally, the third part delves into the
initial template for our kernel itself, named JOS, which resides
in the ``kernel/`` directory.

# Software Setup

The files you will need for this and subsequent lab assignments in this
course are distributed using the [Git](http://www.git-scm.com/)
version control system. To learn more about Git, take a look at the [Git user's manual](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html),
or, if you are already familiar with other version control systems, you
may find this [CS-oriented overview of Git](http://eagain.net/articles/git-for-computer-scientists/) useful.

The URL for the lab Git repository is `https://github.com/EE469/jos.git`

Clone the repository on to the local machine:
```
$ git clone https://github.com/EE469/jos.git
Cloning into jos...
$ cd jos
```

Git allows you to keep track of the changes you make to the code. 
For example, if you are finished with one of the exercises, and want to checkpoint your progress, 
you can commit your changes by running:

```
$ git commit -am 'my solution for lab1 exercise 9'
Created commit 60d2135: my solution for lab1 exercise 9
1 files changed, 1 insertions(+), 0 deletions(-)
$
```

This *commit* will store your progress locally.

You can keep track of your changes by using the git diff command. Running git diff will display the changes to your code 
since your last commit, and git diff `origin/lab1` will display the changes relative to the initial code supplied for this lab. 
Here, `origin/lab1` is the name of the git branch with the initial code you downloaded from our server for this assignment.

We have set up the appropriate compilers and simulators for you on `eceprog.ecn.purdue.edu`. 


If you are working on a non-ecngrid machine, you'll need to install qemu and possibly gcc following the directions 
on the [tools](../assignments/00_assignment) page. We've made several useful debugging changes to qemu and some of the 
later labs depend on these patches, so you must build your own. 

# Hand-In Procedure
You will turn in your assignments through the BrightSpace (one for a group). 

The lab code comes with GNU Make rules to make submission easier. 
After committing your final changes to the lab, type make handin to submit your lab.

```
$ git commit -am "ready to submit my lab"
[lab1 c2e3c8b] ready to submit my lab
2 files changed, 18 insertions(+), 2 deletions(-)
$ make handin
tar: Removing leading `/' from member names
tar: Removing leading `/' from hard link targets
Please submit lab1-handin.tar.gz to Brightspace for your group.
```

Now upload your `lab1-handin.tar.gz` as the submission for your group.

If use `make handin` and you have either uncomitted changes or untracked files, you will see output similar to the following:

```
M hello.c
?? bar.c
?? foo.pyc
Untracked files will not be handed in.  Continue? [y/N]
```

Inspect the above lines and make sure all files that your lab solution 
needs are tracked i.e. not listed in a line that begins with ??.
In the case that `make handin` does not work properly, try fixing the problem with the curl or Git commands. 

You can run `make grade` to test your solutions with the grading program.
After checking your score from running `make grade`, which will run an autograder script (the score shown is not guaranteed, 
but failure to pass make grade will not get the score; score will be finalized after checking your pushed sourcecode)

## Hint
In this lab, you may want to read the following files:
```
    boot/boot.S
    boot/main.c
    inc/elf.h
    kern/entry.S
    kern/entrypgdir.c
    kern/printf.c
    lib/printfmt.c
    kern/console.h
    kern/console.c
    inc/stab.h
    kern/kdebug.h
    kern/kdebug.c
    kern/monitor.h
    kern/monitor.c
```
And, you must write your code to the following files:
```
    lib/printfmt.c;     vprintfmt() (for %o, the octet part)
    kern/kdebug.c;      debuginfo_eip()
    kern/monitor.c;     commands[] and mon_backtrace()
```

# Part 1: PC Bootstrap

The purpose of the first exercise is to introduce you to x86 assembly
language and the PC bootstrap process, and to get you started with QEMU
and QEMU/GDB debugging. You will not have to write any code for this
part of the lab, but you should go through it anyway for your own
understanding and be prepared to answer the questions posed below.

## Getting Started with x86 assembly

If you are not already familiar with x86 assembly language, you will
quickly become familiar with it during this course! 
The [PC Assembly Language Book](/ee469/static_files/read/pcasm-book.pdf) is an excellent place
to start. Hopefully, the book contains mixture of new and old material
for you.

*Warning:* Unfortunately the examples in the book are written for the
NASM assembler, whereas we will be using the GNU assembler. NASM uses
the so-called *Intel* syntax while GNU uses the *AT&T* syntax. While
semantically equivalent, an assembly file will differ quite a lot, at
least superficially, depending on which syntax is used. Luckily the
conversion between the two is pretty simple, and is covered in
[Brennan's Guide to Inline Assembly](http://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html).

## <span style="color:blue">Exercise 1</span>
> Familiarize yourself with the assembly language materials
available on [the EE469 reference page](/ee469/materials/). You
don't have to read them now, but you'll almost certainly want to refer
to some of this material when reading and writing x86 assembly.

> We do recommend reading the section "The Syntax" in
[Brennan's Guide to Inline Assembly](http://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html).
It gives a good (and quite brief) description of the AT&T assembly
syntax we'll be using with the GNU assembler in JOS.

Certainly the definitive reference for x86 assembly language programming
is Intel's instruction set architecture reference, which you can find on
[the EE469 reference page](/ee469/materials/) in two flavors: an
HTML edition of the old [80386 Programmer's Reference Manual](http://www.logix.cz/michal/doc/i386/), which is much shorter and
easier to navigate than more recent manuals but describes all of the x86
processor features that we will make use of in ee469; and the full,
latest and greatest [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
from Intel, covering all the features of the most recent processors that we won't need in class but you may be interested in learning about. An
equivalent (and often friendlier) set of manuals is [available from AMD](https://www.amd.com/system/files/TechDocs/24592.pdf).
Save the Intel/AMD architecture manuals for later or use them for reference when you want to look up the 
definitive explanation of a particular processor feature or instruction.


## Simulating the x86

Instead of developing the operating system on a real, physical personal
computer (PC), we use a program that faithfully emulates a complete PC:
the code you write for the emulator will boot on a real PC and you
will be asked to test in the recitation.
Using an emulator simplifies debugging; you can, for example, set break points
inside of the emulated x86, which is difficult to do with the silicon
version of an x86.

In EE469, we will use the [QEMU Emulator](http://www.qemu.org/),
a modern and relatively fast emulator.
While QEMU's built-in monitor provides only limited debugging support,
QEMU can act as a remote debugging target for the
[GNU debugger](http://www.gnu.org/software/gdb/) (GDB),
which we'll use in this lab to step through the early boot process.

To get started, clone the Lab 1 repo into your own directory
as described above in "Software Setup", then type make
(or gmake on BSD systems) in the `jos/` directory to build the minimal EE469 boot loader and kernel you will start with.
(It's a little generous to call the code we're running here a "kernel," but we'll flesh it out throughout the semester.)

```
$ cd jos
$ make
+ as kern/entry.S
+ cc kern/init.c
+ cc kern/console.c
+ cc kern/monitor.c
+ cc kern/printf.c
+ cc lib/printfmt.c
+ cc lib/readline.c
+ cc lib/string.c
+ ld obj/kern/kernel
+ as boot/boot.S
+ cc -Os boot/main.c
+ ld boot/boot
boot block is 414 bytes (max 510)
+ mk obj/kern/kernel.img
```

Now you're ready to run QEMU, supplying the file `obj/kern/kernel.img`, created above, as the contents of the emulated
PC's "virtual hard disk." This hard disk image contains both our boot  loader (`obj/boot/boot`) and our kernel (`obj/kernel`).

```
$ make qemu-nox
```

This executes QEMU with the options required to set the hard disk and
direct serial port output to the terminal. Some text should appear in
the QEMU window:

```
$ make qemu-nox
***
*** Use Ctrl-a x to exit qemu
***
qemu-system-i386 -nographic -drive file=obj/kern/kernel.img,index=0,media=disk,format=raw -serial mon:stdio -gdb tcp::26003 -D qemu.log
444544 decimal is XXX octal!
entering test_backtrace 5
entering test_backtrace 4
entering test_backtrace 3
entering test_backtrace 2
entering test_backtrace 1
entering test_backtrace 0
leaving test_backtrace 0
leaving test_backtrace 1
leaving test_backtrace 2
leaving test_backtrace 3
leaving test_backtrace 4
leaving test_backtrace 5
Welcome to the JOS kernel monitor!
Type 'help' for a list of commands.
K>
```

Everything after "Booting from Hard Disk..." was printed by our
skeletal JOS kernel; the `K>` is the prompt printed by the small
*monitor*, or interactive control program, that we've included in the
kernel. These lines printed by the kernel will also appear in the
regular shell window from which you ran QEMU. This is because for
testing and lab grading purposes we have set up the JOS kernel to write
its console output not only to the virtual VGA display (as seen in the
QEMU window), but also to the simulated PC's virtual serial port, which
QEMU in turn outputs to its own standard output. Likewise, the JOS
kernel will take input from both the keyboard and the serial port, so
you can give it commands in either the VGA display window or the
terminal running QEMU. Alternatively, you can use the serial console
without the virtual VGA by running make qemu-nox. This may be convenient
if you are SSH'd into a remote server.

There are only two commands you can give to the kernel monitor, `help`
and `kerninfo` (`info-kern` in some versions of QEMU).

```
K> help
help - display this list of commands
kerninfo - display information about the kernel
K> kerninfo
Special kernel symbols:
  entry  f010000c (virt)  0010000c (phys)
  etext  f0101a75 (virt)  00101a75 (phys)
  edata  f0112300 (virt)  00112300 (phys)
  end    f0112960 (virt)  00112960 (phys)
Kernel executable memory footprint: 75KB
K>
```

The `help` command is obvious, and we will shortly discuss the meaning
of what the `kerninfo` command prints. Although simple, it's important
to note that this kernel monitor is running "directly" on the "raw
(virtual) hardware" of the simulated PC. This means that you should be
able to copy the contents of `obj/kern/kernel.img` onto the first few
sectors of a *real* hard disk, insert that hard disk into a real PC,
turn it on, and see exactly the same thing on the PC's real screen as
you did above in the QEMU window. (We don't recommend you do this on a
real machine with useful information on its hard disk, though, because
copying `kernel.img` onto the beginning of its hard disk will trash
the master boot record and the beginning of the first partition,
effectively causing everything previously on the hard disk to be lost!)

## The PC's Physical Address Space

We will now dive into a bit more detail about how a PC starts up. A PC's
physical address space is hard-wired to have the following general
layout:

```

    +------------------+  <- 0xFFFFFFFF (4GB)
    |      32-bit      |
    |  memory mapped   |
    |     devices      |
    |                  |
    /\/\/\/\/\/\/\/\/\/\

    /\/\/\/\/\/\/\/\/\/\
    |                  |
    |      Unused      |
    |                  |
    +------------------+  <- depends on amount of RAM
    |                  |
    |                  |
    | Extended Memory  |
    |                  |
    |                  |
    +------------------+  <- 0x00100000 (1MB)
    |     BIOS ROM     |
    +------------------+  <- 0x000F0000 (960KB)
    |  16-bit devices, |
    |  expansion ROMs  |
    +------------------+  <- 0x000C0000 (768KB)
    |   VGA Display    |
    +------------------+  <- 0x000A0000 (640KB)
    |                  |
    |    Low Memory    |
    |                  |
    +------------------+  <- 0x00000000

```

The first PCs, which were based on the 16-bit Intel 8088 processor, were
only capable of addressing 1MB of physical memory. The physical address
space of an early PC would therefore start at ``0x00000000`` but end at
``0x000FFFFF`` instead of ``0xFFFFFFFF``. The 640KB area marked "Low
Memory" was the *only* random-access memory (RAM) that an early PC could
use; in fact the very earliest PCs only could be configured with 16KB,
32KB, or 64KB of RAM!

The 384KB area from ``0x000A0000`` through ``0x000FFFFF`` was reserved
by the hardware for special uses such as video display buffers and firmware
held in non-volatile memory. The most important part of this reserved
area is the Basic Input/Output System (BIOS), which occupies the 64KB
region from ``0x000F0000`` through ``0x000FFFFF``. In early PCs the
BIOS was held in true read-only memory (ROM), but current PCs store the
BIOS in updateable flash memory. The BIOS is responsible for performing
basic system initialization such as activating the video card and checking
the amount of memory installed. After performing this initialization, the
BIOS loads the operating system from some appropriate location such as
floppy disk, hard disk, CD-ROM, or the network, and passes control of
the machine to the operating system.

When Intel finally "broke the one megabyte barrier" with the 80286 and
80386 processors, which supported 16MB and 4GB physical address spaces
respectively, the PC architects nevertheless preserved the original
layout for the low 1MB of physical address space in order to ensure
backward compatibility with existing software. Modern PCs therefore have
a "hole" in physical memory from ``0x000A0000`` to ``0x00100000``,
dividing RAM into "low" or "conventional memory" (the first 640KB)
and "extended memory" (everything else). In addition, some space at
the very top of the PC's 32-bit physical address space, above all
physical RAM, is now commonly reserved by the BIOS for use by 32-bit
PCI devices.

Recent x86 processors can support *more* than 4GB of physical RAM, so
RAM can extend further above ``0xFFFFFFFF``. In this case the BIOS must
arrange to leave a *second* hole in the system's RAM at the top of the
32-bit addressable region, to leave room for these 32-bit devices to be
mapped. Because of design limitations JOS will use only the first 256MB
of a PC's physical memory anyway, so for now we will pretend that all
PCs have "only" a 32-bit physical address space. But dealing with
complicated physical address spaces and other aspects of hardware
organization that evolved over many years is one of the important
practical challenges of OS development.

## The ROM BIOS


In this portion of the lab, you'll use QEMU's debugging facilities to
investigate how an IA-32 compatible computer boots.

Open two terminal windows. In one, enter ``make qemu-nox-gdb``.
This starts up QEMU, but QEMU stops just before the
processor executes the first instruction and waits for a debugging
connection from GDB. In the second terminal, from the same directory,
run ``gdb``. You should see something like this,

If you are using `eceprg` then one of the best ways is to use `tmux`.

### Debugging with <code>tmux</code>
For debugging we require two `tmux` sessions.
Now let's open two terminals and `ssh` into `eceprog` in both of these terminals.
We will use one to launch the `qemu` in debug mode ( debug-qemu session) and the other 
one to attach the `gdb` to the `qemu` process ( debug-gdb session).

 1. debug-qemu session on terminal #1
```
$ cd $JOS_PATH # set JOS_PATH to JOS source code path
$ tmux new -s debug-qemu
$ make qemu-nox-gdb
```
 2. debug-gdb session on terminal #2
 
```
$ cd $JOS_PATH # set JOS_PATH to JOS source code path
$ tmux new -s debug-gdb
$ gdb
+ target remote localhost:26003
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
warning: A handler for the OS ABI "GNU/Linux" is not built into this configuration
of GDB.  Attempting to continue with the default i8086 settings.

The target architecture is assumed to be i8086
[f000:fff0]    0xffff0: ljmp   $0xf000,$0xe05b
0x0000fff0 in ?? ()
+ symbol-file obj/kern/kernel
gdb$
```

We provided a ``.gdbinit`` file that set up GDB to debug the 16-bit code
used during early boot and directed it to attach to the listening QEMU.

(If it doesn't work, you may have to add an ``add-auto-load-safe-path $JOS_PATH/.gdbinit``
in your ``.gdbinit`` file in the home directory. This is to convince ``gdb`` to
process the ``.gdbinit`` we have provided. Substitute ``$JOS_PATH`` with the JOS source code path
from where you are running the ``gdb``. ``gdb`` will tell you if you have to add ``add-auto-load-safe-path``.)

The following line:

```
[f000:fff0] 0xffff0:    ljmp   $0xf000,$0xe05b
```

is GDB's disassembly of the first instruction to be executed. From this
output you can conclude a few things:

-  The IBM PC starts executing at physical address 0x000ffff0, which is
   at the very top of the 64KB area reserved for the ROM BIOS.
-  The PC starts executing with ``CS = 0xf000`` and ``IP = 0xfff0``.
-  The first instruction to be executed is a ``jmp`` instruction, which
   jumps to the segmented address ``CS = 0xf000`` and ``IP = 0xe05b``.

Why does QEMU start like this? This is how Intel designed the 8088
processor, which IBM used in their original PC. Because the BIOS in a PC
is "hard-wired" to the physical address range ``0x000f0000-0x000fffff``,
this design ensures that the BIOS always gets control of the machine
first after power-up or any system restart - which is crucial because on
power-up there *is* no other software anywhere in the machine's RAM that
the processor could execute. The QEMU emulator comes with its own BIOS,
which it places at this location in the processor's simulated physical
address space. On processor reset, the (simulated) processor enters real
mode and sets CS to ``0xf000`` and the IP to ``0xfff0``, so that
execution begins at that (CS:IP) segment address. How does the segmented
address ``0xf000:0xfff0`` turn into a physical address?

To answer that we need to know a bit about real mode addressing. In real
mode (the mode that PC starts off in), address translation works
according to the formula:

`physical address = 16 * segment + offset`

So, when the PC sets CS to ``0xf000`` and IP to ``0xfff0``, the
physical address referenced is:

```
16 * 0xf000 + 0xfff0   # in hex multiplication by 16 is
= 0xf0000 + 0xfff0     # easy--just append a 0.
= 0xffff0
```

``0xffff0`` is 16 bytes before the end of the BIOS (0x100000).
Therefore we shouldn't be surprised that the first thing that the BIOS
does is ``jmp`` backwards to an earlier location in the BIOS; after all
how much could it accomplish in just 16 bytes?

## <span style="color:blue">Exercise 2</span>
> Use GDB's ``si`` (Step Instruction) command to trace into the
ROM BIOS for a few more instructions, and try to guess what it might be
doing. You might want to look at
[Phil Storrs](http://philipstorr.id.au/pcbook/) and his
[I/O Ports Description](http://philipstorr.id.au/pcbook/book2/book2.htm),
as well as other materials on the
[EE469 reference materials page](/ee469/materials/).

> No need to figure out all the details - just the general idea of what the BIOS is doing first.

When the BIOS runs, it sets up an interrupt descriptor table and
initializes various devices such as the VGA display. This is where the
[Starting SeaBIOS](http://www.seabios.org/) message you see in the QEMU window comes from.

After initializing the PCI bus and all the important devices the BIOS
knows about, it searches for a bootable device such as a floppy, hard
drive, or CD-ROM. Eventually, when it finds a bootable disk, the BIOS
reads the *boot loader* from the disk and transfers control to it.

# Part 2: The Boot Loader

Floppy and hard disks for PCs are divided into 512 byte regions called
*sectors*. A sector is the disk's minimum transfer granularity: each
read or write operation must be one or more sectors in size and aligned
on a sector boundary. If the disk is bootable, the first sector is
called the *boot sector*, since this is where the boot loader code
resides. When the BIOS finds a bootable floppy or hard disk, it loads
the 512-byte boot sector into memory at physical addresses ``0x7c00``
through ``0x7dff``, and then uses a ``jmp`` instruction to set the CS:IP to
``0000:7c00``, passing control to the boot loader. Like the BIOS load
address, these addresses are fairly arbitrary - but they are fixed and
standardized for PCs.
If you are curious, see how SeaBIOS actually
handles this booting procedure
[seabios/src/boot.c](https://github.com/coreboot/seabios/blob/master/src/boot.c#L628).

The ability to boot from a CD-ROM came much later during the evolution
of the PC, and as a result the PC architects took the opportunity to
rethink the boot process slightly. As a result, the way a modern BIOS
boots from a CD-ROM is a bit more complicated (and more powerful).
CD-ROMs use a sector size of 2048 bytes instead of 512, and the BIOS can
load a much larger boot image from the disk into memory (not just one
sector) before transferring control to it. For more information, see the
[El Torito" Bootable CD-ROM Format Specification](https://pdos.csail.mit.edu/6.828/2014/readings/boot-cdrom.pdf).

For however, we will use the conventional hard drive boot
mechanism, which means that our boot loader must fit into a measly 512
bytes. The boot loader consists of one assembly language source file,
``boot/boot.S``, and one C source file, ``boot/main.c``. Look through
these source files carefully and make sure you understand what's going
on. The boot loader must perform two main functions:

1. First, the boot loader switches the processor from real mode to
   *32-bit protected mode*, because it is only in this mode that
   software can access all the memory above 1MB in the processor's
   physical address space. Protected mode is described briefly in
   sections 1.2.7 and 1.2.8 of [PC Assembly Language](/ee469/static_files/read/pcasm-book.pdf), and in great detail in
   the Intel architecture manuals. At this point you only have to
   understand that translation of segmented addresses (segment:offset
   pairs) into physical addresses happens differently in protected mode,
   and that after the transition offsets are 32 bits instead of 16 bits.

2. Second, the boot loader reads the kernel from the hard disk by
   directly accessing the IDE disk device registers via the x86's
   special I/O instructions. If you would like to understand better what
   the particular I/O instructions here mean, check out the
   "IDE hard drive controller" section on [the reference page](/ee469/materials/).
   You will not need to learn much about
   programming specific devices in this class: writing device drivers is
   in practice a very important part of OS development, but from a
   conceptual or architectural viewpoint it is also one of the least
   interesting.

After you understand the boot loader source code, look at the file
``obj/boot/boot.asm``. This file is a disassembly of the boot loader
that our ``Makefile`` creates *after* compiling the boot loader. This
disassembly file makes it easy to see exactly where in physical memory
all of the boot loader's code resides, and makes it easier to track
what's happening while stepping through the boot loader in GDB.
Likewise, ``obj/kern/kernel.asm`` contains a disassembly of the JOS
kernel, which can often be useful for debugging.

You can set address breakpoints in GDB with the `b` command. For
example, `b *0x7c00` sets a breakpoint at address 0x7C00. Once at a
breakpoint, you can continue execution using the `c` and `si`
commands:
`c` causes QEMU to continue execution until the next breakpoint (or until
you press Ctrl-C in GDB), and `si N` steps through the instructions
*N* at a time.

To examine instructions in memory (besides the immediate next one to be
executed, which GDB prints automatically), you use the `x/i` command. This
command has the syntax `x/Ni ADDR`, where *N* is the number of
consecutive instructions to disassemble and *ADDR* is the memory address
at which to start disassembling.

## <span style="color:blue">Exercise 3</span>
> Take a look at the [tools](../assignments/00_assignment),
    especially the section on GDB commands.
    Even if you're familiar with GDB, this includes some esoteric GDB
    commands that are useful for OS work.

> Set a breakpoint at address 0x7c00, which is where the boot sector will
    be loaded. Continue execution until that breakpoint. Trace through the
    code in ``boot/boot.S``, using the source code and the disassembly file
    ``obj/boot/boot.asm`` to keep track of where you are. Also use the
    `x/i` command in GDB to disassemble sequences of instructions in the
    boot loader, and compare the original boot loader source code with both
    the disassembly in ``obj/boot/boot.asm`` and GDB.

> Trace into ``bootmain()`` in ``boot/main.c``, and then into
    ``readsect()``. Identify the exact assembly instructions that correspond
    to each of the statements in ``readsect()``. Trace through the rest of
    ``readsect()`` and back out into ``bootmain()``, and identify the begin
    and end of the ``for`` loop that reads the remaining sectors of the
    kernel from the disk. Find out what code will run when the loop is
    finished, set a breakpoint there, and continue to that breakpoint. Then
    step through the remainder of the boot loader.

Be able to answer the following questions:

- At what point does the processor start executing 32-bit code? What
   exactly causes the switch from 16- to 32-bit mode?
- What is the *last* instruction of the boot loader executed, and what
   is the *first* instruction of the kernel it just loaded?
- *Where* is the first instruction of the kernel?
- How does the boot loader decide how many sectors it must read in
   order to fetch the entire kernel from disk? Where does it find this
   information?

## Loading the Kernel

We will now look in further detail at the C language portion of the boot
loader, in ``boot/main.c``. But before doing so, this is a good time to
stop and review some of the basics of C programming.

## <span style="color:blue">Exercise 4</span>
> Read about programming with pointers in C.

> Download the code for [pointers.c](/ee469/static_files/code/pointers.c) run it, 
and make sure you understand where all of the printed values come from. In particular, make sure you
understand where the pointer addresses in lines 1 and 6 come from, how  all the values in 
lines 2 through 4 get there, and why the values printed in line 5 are seemingly corrupted.

> There are other references on pointers in C 
  (e.g., [A tutorial by Ted Jensen](/ee469/static_files/read/pointers.pdf) that cites K&R heavily), though not as strongly recommended.

__Unless you are already thoroughly versed in C, do not skip or
    even skim this reading exercise. If you do not really understand
    pointers in C, you will suffer untold pain and misery in subsequent
    labs, and then eventually come to understand them the hard way. Trust
    us; you don't want to find out what "the hard way" is.__

To make sense out of ``boot/main.c`` you'll need to know what an ELF
binary is. When you compile and link a C program such as the JOS kernel,
the compiler transforms each C source (.c) file into an *object*
(.o) file containing assembly language instructions encoded in
the binary format expected by the hardware. The linker then combines all
of the compiled object files into a single *binary image* such as
``obj/kern/kernel``, which in this case is a binary in the ELF format,
which stands for "Executable and Linkable Format".

Full information about this format is available in
[the ELF specification](/ee469/static_files/read/elf.pdf) on [the reference page](/ee469/materials/),
but you will not need to delve very
deeply into the details of this format in this class. Although as a
whole the format is quite powerful and complex, most of the complex
parts are for supporting dynamic loading of shared libraries, which we
will not do in this class. The
[Wikipedia page](http://en.wikipedia.org/wiki/Executable_and_Linkable_Format) has a short description.

For purposes of ee469, you can consider an ELF executable to be a header
with loading information, followed by several *program sections*, each
of which is a contiguous chunk of code or data intended to be loaded
into memory at a specified address. The boot loader does not modify the
code or data; it loads it into memory and starts executing it.

An ELF binary starts with a fixed-length *ELF header*, followed by a
variable-length *program header* listing each of the program sections to
be loaded. The C definitions for these ELF headers are in ``inc/elf.h``.
The program sections we're interested in are:

-  ``.text``: The program's executable instructions.
-  ``.rodata``: Read-only data, such as ASCII string constants produced
   by the C compiler. (We will not bother setting up the hardware to
   prohibit writing, however.)
-  ``.data``: The data section holds the program's initialized data,
   such as global variables declared with initializers like
   ``int x = 5;``.

When the linker computes the memory layout of a program, it reserves
space for *uninitialized* global variables, such as ``int x;``, in a
section called ``.bss`` that immediately follows ``.data`` in memory. C
requires that "uninitialized" global variables start with a value of
zero. Thus there is no need to store contents for ``.bss`` in the ELF
binary; instead, the linker records just the address and size of the
``.bss`` section. The loader or the program itself must arrange to zero
the ``.bss`` section.

Examine the full list of the names, sizes, and link addresses of all the
sections in the kernel executable by typing:

```
$ objdump -h obj/kern/kernel

obj/kern/kernel:     file format elf32-i386

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .text         00001861  f0100000  00100000  00001000  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .rodata       00000714  f0101880  00101880  00002880  2**5
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  ...
  4 .data         0000a300  f0108000  00108000  00009000  2**12
                  CONTENTS, ALLOC, LOAD, DATA
  5 .bss          00000644  f0112300  00112300  00013300  2**5
                  ALLOC
  ...
```

You will see many more sections than the ones we listed above, but the
others are not important for our purposes. Most of the others are to
hold debugging information, which is typically included in the program's
executable file but not loaded into memory by the program loader.

Take particular note of the "VMA" (or *link address*) and the "LMA" (or
*load address*) of the ``.text`` section. The load address of a section
is the memory address at which that section should be loaded into
memory.

The link address of a section is the memory address from which the
section expects to execute. The linker encodes the link address in the
binary in various ways, such as when the code needs the address of a
global variable, with the result that a binary usually won't work if it
is executing from an address that it is not linked for. (It is possible
to generate *position-independent* code that does not contain any such
absolute addresses. This is used extensively by modern shared libraries,
but it has performance and complexity costs, so we won't be using it in ee469.)

Typically, the link and load addresses are the same. For example, look
at the ``.text`` section of the boot loader:

```
$ objdump -h obj/boot/boot.out

obj/boot/boot.out:     file format elf32-i386

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .text         00000186  00007c00  00007c00  00000074  2**2
                  CONTENTS, ALLOC, LOAD, CODE
  ...
```

The boot loader uses the ELF *program headers* to decide how to load the
sections. The program headers specify which parts of the ELF object to
load into memory and the destination address each should occupy. You can
inspect the program headers by typing:

```
$ objdump -x obj/kern/kernel

obj/kern/kernel:     file format elf32-i386
obj/kern/kernel
architecture: i386, flags 0x00000112:
EXEC_P, HAS_SYMS, D_PAGED
start address 0x0010000c

Program Header:
    LOAD off    0x00001000 vaddr 0xf0100000 paddr 0x00100000 align 2**12
         filesz 0x00007108 memsz 0x00007108 flags r-x
    LOAD off    0x00009000 vaddr 0xf0108000 paddr 0x00108000 align 2**12
         filesz 0x0000a300 memsz 0x0000a944 flags rw-
   STACK off    0x00000000 vaddr 0x00000000 paddr 0x00000000 align 2**4
         filesz 0x00000000 memsz 0x00000000 flags rwx
...
```

The program headers are then listed under "Program Headers" in the
output of objdump. The areas of the ELF object that need to be loaded
into memory are those that are marked as "LOAD". Other information for
each program header is given, such as the virtual address ("vaddr"), the
physical address ("paddr"), and the size of the loaded area ("memsz" and
"filesz").

Back in ``boot/main.c``, the ``ph->p_pa`` field of each program header
contains the segment's destination physical address (in this case, it
really is a physical address, though the ELF specification is vague on
the actual meaning of this field).

The BIOS loads the boot sector into memory starting at address 0x7c00,
so this is the boot sector's load address. This is also where the boot
sector executes from, so this is also its link address. We set the link
address by passing ``-Ttext 0x7C00`` to the linker in ``boot/Makefrag``,
so the linker will produce the correct memory addresses in the generated
code.

## <span style="color:blue">Exercise 5</span>
> Trace through the first few instructions of the boot loader
again and identify the first instruction that would "break" or otherwise
do the wrong thing if you were to get the boot loader's link address
wrong.

> Then change the link address in ``boot/Makefrag`` to something
wrong, run ``make clean``, recompile the lab with ``make``, and trace into the
boot loader again to see what happens. Don't forget to change the link
address back and ``make clean`` again afterward!

Look back at the load and link addresses for the kernel. Unlike the boot
loader, these two addresses aren't the same: the kernel is telling the
boot loader to load it into memory at a low address (1 megabyte), but it
expects to execute from a high address. We'll dig in to how we make this
work in the next section.

Besides the section information, there is one more field in the ELF
header that is important to us, named ``e_entry``. This field holds the
link address of the *entry point* in the program: the memory address in
the program's text section at which the program should begin executing.
You can see the entry point:

```
$ readelf -h obj/kern/kernel
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Intel 80386
  Version:                           0x1
  Entry point address:               0x10000c
  Start of program headers:          52 (bytes into file)
  Start of section headers:          81124 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         3
  Size of section headers:           40 (bytes)
  Number of section headers:         11
  Section header string table index: 8
```

You should now be able to understand the minimal ELF loader in
``boot/main.c``. It reads each section of the kernel from disk into
memory at the section's load address and then jumps to the kernel's
entry point.

## <span style="color:blue">Exercise 6</span>
> We can examine memory using GDB's x command. The
[GDB manual](https://sourceware.org/gdb/onlinedocs/gdb/Memory.html)
has full details, but for now, it is enough to know that the command
`x/Nx ADDR` prints N words of memory at ADDR (e.g.,
`x/10x 0x10000c` to print 10 words from 0x10000c).

> **Warning**: The size of a word is not a universal standard.
In GNU assembly, a word is two
bytes (the 'w' in xorw, which stands for word, means 2 bytes) and
in GDB, a word means 4 bytes (and a giant for 8 bytes).

> Reset the machine (exit QEMU/GDB and start them again). Examine the 8
words of memory at 0x00100000 at the point the BIOS enters the boot
loader, and then again at the point the boot loader enters the kernel.
Why are they different? What is there at the second breakpoint? (You do
not really need to use QEMU to answer this question. Just think.)


# Part 3: The Kernel

We will now start to examine the minimal JOS kernel in a bit more
detail. (And you will finally get to write some code!). Like the boot
loader, the kernel begins with some assembly language code that sets
things up so that C language code can execute properly.

## Using virtual memory to work around position dependence

When you inspected the boot loader's link and load addresses above, they
matched perfectly, but there was a (rather large) disparity between the
*kernel's* link address (as printed by ``objdump``) and its load address. Go
back and check both and make sure you can see what we're talking about.
(Linking the kernel is more complicated than the boot loader, so the
link and load addresses are at the top of ``kern/kernel.ld``.)

Operating system kernels often like to be linked and run at very high
*virtual address*, such as ``0xf0100000``, in order to leave the lower part
of the processor's virtual address space for user programs to use. The
reason for this arrangement will become clearer in the next lab.

Many machines don't have any physical memory at address ``0xf0100000``, so
we can't count on being able to store the kernel there. Instead, we will
use the processor's memory management hardware to map virtual address
``0xf0100000`` (the link address at which the kernel code *expects* to run)
to physical address ``0x00100000`` (where the boot loader loaded the kernel
into physical memory). This way, although the kernel's virtual address
is high enough to leave plenty of address space for user processes, it
will be loaded in physical memory at the 1MB point in the PC's RAM, just
above the BIOS ROM. This approach requires that the PC have at least a
few megabytes of physical memory (so that physical address ``0x00100000``
works), but this is likely to be true of any PC built after about 1990.

In fact, in the next lab, we will map the *entire* bottom 256MB of the
PC's physical address space, from physical addresses ``0x00000000`` through
``0x0fffffff``, to virtual addresses ``0xf0000000`` through ``0xffffffff``
respectively. You should now see why JOS can only use the first 256MB of
physical memory.

For now, we'll just map the first 4MB of physical memory, which will be
enough to get us up and running. We do this using the hand-written,
statically-initialized page directory and page table in
``kern/entrypgdir.c``. For now, you don't have to understand the details
of how this works, just the effect that it accomplishes. Up until
``kern/entry.S`` sets the ``CR0_PG`` flag, memory references are treated
as physical addresses (strictly speaking, they're linear addresses, but
``boot/boot.S`` set up an identity mapping from linear addresses to physical
addresses and we're never going to change that). Once ``CR0_PG`` is set,
memory references are virtual addresses that get translated by the
virtual memory hardware to physical addresses. ``entry_pgdir``
translates virtual addresses in the range ``0xf0000000`` through ``0xf0400000``
to physical addresses ``0x00000000`` through ``0x00400000``, as well as virtual
addresses ``0x00000000`` through ``0x00400000`` to physical addresses ``0x00000000``
through ``0x00400000``. Any virtual address that is not in one of these two
ranges will cause a hardware exception which, since we haven't set up
interrupt handling yet, will cause QEMU to dump the machine state and
exit.

## <span style="color:blue">Exercise 7</span>
> Use QEMU and GDB to trace into the JOS kernel and stop at
the `movl %eax, %cr0`. Examine memory at 0x00100000 and at 0xf0100000.
Now, single step over that instruction using the stepi GDB command.
Again, examine memory at 0x00100000 and at 0xf0100000. Make sure you
understand what just happened.

> What is the first instruction *after* the new mapping is established
that would fail to work properly if the mapping weren't in place?
Comment out the `movl %eax, %cr0` in ``kern/entry.S``, trace into it,
and see if you were right.


## Formatted Printing to the Console

Most people take functions like ``printf()`` for granted, sometimes even
thinking of them as "primitives" of the C language. But in an OS kernel,
we have to implement all I/O ourselves.

Read through ``kern/printf.c``, ``lib/printfmt.c``, and
``kern/console.c``, and make sure you understand their relationship. It
will become clear in later labs why ``printfmt.c`` is located in the
separate ``lib/`` directory.

## <span style="color:blue">Exercise 8</span>
> We have omitted a small fragment of code - the code
necessary to print octal numbers using patterns of the form "%o". Find
and fill in this code fragment.

Be able to answer the following questions:

1. Explain the interface between ``kern/printf.c`` and ``kern/console.c``.
   Specifically, what function does ``kern/console.c`` export? How is this
   function used by ``kern/printf.c``?
2. Explain the following from ``kern/console.c``:

```

    if (crt_pos >= CRT_SIZE) {
      int i;
      memcpy(crt_buf, crt_buf + CRT_COLS, (CRT_SIZE - CRT_COLS) * sizeof(uint16_t));
      for (i = CRT_SIZE - CRT_COLS; i < CRT_SIZE; i++)
        crt_buf[i] = 0x0700 | ' ';
      crt_pos -= CRT_COLS;
    }
```

3. For the following questions you might wish to consult the notes for
   Lecture 2. These notes cover GCC's calling convention on the x86.

   Trace the execution of the following code step-by-step:

```

    int x = 1, y = 3, z = 4;
    cprintf("x %d, y %x, z %d\n", x, y, z);
```

-  In the call to ``cprintf()``, to what does ``fmt`` point? To what
   does ``ap`` point?
-  List (in order of execution) each call to ``cons_putc``,
   ``va_arg``, and ``vcprintf``. For ``cons_putc``, list its argument
   as well. For ``va_arg``, list what ``ap`` points to before and
   after the call. For ``vcprintf`` list the values of its two
   arguments.

4. Run the following code.

    ```    
    unsigned int i = 0x00646c72;
    cprintf("H%x Wo%s", 57616, &i);
    ```

    What is the output? Explain how this output is arrived at in the
    step-by-step manner of the previous exercise.
   [Here's an ASCII table](http://man7.org/linux/man-pages/man7/ascii.7.html)
    (or type `man ascii`)
    that maps bytes to characters.
    
    The output depends on that fact that the x86 is little-endian. If the
    x86 were instead big-endian what would you set ``i`` to in order to
    yield the same output? Would you need to change ``57616`` to a
    different value?
    
    [Here's a description of little- and big-endian](http://www.webopedia.com/TERM/b/big_endian.html)
    and [a more whimsical description](http://www.networksorcery.com/enp/ien/ien137.txt).

5. In the following code, what is going to be printed after ``'y='``?
   (note: the answer is not a specific value.) Why does this happen?

    ```
    cprintf("x=%d y=%d", 3);
    ```

6. Let's say that GCC changed its calling convention so that it pushed
   arguments on the stack in declaration order, so that the last
   argument is pushed last. How would you have to change ``cprintf`` or
   its interface so that it would still be possible to pass it a
   variable number of arguments?

## <span style="color:green">Challenge (Extra Credit: 1%)</span>
> Enhance the console to allow text to be printed in different
colors. The traditional way to do this is to make it interpret [ANSI escape sequences](http://rrbrandt.dee.ufcg.edu.br/en/docs/ansi/)
embedded in the text strings printed to the console, but you may use any
mechanism you like. There is plenty of information on [the EE469 reference page](/ee469/materials/) and elsewhere on the web on
programming the VGA display hardware. If you're feeling really
adventurous, you could try switching the VGA hardware into a graphics
mode and making the console draw text onto the graphical frame buffer.

> To get 1% of credit, please create a command 'show' in the monitor and
print a beautiful ASCII Art with 5 or more colors when the command is
typed on the console.

> Once you finish this, please create a file ``.lab1-extra`` at the root of
your repository directory (under ``jos/``). We will use that file as an
indicator that you finished this extra-credit and then grade your work
accordingly.

## The Stack

In the final exercise of this lab, we will explore in more detail the
way the C language uses the stack on the x86, and in the process write a
useful new kernel monitor function that prints a *backtrace* of the
stack: a list of the saved Instruction Pointer (IP) values from the
nested ``call`` instructions that led to the current point of execution.

## <span style="color:blue">Exercise 9</span>
> Determine where the kernel initializes its stack, and
exactly where in memory its stack is located. How does the kernel
reserve space for its stack? And at which "end" of this reserved area is
the stack pointer initialized to point to?

The x86 stack pointer (``esp`` register) points to the lowest location
on the stack that is currently in use. Everything *below* that location
in the region reserved for the stack is free. Pushing a value onto the
stack involves decreasing the stack pointer and then writing the value
to the place the stack pointer points to. Popping a value from the stack
involves reading the value the stack pointer points to and then
increasing the stack pointer. In 32-bit mode, the stack can only hold
32-bit values, and esp is always divisible by four. Various x86
instructions, such as ``call``, are "hard-wired" to use the stack
pointer register.

The ``ebp`` (base pointer) register, in contrast, is associated with the
stack primarily by software convention. On entry to a C function, the
function's *prologue* code normally saves the previous function's base
pointer by pushing it onto the stack, and then copies the current
``esp`` value into ``ebp`` for the duration of the function. If all the
functions in a program obey this convention, then at any given point
during the program's execution, it is possible to trace back through the
stack by following the chain of saved ``ebp`` pointers and determining
exactly what nested sequence of function calls caused this particular
point in the program to be reached. This capability can be particularly
useful, for example, when a particular function causes an ``assert``
failure or ``panic`` because bad arguments were passed to it, but you
aren't sure *who* passed the bad arguments. A stack backtrace lets you
find the offending function.

## <span style="color:blue">Exercise 10</span>
> To become familiar with the C calling conventions on the
x86, find the address of the `test_backtrace`` function in
``obj/kern/kernel.asm``, set a breakpoint there, and examine what
happens each time it gets called after the kernel starts. How many
32-bit words does each recursive nesting level of ``test_backtrace``
push on the stack, and what are those words?
**NOTE. you'll have to manually translate all
breakpoint and memory addresses to linear addresses.**

The above exercise should give you the information you need to implement
a stack backtrace function, which you should call ``mon_backtrace()``. A
prototype for this function is already waiting for you in
``kern/monitor.c``. You can do it entirely in C, but you may find the
``read_ebp()`` function in ``inc/x86.h`` useful. You'll also have to
hook this new function into the kernel monitor's command list so that it
can be invoked interactively by the user.

The backtrace function should display a listing of function call frames
in the following format:

```
Stack backtrace:
  ebp f0109e58  eip f0100a62  args 00000001 f0109e80 f0109e98 f0100ed2 00000031
  ebp f0109ed8  eip f01000d6  args 00000000 00000000 f0100058 f0109f28 00000061
  ...
```

The first line printed reflects the *currently executing* function,
namely ``mon_backtrace`` itself, the second line reflects the function
that called ``mon_backtrace``, the third line reflects the function that
called that one, and so on. You should print *all* the outstanding stack
frames. By studying ``kern/entry.S`` you'll find that there is an easy
way to tell when to stop.

Within each line, the ``ebp`` value indicates the base pointer into the
stack used by that function: i.e., the position of the stack pointer
just after the function was entered and the function prologue code set
up the base pointer. The listed ``eip`` value is the function's *return
instruction pointer*: the instruction address to which control will
return when the function returns. The return instruction pointer
typically points to the instruction after the ``call`` instruction
(why?). Finally, the five hex values listed after ``args`` are the first
five arguments to the function in question, which would have been pushed
on the stack just before the function was called. If the function was
called with fewer than five arguments, of course, then not all five of
these values will be useful. (Why can't the backtrace code detect how
many arguments there actually are? How could this limitation be fixed?)

You should use ``ebp`` to trace back. Look at the below figure:

![stacktrace](/ee469/static_files/fig/stack_on_call.png)

The behavior of stack when foo calls bar **without any arguments**:

![framewithoutparams](/ee469/static_files/fig/frame_without_param.gif)

The behavior of stack when foo calls bar **with arguments**:

![framewithparams](/ee469/static_files/fig/frame_with_param.gif)

Here are a few specific points you read about in K&R Chapter 5 that are
worth remembering for the following exercise and for future labs.

-  If ``int *p = (int*)100``, then ``(int)p + 1`` and ``(int)(p + 1)``
   are different numbers: the first is ``101`` but the second is
   ``104``. When adding an integer to a pointer, as in the second case,
   the integer is implicitly multiplied by the size of the object the
   pointer points to.
-  ``p[i]`` is defined to be the same as ``*(p+i)``, referring to the
   i'th object in the memory pointed to by p. The above rule for
   addition helps this definition work when the objects are larger than
   one byte.
-  ``&p[i]`` is the same as ``(p+i)``, yielding the address of the i'th
   object in the memory pointed to by p.

Although most C programs never need to cast between pointers and
integers, operating systems frequently do. Whenever you see an addition
involving a memory address, ask yourself whether it is an integer
addition or pointer addition and make sure the value being added is
appropriately multiplied or not.

## <span style="color:blue">Exercise 11</span>
> Implement the backtrace function as specified above. Use
the same format as in the example, since otherwise the grading script
will be confused. When you think you have it working right, run make
grade to see if its output conforms to what our grading script expects,
and fix it if it doesn't. *After* you have handed in your Lab 1 code,
you are welcome to change the output format of the backtrace function
any way you like.

If you use ``read_ebp()``, note that GCC may generate "optimized" code
that calls ``read_ebp()`` *before* ``mon_backtrace()``'s function
prologue, which results in an incomplete stack trace (the stack frame of
the most recent function call is missing). While we have tried to
disable optimizations that cause this reordering, you may want to
examine the assembly of ``mon_backtrace()`` and make sure the call to
``read_ebp()`` is happening after the function prologue.

At this point, your backtrace function should give you the addresses of
the function callers on the stack that lead to ``mon_backtrace()`` being
executed. However, in practice you often want to know the function names
corresponding to those addresses. For instance, you may want to know
which functions could contain a bug that's causing your kernel to crash.

To help you implement this functionality, we have provided the function
``debuginfo_eip()``, which looks up ``eip`` in the symbol table and
returns the debugging information for that address. This function is
defined in ``kern/kdebug.c``.

## <span style="color:blue">Exercise 12</span>
> Modify your stack backtrace function to display, for each
``eip``, the function name, source file name, and line number
corresponding to that ``eip``.

In ``debuginfo_eip``, where do ``__STAB_*`` come from? This question has
a long answer; to help you to discover the answer, here are some things
you might want to do:

-  look in the file ``kern/kernel.ld`` for ``__STAB_*``
-  run ``objdump -h obj/kern/kernel``
-  run ``objdump -G obj/kern/kernel``
-  run ``gcc -pipe -nostdinc -O2 -fno-builtin -I. -MD -Wall
   -Wno-format -DJOS\_KERNEL -gstabs -c -S kern/init.c``, and look at
   ``kern/init.s``.
-  see if the bootloader loads the symbol table in memory as part of
   loading the kernel binary

Complete the implementation of ``debuginfo_eip`` by inserting the call
to ``stab_binsearch`` to find the line number for an address.

Add a ``backtrace`` command to the kernel monitor, and extend your
implementation of ``mon_backtrace`` to call ``debuginfo_eip`` and print
a line for each stack frame of the form:

```
K> backtrace
Stack backtrace:
  ebp f010ff78  eip f01008ae  args 00000001 f010ff8c 00000000 f0110580 00000000
         kern/monitor.c:143: monitor+106
  ebp f010ffd8  eip f0100193  args 00000000 00001aac 00000660 00000000 00000000
         kern/init.c:49: i386_init+59
  ebp f010fff8  eip f010003d  args 00000000 00000000 0000ffff 10cf9a00 0000ffff
         kern/entry.S:70: <unknown>+0
K>
```

Each line gives the file name and line within that file of the stack
frame's ``eip``, followed by the name of the function and the offset of
the ``eip`` from the first instruction of the function (e.g.,
``monitor+106`` means the return ``eip`` is 106 bytes past the beginning
of ``monitor``).

Be sure to print the file and function names on a separate line, to
avoid confusing the grading script.

**Tip: ``printf`` format strings provide an easy, albeit obscure, way to print
non-null-terminated strings like those in STABS tables.
``printf("%.*s", length, string)`` prints at most ``length`` characters
of ``string``. Take a look at the printf man page to find out why this works.**

You may find that some functions are missing from the backtrace. For
example, you will probably see a call to ``monitor()`` but not to
``runcmd()``. This is because the compiler in-lines some function calls.
Other optimizations may cause you to see unexpected line numbers. If you
get rid of the ``-O2`` from ``Makefile``, the backtraces may make
more sense (but your kernel will run more slowly).

**This completes the lab.** In the ``jos`` directory, commit your
changes with ``git commit``, and ``make handin`` to get the instructions to submit your code. 
Please do not forget to create and include the file ``.lab1-extra`` in case you finished
extra-credit challenge.

**Grading example**

```
$ make grade
make clean
make[1]: Entering directory '/home/red9057/jos'
rm -rf obj .gdbinit jos.in qemu.log
make[1]: Leaving directory '/home/red9057/jos'
./grade-lab1
make[1]: Entering directory '/home/red9057/jos'
+ as kern/entry.S
+ cc kern/entrypgdir.c
+ cc kern/init.c
+ cc kern/console.c
+ cc kern/monitor.c
+ cc kern/printf.c
+ cc kern/kdebug.c
+ cc lib/printfmt.c
+ cc lib/readline.c
+ cc lib/string.c
+ ld obj/kern/kernel
ld: warning: section '.bss' type changed to PROGBITS
+ as boot/boot.S
+ cc -Os boot/main.c
+ ld boot/boot
boot block is 390 bytes (max 510)
+ mk obj/kern/kernel.img
make[1]: Leaving directory '/home/red9057/jos'
running JOS: (0.6s)
  printf: OK
  backtrace count: OK
  backtrace arguments: OK
  backtrace symbols: OK
  backtrace lines: OK
Score: 50/50
```

**Sample Output:** You may get a different result for the address or
source line numbers, however, the number of backtraces and
each of function/file names must be matched.

```
$ make qemu-nox
***
*** Use Ctrl-a x to exit qemu
***
qemu-system-i386 -nographic -drive file=obj/kern/kernel.img,index=0,media=disk,format=raw -serial mon:stdio -gdb tcp::26003 -D qemu.log
444544 decimal is XXX octal!
entering test_backtrace 5
entering test_backtrace 4
entering test_backtrace 3
entering test_backtrace 2
entering test_backtrace 1
entering test_backtrace 0
Stack backtrace:
  ebp f010ff18  eip f0100078  args 00000000 00000000 00000000 f010004a f0111308
         kern/init.c:18: test_backtrace+56
  ebp f010ff38  eip f01000a1  args 00000000 00000001 f010ff78 f010004a f0111308
         kern/init.c:16: test_backtrace+97
  ebp f010ff58  eip f01000a1  args 00000001 00000002 f010ff98 f010004a f0111308
         kern/init.c:16: test_backtrace+97
  ebp f010ff78  eip f01000a1  args 00000002 00000003 f010ffb8 f010004a f0111308
         kern/init.c:16: test_backtrace+97
  ebp f010ff98  eip f01000a1  args 00000003 00000004 00000000 f010004a f0111308
         kern/init.c:16: test_backtrace+97
  ebp f010ffb8  eip f01000a1  args 00000004 00000005 00000000 f010004a f0111308
         kern/init.c:16: test_backtrace+97
  ebp f010ffd8  eip f01000f4  args 00000005 00001aac 00000640 00000000 00000000
         kern/init.c:39: i386_init+78
  ebp f010fff8  eip f010003e  args 00000003 00001003 00002003 00003003 00004003
         kern/entry.S:83: <unknown>+0
leaving test_backtrace 0
leaving test_backtrace 1
leaving test_backtrace 2
leaving test_backtrace 3
leaving test_backtrace 4
leaving test_backtrace 5
Welcome to the JOS kernel monitor!
Type 'help' for a list of commands.
K> help
help - Display this list of commands
kerninfo - Display information about the kernel
backtrace - Show the backtrace of the current kernel stack
K> backtrace
Stack backtrace:
  ebp f010ff58  eip f0100b1a  args 00000001 f010ff80 00000000 f0100b7e f0100b2d
         kern/monitor.c:154: monitor+332
  ebp f010ffd8  eip f0100101  args 00000000 00001aac 00000640 00000000 00000000
         kern/init.c:43: i386_init+91
  ebp f010fff8  eip f010003e  args 00000003 00001003 00002003 00003003 00004003
         kern/entry.S:83: <unknown>+0
```
