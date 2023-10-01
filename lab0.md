# Lab 0: Getting Real

## References

This set of lab notes is based on the [UCB CS-162 Fall 2020](https://inst.eecs.berkeley.edu/~cs162/fa20/) class,  [JHU CS-318 Project 0: Getting Real](https://www.cs.jhu.edu/~huang/cs318/fall21/project/project0.html) and the [Pintos Lab Guide](https://alfredthiel.gitbook.io/pintosbook/) provided by Peking University.

Some other recommended readings include:

- [Stanford CS-140 Pintos Specification](https://web.stanford.edu/class/cs140/projects/pintos/pintos.pdf)

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
- For a list of Interruptions, see the referenced [Ralf Brown's Interrupt List](https://www.ctyme.com/rbrown.htm) (i.e., `IntrList`).

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

This time we turn back to the codes in `loader.S` for `read_mbr`. (FYI: `mbr` stands for `Master Boot Records`).

Notice in the parts covered by the previous question, `read_sector` modified error code in `CF` (`int $0x13`).

Follow through the code, `jc no_such_drive` branches for drive read error. If no such error exists, status messages are printed to show the disk and partition being scanned.

Then:

```assembly
# Check for MBR signature--if not present, it's not a
# partitioned hard disk.
cmpw $0xaa55, %es:510
jne next_drive

mov $446, %si   # Offset of partition table entry 1.
mov $'1', %al
check_partition:
    # Is it an unused partition?
    cmpl $0, %es:(%si)
    je next_partition

    # Print [1-4].
    call putc

    # Is it a Pintos kernel partition?
    cmpb $0x20, %es:4(%si)
    jne next_partition

    # Is it a bootable partition?
    cmpb $0x80, %es:(%si)
    je load_kernel

next_partition:
    # No match for this partition, go on to the next one.
    add $16, %si # Offset to next partition table entry.
    inc %al
    cmp $510, %si
    jb check_partition
```

So the `MBR` should have the feature of ending with `0xaa55` at address `%es:510` (`510 = 0x1FE`).
If the bits recorded does not match, the bootloader will jump to the next drive to try again (`jne next_drive`).

Then jump to check partition records. (`offset = 446`).

- If read result is zero, then this partition is apparently unused, hence will lead to next partition.
- Pintos kernel uses a type of `0x20` for partition type. (see comments before `read_mbr`) If not match: jump to next partition.
- Check the first byte of the value recorded in the current partition. If `0x80` is detected then this is bootable.

If all of the checks above are pass, then it means we have found the partition for Pintos kernel.

---

> What happens when the bootloader could not find the Pintos kernel?

Read through the codes:

```assembly
next_partition:
    # No match for this partition, go on to the next one.
    add $16, %si        # Offset to next partition table entry.
    inc %al
    cmp $510, %si
    jb check_partition
next_drive:
    # No match on this drive, go on to the next one.
    inc %dl
    jnc read_mbr

no_such_drive:
no_boot_partition:
    # Didn't find a Pintos kernel partition anywhere, give up.
    call puts
    .string "\rNot found\r"

    # Notify BIOS that boot failed.  See [IntrList].
    int $0x18
```

`next_partition` will fall through to `next_drive` if partitions in current drive are all read. If all drives are read and no Pintos kernel is found, `jnc read_mbr` will not be executed and hence the program will fall through to `no_such_drive`, which further leads to falling through to `no_boot_partition` (no partition for booting is found). It will then output error message and return an interruption indicating [Interrupt 18](http://www.ctyme.com/intr/rb-2241.htm), no bootable disk available to the system.

---

> At what point and how exactly does the bootloader transfer control to the Pintos kernel?

After finding a satisfactory kernel location, we will then move to loading the kernel (also transferring control from 16-bit mode to 32-bit mode).

By inspecting the codes for `load_kernel` and simultaneously inspecting the execution logic, we found out that the loader will first read the partition's contents into the start load address `0x2000` (but capped at a size of `512kiB` due to BIOS restrictions).

> The kernel is at the beginning of the partition, which might be larger than necessary due to partition boundary alignment conventions, so the loader reads no more than `512 kB` (and the Pintos build process will refuse to produce kernels larger than that). Reading more data than this would cross into the region from `640 kB` to `1 MB` that the PC architecture reserves for hardware and the BIOS, and a standard PC BIOS does not provide any means to load the kernel above `1 MB`.

After such reading process, the loader will eventually pass control to the kernel.

```assembly
#### Transfer control to the kernel that we loaded.  We read the start
#### address out of the ELF header (see [ELF1]) and convert it from a
#### 32-bit linear address into a 16:16 segment:offset address for
#### real mode, then jump to the converted address.  The 80x86 doesn't
#### have an instruction to jump to an absolute segment:offset kept in
#### registers, so in fact we store the address in a temporary memory
#### location, then jump indirectly through that location.  To save 4
#### bytes in the loader, we reuse 4 bytes of the loader's code for
#### this temporary pointer.

    mov $0x2000, %ax
    mov %ax, %es
    mov %es:0x18, %dx
    mov %dx, start
    movw $0x2000, start + 2
    ljmp *start
```

The location of the kernel is stored at a pointer address of its `ELF` header, here namely at address `0x18`. Then this pointer's address data was saved in `dx` and later moved to `start`. See the code snippets for the reason of doing so.

The `start` has recorded the entrance to the kernel and hence have kernel taken control.

(For further information plz refer to _Appendix A_ in the specification).

### Exercise 2.3

Suppose we are interested in **tracing the behavior of one kernel function `palloc_get_page()` and one global variable `uint32_t *init_page_dir`**. For this exercise, you do not need to understand their meaning and the terminology used in them. You will get to know them better in Lab3: Virtual Memory.

> Trace the Pintos kernel and answer the following questions in your design document:
>
> - At **the entry of** `pintos_init()`, **what is the value of the expression** `init_page_dir[pd_no(ptov(0))]` in hexadecimal format?
> - When `palloc_get_page()` is called for the first time,
>   - what does **the call stack** look like?
>   - what is **the return value** in hexadecimal format?
>   - what is **the value of expression** `init_page_dir[pd_no(ptov(0))]` in hexadecimal format?
> - When `palloc_get_page()` is called for the third time,
>   - what does **the call stack** look like?
>   - what is **the return value** in hexadecimal format?
>   - what is **the value of expression** `init_page_dir[pd_no(ptov(0))]` in hexadecimal format?

#### Hints and Tips

- The GDB command **_p_** may be helpful.
- After the Pintos kernel takes control, **the initial setup is done in assembly code `threads/start.S`**.
- Later on, **the kernel will finally kick into the C world by calling the `pintos_init()` function in `threads/init.c`**.
- **Set a breakpoint at `pintos_init()`** and then continue tracing a bit into the C initialization code. Then read the source code of `pintos_init()` function.

### Notes and Answers 2.3

We invoke Pintos and `gdb` normally as the manual instructed.

---

> - At **the entry of** `pintos_init()`, **what is the value of the expression** `init_page_dir[pd_no(ptov(0))]` in hexadecimal format?

Here are the `gdb` interaction records:

```plain
(gdb) b pintos_init()
Function "pintos_init()" not defined.
Make breakpoint pending on future shared library load? (y or [n])
(gdb) b pintos_init
Breakpoint 1 at 0xc00202b6: file ../../threads/init.c, line 78.
(gdb) b palloc_get_page
Breakpoint 2 at 0xc0023114: file ../../threads/palloc.c, line 112.
(gdb) c
Continuing.
The target architecture is assumed to be i386
=> 0xc00202b6 <pintos_init>:    push   %ebp

Breakpoint 1, pintos_init () at ../../threads/init.c:78
(gdb) p init_page_dir[pd_no(ptov(0))]
=> 0xc000efef:  int3   
=> 0xc000efef:  int3   
$1 = 0
```

Hence at the entry of `pintos_init`, the value of the expression `init_page_dir[pd_no(ptov(0))]` in hexadecimal form is `0`.

---

> - When `palloc_get_page()` is called for the first time,
>   - what does **the call stack** look like?
>   - what is **the return value** in hexadecimal format?
>   - what is **the value of expression `init_page_dir[pd_no(ptov(0))]`** in hexadecimal format?

Following the previous question, here are the new commands I issued:

```plain
(gdb) c
Continuing.
=> 0xc0023114 <palloc_get_page>:        push   %ebp

Breakpoint 2, palloc_get_page (flags=(PAL_ASSERT | PAL_ZERO)) at ../../threads/palloc.c:112
(gdb) backtrace
#0  palloc_get_page (flags=(PAL_ASSERT | PAL_ZERO)) at ../../threads/palloc.c:112
#1  0xc00203aa in paging_init () at ../../threads/init.c:168
#2  0xc002031b in pintos_init () at ../../threads/init.c:100
#3  0xc002013d in start () at ../../threads/start.S:180
(gdb) finish
Run till exit from #0  palloc_get_page (flags=(PAL_ASSERT | PAL_ZERO)) at ../../threads/palloc.c:112
=> 0xc00203aa <paging_init+17>: add    $0x10,%esp
0xc00203aa in paging_init () at ../../threads/init.c:168
Value returned is $2 = (void *) 0xc0101000
(gdb) p/x init_page_dir[pd_no(ptov(0))]
=> 0xc000ef8f:  int3
=> 0xc000ef8f:  int3
$3 = 0x0
```

To inspect call stack: use `backtrace` (aka `bt`).

```plain
(gdb) backtrace
```

This displayed the call stack frame of:

```plain
#0  palloc_get_page (flags=(PAL_ASSERT | PAL_ZERO)) at ../../threads/palloc.c:11
2
#1  0xc00203aa in paging_init () at ../../threads/init.c:168
#2  0xc002031b in pintos_init () at ../../threads/init.c:100
#3  0xc002013d in start () at ../../threads/start.S:180
```

For the return value, issue `finish` (a.k.a., `fi`) command to inspect return value.

```plain
(gdb) finish
Run till exit from #0  palloc_get_page (flags=(PAL_ASSERT | PAL_ZERO)) at ../../threads/palloc.c:112
=> 0xc00203aa <paging_init+17>: add    $0x10,%esp
0xc00203aa in paging_init () at ../../threads/init.c:168
Value returned is $2 = (void *) 0xc0101000
```

The return value is a `(void *)` pointer to address `0xc0101000`.

For the value of expression `init_page_dir[pd_no(ptov(0))]`: issue `p/x` (`print in hexadecimal format`) command.

```plain
(gdb) p/x init_page_dir[pd_no(ptov(0))]
=> 0xc000ef8f:  int3
=> 0xc000ef8f:  int3
$3 = 0x0
```

Hence after the first call to `palloc_get_page()`, the value of `init_page_dir[pd_no(ptov(0))]` is still `0`.

---

> - When `palloc_get_page()` is called for the third time,
>   - what does **the call stack** look like?
>   - what is **the return value** in hexadecimal format?
>   - what is **the value of expression** `init_page_dir[pd_no(ptov(0))]` in hexadecimal format?

Similarly, `continue` to run the program for two more times to reach the third call.

```plain
(gdb) c
Continuing.
=> 0xc0023114 <palloc_get_page>:        push   %ebp

Breakpoint 2, palloc_get_page (flags=(PAL_ASSERT | PAL_ZERO)) at ../../threads/palloc.c:112
(gdb) c
Continuing.
=> 0xc0023114 <palloc_get_page>:        push   %ebp

Breakpoint 2, palloc_get_page (flags=PAL_ZERO) at ../../threads/palloc.c:112
```

The call stack at this moment looks like:

```plain
(gdb) bt
#0  palloc_get_page (flags=PAL_ZERO) at ../../threads/palloc.c:112
#1  0xc0020a81 in thread_create (name=0xc002e895 "idle", priority=0, function=0xc0020eb0 <idle>, aux=0xc000efbc) at ../../threads/thread.c:178
#2  0xc0020976 in thread_start () at ../../threads/thread.c:111
#3  0xc0020334 in pintos_init () at ../../threads/init.c:119
#4  0xc002013d in start () at ../../threads/start.S:180
```

We will print the return value in hexadecimal format:

```plain
Run till exit from #0  palloc_get_page (flags=PAL_ZERO) at ../../threads/palloc.c:112
=> 0xc0020a81 <thread_create+55>:       add    $0x10,%esp
0xc0020a81 in thread_create (name=0xc002e895 "idle", priority=0, function=0xc0020eb0 <idle>, aux=0xc000efbc) at ../../threads/thread.c:178
Value returned is $4 = (void *) 0xc0103000
```

We can see that this is still a `void *` pointer pointing to address `0xc0103000`.

At this time, we again inspect the value of `init_page_dir[pd_no(ptov(0))]`:

```plain
(gdb) p/x init_page_dir[pd_no(ptov(0))]
=> 0xc000ef4f:  int3   
=> 0xc000ef4f:  int3   
$5 = 0x102027
```

Hence the value of `init_page_dir[pd_no(ptov(0))]` at this time would be `0x102027`.

## Task 3: Kernel Monitor

### Exercise 3.1

At last, you will get to make a small enhancement to Pintos and write some code!

- In particular, when Pintos finishes booting, it will check for the supplied command line arguments stored in the kernel image. Typically you will pass some tests for the kernel to run, e.g., `pintos -- run alarm-zero`.
- If there is no command line argument passed (i.e., `pintos --`, note that `--` is needed as a separator for the pintos perl script and is not passed as part of command line arguments to the kernel), the kernel will simply finish up. This is a little boring.

**Your task is to add a tiny kernel shell** to Pintos so that when no command line argument is passed, it will run this shell interactively.

- Note that this is a kernel-level shell. In later projects, you will be enhancing the user program and file system parts of Pintos, at which point you will get to run the regular shell.
- **You only need to make this monitor very simple.** Its requirements are described below.

> Enhance `threads/init.c` to implement a tiny kernel monitor in Pintos.
>
> Requirments:
>
> - It starts with a prompt `PKUOS>` and waits for user input.
> - **As the user types in a printable character, display the character.**
> - When a newline is entered, it parses the input and checks if it is **`whoami`**. If it is **`whoami`**, print your student id. Afterward, the monitor will print the command prompt `PKUOS>` again in the next line and repeat.
> - If the user input is **`exit`**, the monitor will quit to allow the kernel to finish. For the other input, print invalid command. Handling special input such as backspace is not required.
> - If you implement such an enhancement, mention this in your design document.

The code place for you to add this feature is in line 136 of `threads/init.c` with

```c
// TODO: no command line passed to kernel. Run interactively.
```

You may need to use some functions provided in `lib/kernel/console.c`, `lib/stdio.c` and `devices/input.c`.

### Notes and Solutions 3.1

See code.

**Tips:**

1. the standard functions from `stdlib` should not be taken for granted as they are actually not part of the program. Instead they should be manually imported from other libraries.
2. General tips for writing `c` programs: remember to `free` after memory allocation and also remember to be careful during `string` treatment.

#### Code

```c
  if (*argv != NULL) {
    /* Run actions specified on kernel command line. */
    run_actions (argv);
  } else {
    // TODO: no command line passed to kernel. Run interactively 
    size_t command_max_input = 20;
    char* buf = (char *) malloc(command_max_input);
    while (true) {
      printf("PKUOS>");
      memset(buf, '\0', command_max_input);
      size_t index = 0;
      while (true) {
        char c = input_getc();
        if (c == new_line_char) {
          printf("\n"); // a line terminates here and switch to the next line
          break;
        } else 
        if (c == backspace_char) {
          if (index > 0) {
            buf[--index] = '\0';
            printf("\b \b"); // double backspace
          }
          continue;
        }
        buf[index++] = c;
        if (isprint(c)) {
          printf("%c", c);
        }
      }
      if (strcmp(buf, "whoami") == 0) {
        printf("MyStudentID\n");
      } 
      else
      if (strcmp(buf, "exit") == 0) {
        break;
      } 
      else 
      {
        printf("Invalid Command\n");
      }
    }
    free(buf);
    printf("Interactive Shell Terminated.\n");
  }

  /* Finish up. */
  shutdown ();
  thread_exit ();
```
