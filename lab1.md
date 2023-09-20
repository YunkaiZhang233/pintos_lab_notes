# Lab 0: Getting Real

## References

This set of lab notes is based on the [UCB CS-162 Fall 2020](https://inst.eecs.berkeley.edu/~cs162/fa20/) class and the [Pintos Lab Guide](https://alfredthiel.gitbook.io/pintosbook/) provided by Peking University.

Some other recommended readings include:

- [JHU CS-318 Project 0: Getting Real](https://www.cs.jhu.edu/~huang/cs318/fall21/project/project0.html)
- Stanford

## Task 1: Booting Pintos

### Exercise 1.1

1.  
    Have Pintos development environment setup as described in [Environment Setup](https://alfredthiel.gitbook.io/pintosbook/getting-started/environment-setup).
2.  
    Afterwards, execute

    ```bash
    cd pintos/src/threads
    make
    cd build
    pintos --
    ```

    If everything works, you should see Pintos booting in the [QEMU emulator](https://www.qemu.org), and print `Boot complete` near the end.

    While by default we run Pintos in QEMU, Pintos can also run in the [Bochs](https://bochs.sourceforge.io) and VMWare Player emulator. Bochs will be useful for the Lab1: Threads. 

    To run Pintos with Bochs, execute

    ```bash
    cd pintos/src/threads
    make
    cd build
    pintos --bochs --
    ```

> Take screenshots of the successful booting of Pintos in QEMU and Bochs.

#### My Records

Well this is basically following-steps-and-mimicing. Should be simple.

Created shorthand for starting pintos (i.e., used `alias` linux utility and updated `docker run -it --rm --name pintos --mount type=bind,source=absolute/path/to/pintos/on/your/host/machine,target=/home/PKUOS/pintos pkuflyingpig/pintos bash` to an alias of `pintos-up` in order to avoid tedious memorizing).

#### Screenshot

(Actually command line duplicates here for easier maintenance)

##### QEMU

```plain
➜  ~ pintos-up
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
root@9008df090274:~# cd pintos/src/threads
root@9008df090274:~/pintos/src/threads# make
cd build && make all
make[1]: Entering directory '/home/PKUOS/pintos/src/threads/build'
make[1]: Nothing to be done for 'all'.
make[1]: Leaving directory '/home/PKUOS/pintos/src/threads/build'
root@9008df090274:~/pintos/src/threads# cd build
root@9008df090274:~/pintos/src/threads/build# pintos --
qemu-system-i386 -device isa-debug-exit -drive format=raw,media=disk,index=0,file=/tmp/scsXvvoKMi.dsk -m 4 -net none -nographic -monitor null
Pintos hda1
Loading...........
Kernel command line:
Pintos booting with 3,968 kB RAM...
367 pages available in kernel pool.
367 pages available in user pool.
Calibrating timer...  13,094,400 loops/s.
Boot complete.
```

##### Bochs

Tips: Use `^C` to quit emulator.

Following previous steps, just change `pintos --` to `pintos --bochs --`.

```plain
root@9008df090274:~/pintos/src/threads/build# pintos --bochs --
squish-pty bochs -q
========================================================================
                       Bochs x86 Emulator 2.6.2
                Built from SVN snapshot on May 26, 2013
                  Compiled on Mar  1 2022 at 16:09:16
========================================================================
00000000000i[     ] reading configuration from bochsrc.txt
00000000000e[     ] bochsrc.txt:8: 'user_shortcut' will be replaced by new 'keyboard' option.
00000000000i[     ] installing nogui module as the Bochs GUI
00000000000i[     ] using log file bochsout.txt
Pintos hda1
Loading...........
Kernel command line:
Pintos booting with 4,096 kB RAM...
383 pages available in kernel pool.
383 pages available in user pool.
Calibrating timer...  102,400 loops/s.
Boot complete.
```

## Task 2: Debugging

### Exercise 2.1

**While you are working on the projects, you will frequently use the GNU Debugger (GDB) to help you find bugs in your code.** Make sure you read the [Debugging](https://alfredthiel.gitbook.io/pintosbook/getting-started/debug-and-test/debugging) section first.

In addition, if you are unfamiliar with **x86 assembly**, the [PCASM](https://www.cs.jhu.edu/~huang/cs318/fall21/project/specs/pcasm-book.pdf) is an excellent book to start. Note that you don't need to read the entire book, just the basic ones are enough.

> Your first task in this section is to  use GDB to trace the QEMU BIOS a bit to understand how an IA-32 compatible computer boots. Answer the following questions in your design document:
>
> - What is the first instruction that gets executed?
> - At which physical address is this instruction located?

### Notes and Answers 2.1

Operations: just follow the steps. Trivial.

Starting `gdb` as instructed will automatically lead us to having executed `target remote localhost:1234` via calling `debugpintos`.

The first piece of information displayed:

```plain
(gdb) debugpintos
The target architecture is assumed to be i8086
[f000:fff0]    0xffff0: ljmp   $0x3630,$0xf000e05b
0x0000fff0 in ?? ()
```

Hence the first instruction get executed is `ljmp`, located at physical address `0x0000fff0`.

### Exercise 2.2

> **Trace the Pintos bootloader** and answer the following questions in your design document:
>
> - How does the bootloader read disk sectors? In particular, what BIOS interrupt is used?
> - How does the bootloader decide whether it successfully finds the Pintos kernel?
> - What happens when the bootloader could not find the Pintos kernel?
> - At what point and how exactly does the bootloader transfer control to the Pintos kernel?

#### Tips

- In the second task, you will be tracing the Pintos bootloader. **Set a breakpoint at address `0x7c00`, which is where the boot sector will be loaded.** Continue execution until that breakpoint.
- Trace through the code in `threads/loader.S`, using the source code and the disassembly file `threads/build/loader.asm` to keep track of where you are.
- Also, **use the `x/i` command** in GDB to disassemble sequences of instructions in the boot loader, and compare the original boot loader source code with both the disassembly in threads/build/loader.asm and GDB.

### Notes and Answers 2.2

Invoke `gdb` and set a breakpoint at `0x7c00`. Following 2.1, we will issue these commands:

```plain
(gdb) debugpintos
The target architecture is assumed to be i8086
[f000:fff0]    0xffff0: ljmp   $0x3630,$0xf000e05b
0x0000fff0 in ?? ()
(gdb) b *0x7c00
Breakpoint 1 at 0x7c00
(gdb) continue
Continuing.
[   0:7c00] => 0x7c00:  sub    %eax,%eax

Breakpoint 1, 0x00007c00 in ?? ()
```

#### General Tips for this Exercise

- **read THROUGH the codes** in `loader.S`.
- And also read the comments. They are quite self-explanatory.
- For this exercise, [The JHU Project 0](https://www.cs.jhu.edu/~huang/cs318/fall21/project/project0.html) provides a more detailed explanation.

---

> How does the bootloader read disk sectors? In particular, what BIOS interrupt is used?

At this point, Bootloader has taken control from BIOS and the instructions to be executed correspond to the ones in `loader.S`.

We will refer to `loader.S` and the corresponding `loader.asm` for code tracing.

From `loader.asm`: address `0x00007c00` corresponds to `<read_mbr>`. Leading us back to the part in `loader.S`: it starts at `line 51` and then calls `read_sector`.

Trace to `read_sector`: Here are its comments:

```assembly
#### Sector read subroutine.  Takes a drive number in DL (0x80 = hard
#### disk 0, 0x81 = hard disk 1, ...) and a sector number in EBX, and
#### reads the specified sector into memory at ES:0000.  Returns with
#### carry set on error, clear otherwise.  Preserves all
#### general-purpose registers.
```

We can see the major functionality of `read_sector` is to enable addressing mode switching via interruption and set flags. By viewing its code:

```assembly
read_sector:
    pusha
    sub %ax, %ax
    push %ax   # LBA sector number [48:63]
    push %ax   # LBA sector number [32:47]
    push %ebx   # LBA sector number [0:31]
    push %es   # Buffer segment
    push %ax   # Buffer offset (always 0)
    push $1    # Number of sectors to read
    push $16   # Packet size
    mov $0x42, %ah   # Extended read
    mov %sp, %si   # DS:SI -> packet
    int $0x13   # Error code in CF
    popa    # Pop 16 bytes, preserve flags
```

We can see the Error Code in CF is `0x13` with having the `ah` parameter as `0x42`. By referring to the [BIOS Interrupt Call Wikipedia Page](https://en.wikipedia.org/wiki/BIOS_interrupt_call), I noticed that:

> The interrupt number is specified as the parameter of the software interrupt instruction (in Intel assembly language, an "`INT`" instruction), and the function number is specified in the `AH` register; that is, the caller sets the `AH` register to the number of the desired function. In general, the BIOS services corresponding to each interrupt number operate independently of each other, but the functions within one interrupt service are handled by the same BIOS program and are not independent.

This further confirmed by hypothetical guess and by referencing the interruption table, we found that the specific interrupt used is `Extended Read Sectors`.

---

> How does the bootloader decide whether it successfully finds the Pintos kernel?

This time we turn back to the codes in `loader.S`. 
