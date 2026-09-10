# HW1 (W4118 Fall 2026)

> <span style="color:red">**DUE: Wednesday 9/23/2026 at 11:59pm ET**</span>

All homework submissions are to be made via [Git](http://git-scm.com/).
You must submit a detailed list of references as part your homework
submission indicating clearly what sources you referenced for each
homework problem. You do not need to cite the course textbooks and
instructional staff. All other sources must be cited. Please edit and
include this [file](https://nieh.net/teaching/w4118_f26/homeworks/references.txt) in the top-level directory of your
homework submission. **Homeworks submitted without this file will
receive an automatic deduction of 10 points.**

### Before You Begin Programming

Before you begin the programming for this assignment, you must first do
two things: get access to GitHub and set up a Google Cloud (GCP) Arm
virtual machine (VM) that you will use for your development work.

You will be using Git via GitHub for course submissions for the class.
Please make sure you sign up for a GitHub account if you do not yet have
one, and follow the
[instructions](https://w4118.github.io/guides/git.html) for the W4118
GitHub organization, including filling out the Google Form listed there
so that we can associate your GitHub username with your Columbia UNI.
**You must complete this form by Thursday 9/10 at 11:59pm ET.**

Once you have submitted the form, the instructional staff will create
a private GitHub repository for this assignment in the W4118
organization and invite you to it. Accept the invitation (GitHub emails
it to you, and it is also listed at
[github.com/notifications](https://github.com/notifications)). The
repository can then be cloned using
`git clone git@github.com:W4118/f26-hmwk1-UserName.git` (replace
UserName with your own GitHub username). If you have not received an
invitation within a day of submitting the form, contact the
instructional staff. **Be aware that commits pushed
after the deadline will not be considered.** Refer to the homework
policy section on the [class web
site](https://www.cs.columbia.edu/~nieh/teaching/w4118/) for further
details.

You will be using a VM that you will set up for all homework
assignments. Follow the [setup
instructions](https://w4118.github.io/guides/gcp_setup.html) to create
the VM, then the [SSH guide](https://w4118.github.io/guides/gcp-ssh-guide.html)
to connect to it.

### Programming Problems

For all programming problems you will be required to submit source code,
Makefile(s), a README file documenting your files and code, and a test
run of your programs. The README should explain any way in which your
solution differs from what was assigned, and any assumptions you made.
For this assignment, you will have a separate subdirectory for each part
of the assignment, and each subdirectory should contain its own Makefile
and source code. **You must provide a Makefile for each part of this
assignment.** The README should be placed in the top level directory of
your GitHub repository for this assignment. Refer to the homework
submission page on the class web site for additional submission
instructions. In addition, please pay attention to the [additional
requirements](#additional-requirements) listed at the bottom of this assignment.

Part 1: The Simple Shell
------

An operating system like Linux makes it easy to run programs. For
example, from a shell, it is easy to write, compile, and run a simple
hello world C program:

    $vi hello.c
    #include <stdio.h>
    int main() { printf("hello, world\n"); }
    $gcc hello.c -o hello
    $./hello
    hello, world

The operating system makes this easy by providing various functions to
enable the program to perform I/O such as printing, and the shell to
execute the program in response to typing the program executable name at
the shell prompt. The shell itself is just another program. For example,
the Bash shell is an executable named `bash` that is usually
located in the /bin directory. So, /bin/bash.

Try running `/bin/bash` or just `bash` on a Linux (or
BSD-based, such as Mac OS X) operating system\'s command line, and
you\'ll likely discover that it will successfully run just like any
other program. Type `exit` to end your shell session and return to
your usual shell. (If your system doesn\'t have Bash, try running
`sh` instead.) When you log into a computer, this is essentially
what happens: Bash is executed. The only special thing about logging in
is that a special entry in `/etc/passwd` determines what shell
runs at log in time.

Your VM does not come with a C compiler or `make`. Install them once
before you start (the [setup
guide](https://w4118.github.io/guides/gcp_setup.html) also covers this):

    sudo apt install build-essential

Write a simple shell in C. The requirements are as follows.

1. **Your shell executable should be named w4118_sh.** Your shell source
code should be mainly in shell.c, but you are free to add additional
source code files as long as your Makefile works, and compiles and
generates an executable named w4118_sh in the same top level directory
as the Makefile. If we cannot simply run make and then w4118_sh, you
will be heavily penalized.

2. **The shell should run continuously, and display a prompt when waiting
for input.** The prompt should be EXACTLY `$`. No spaces, no extra
characters. Example with a command:

        $/bin/ls -lha /home/w4118/my_docs

3. **Your shell should read a line from stdin one at a time.** This line
should be parsed out into a *command* and *all its arguments*. In other
words, tokenize it.

    - You may assume that the only supported delimiter is the whitespace
      character (ASCII character number 32).
    - You do not need to handle \"special\" characters. Do not worry about
      handling quotation marks, backslashes, and tab characters. This means
      your shell will be unable support arguments with spaces in them. For
      example, your shell will not support file paths with spaces in them.
    - You may set a reasonable maximum on the number of command line
      arguments, but your shell should handle input lines of any length. You
      may find `getline()` useful.

4. **After parsing and lexing the command, your shell should execute it.**
A command can either be a reference to an executable OR a built-in shell
command (see below). For now, just focus on running executables, and not
on built-in commands.

    - Executing commands that are not shell built-ins is done by invoking
      `fork()` and then invoking `exec()`
    - You may **NOT** use the `system()` function, as it just invokes
      the `/bin/sh` shell to do all the work.

5. **Ensure Ctrl-C works.** Typing Ctrl-C in your shell should function as
expected, that is, if a command is running, the command will terminate,
but your shell should not terminate.

6. **Implement Built-in Commands, `exit` and `cd`.** `exit`
simply exits your shell after performing any necessary clean up. `cd
[dir]`, short for \"change directory\", changes the current
working directory of your shell. Do not worry about implementing the
command line options that the real cd command has in Bash. Just
implement cd such that it takes a single command line parameter: the
directory to change to. cd should be done by invoking `chdir()`.

7. **Error messages should be printed using exactly one of two string
formats.** The first format is for errors where
[`errno`](http://linux.die.net/man/3/errno) is set. The second
format is for when `errno` is not set, in which case you may
provide any error text message you like on a single line.

        "error: %s\n", strerror(errno)

        OR

        "error: %s\n", "your error message"
    So for example, you would likely use: `fprintf(stderr, "error: %s\n", strerror(errno));`

8. **Check the return values of all functions utilizing system resources.**
Do not blithely assume all requests for memory will succeed and all
writes to a file will occur correctly. Your code should handle errors
properly. Many failed function calls should not be fatal to a program.

    Typically, a system call will return -1 in the case of an error (malloc will return NULL). If a function call sets the errno variable (see the function\'s man page to find out if it does), you should use the first error message as described above. As far as system calls are concerned, you will want to use one of the mechanisms described in [Error Reporting](https://www.gnu.org/software/libc/manual/html_mono/libc.html#Error-Reporting).

9. **A testing script skeleton is provided in a [GitHub
repository](https://github.com/W4118/f26-tester-hmwk1) to help you with
testing your program.** You should make sure your program works correct
with this script. For grading purposes, we will conduct much more
extensive testing than what is provided with the testing skeleton, so
you should make sure to write additional test cases yourself to test
your code.

Part 2: Simple Shell Directly Calling System Calls
------

The simple shell you wrote in Part 1 relies on various C library
functions that in turn call system calls. You can use `strace` to
see what system calls are being called when you run simple shell. First,
install strace:

    sudo apt install strace

Then you can run `strace` with simple shell:

    strace -o trace.txt ./w4118_sh

which will dump the system calls executed into the file trace.txt. For
example, if you used `printf()` to output text in simple shell,
you will find that it in turn calls a system call to actually perform
the I/O operation because I/O is controlled by the operating system. C
library functions such as `printf()` are technically not part of
the C language, but made possible by relying on functionality provided
by the operating system.

If you want to trace not only the system calls executed by the shell but
any processes it creates, you can add the follow-forks option:

    strace -o trace.txt -f ./w4118_sh

To gain a better understanding of how C library functions rely on
operating system functionality, modify your simple shell so that it does
not call any C library functions that call other system calls. Instead,
your simple shell should directly call any system calls that it
implicitly uses. For example, your simple shell should not call
`printf()` but instead call `write()` on STDOUT. Other C
library functions that you may also have to replace include
`getline()`, `malloc()`, etc. You do not have to be overly
concerned with efficiency, so you may find it easier to use
[`mmap()`](https://man7.org/linux/man-pages/man2/mmap.2.html)
instead of `sbrk()` for any dynamic memory allocation you need to
do. For example, you may find it helpful to see this
[implementation](https://elixir.bootlin.com/linux/v7.0/source/tools/include/nolibc/stdlib.h#L129)
of `malloc()`. String manipulation functions such as `strtok` and `strcmp` do not call system calls and do not need to be replaced.

Linux provides system calls which may have duplicative functionality and
which system calls your simple shell uses depends the implementation
choices made by the C library; you should carefully check the trace you
generated to see which system calls the shell should directly call. For
example, is the `fork()` system call actually used? You may find
it useful in some cases to utilize the the general purpose `syscall(2)` system call. You can consult its man page for more details: `man 2 syscall`

Note that your implementation of the various functions only has to work
specifically for your simple shell. For example, you do not need to
implement all functionality supported by `printf()`, only what
functionality is required to print the output that your shell generates.
Similarly, your input functionality only needs to work for any ascii
characters generated from a keyboard.

**Your shell executable should be named w4118_sh2.** Your shell source
code should be mainly in shell2.c, but you are free to add additional
source code files as long as your Makefile works, and compiles and
generates an executable named w4118_sh2 in the same top level directory
as the Makefile. If we cannot simply run make and then w4118_sh2, you
will be heavily penalized. w4118_sh2 should have all the same
functionality as w4118_sh, except that it does not call any C library
functions that call other system calls.

Part 3: Bare-metal Hello World OS
------

Without an operating system, running a program on a computer is
harder. When the power button is pressed, the CPU is reset to its
initial state and firmware built into the machine runs. The firmware
checks the hardware, loads the first program it finds on the disk into
RAM, and transfers control to that program.

The firmware on modern computers, including the Arm servers that run
your GCP VM, follows a standard called UEFI (Unified Extensible
Firmware Interface). UEFI understands a simple filesystem, so it looks
for a program at a fixed path on the disk and runs it. On Arm machines
that path is `\EFI\BOOT\BOOTAA64.EFI`.

Usually the program at that path is a bootloader, which goes on to load
an operating system such as Linux. But it does not have to be. A
program that prints "hello, world" is also a valid operating system, as
long as it can be loaded by the firmware. What such a program does not
have is any of the comforts an operating system normally provides:
there is no C library, no `printf`, no `malloc`, and nothing running
underneath you.

There is also no screen. Your VM has no display at all. What it does
have is a serial port: a very simple device that sends one byte at a
time down a wire. On a cloud VM the other end of that wire is a log
that Google keeps for you. Everything your Hello World OS prints will
come out of that serial port.

1.  **Implement the Hello World OS.** Write a Hello World OS that boots
    under UEFI and prints "hello, world" to the serial port. Your code
    will be in C, plus two lines of inline assembly in step 4.

    We provide the starter code `part3/main.c` and a directory
    `part3/uefi` with header files describing the UEFI environment. Do
    not modify anything under `part3/uefi`; it is not your code, and
    the testing script's checkpatch wrapper skips it. Add your code to
    `main.c` below the comment that says `WRITE YOUR CODE BELOW THIS
    LINE`.

    The firmware calls this function in `main.c`:

        EFI_STATUS EFIAPI efi_main(EFI_HANDLE ImageHandle,
                                   EFI_SYSTEM_TABLE *SystemTable)

    `SystemTable` is a struct full of pointers to services the firmware
    offers. The only one you need is the console,
    `SystemTable->ConOut`, and its `OutputString` function, which
    writes a string to the serial port:

        SystemTable->ConOut->OutputString(SystemTable->ConOut, L"hi\r\n");

    Two things to note. UEFI strings are made of 16-bit characters
    (type `CHAR16`), so string literals are written with an `L` prefix.
    And the serial port behaves like a terminal, so a new line is
    `\r\n`: carriage return, then line feed.

    You should **print exactly `hello, world` on a line of its own, all
    lower case, with the comma after the first word and one space after
    the comma before the second word.** Print `\r\n` before it, so that
    it starts on a fresh line after the firmware's own messages, and
    `\r\n` after it.

    Once you have printed your message, do not let `efi_main` return.
    Returning hands control back to the firmware, which goes looking for
    something else to boot and, finding nothing, drops into its setup
    menu. A loop of some kind here would be useful.

    `SystemTable` gives you access to much more than the console, but
    only `ConOut` may be used in your submission.

2.  **Build the program and a disk image that holds it.** UEFI programs
    use the same executable file format as Windows, so the code is
    compiled with `clang` for a Windows-style target and linked with
    `lld`. Install the tools in your VM:

        sudo apt install make clang lld mtools

    Then compile and link:

        clang -target aarch64-unknown-windows -ffreestanding -fshort-wchar -mno-red-zone -Wall -c -o main.o main.c
        clang -target aarch64-unknown-windows -nostdlib -Wl,-entry:efi_main -Wl,-subsystem:efi_application -fuse-ld=lld-link -o BOOTAA64.EFI main.o

    `-ffreestanding` and `-nostdlib` tell the compiler there is no C
    library and no operating system. The output, `BOOTAA64.EFI`, is
    your entire operating system.

    The firmware needs to find that file at `\EFI\BOOT\BOOTAA64.EFI` on
    a disk formatted with the FAT filesystem. `mtools` lets you build
    such a disk image as a plain file, without mounting anything:

        dd if=/dev/zero of=disk.img bs=1k count=1440
        mformat -i disk.img -f 1440 ::
        mmd -i disk.img ::/EFI ::/EFI/BOOT
        mcopy -i disk.img BOOTAA64.EFI ::/EFI/BOOT

    The provided `Makefile` runs all of the above; `make` should produce
    `disk.img`.

3.  **Boot your disk image in QEMU.** QEMU is a machine emulator: it
    creates a virtual Arm computer inside your VM, with the same UEFI
    firmware family that real Arm cloud VMs use. This is your fast
    edit-build-test loop. Install QEMU and the Arm UEFI firmware:

        sudo apt install qemu-system-arm qemu-efi-aarch64

    Then boot:

        qemu-system-aarch64 -M virt -cpu cortex-a57 -m 512 \
            -bios /usr/share/AAVMF/AAVMF_CODE.fd \
            -drive file=disk.img,format=raw,if=virtio -nic none -nographic

    `-nographic` connects the virtual machine's serial port to your
    terminal, so this works over SSH. You will first see a few lines
    from the firmware as it starts up (some look like errors; that is
    normal), then, after a few seconds, `hello, world`. **To exit QEMU,
    press `Ctrl-A` and then `X`.**

    If instead the firmware prints `BdsDxe: failed to load` or drops you
    into a `Shell>` prompt, it did not find your program. Check that
    `disk.img` contains it with `mdir -i disk.img ::/EFI/BOOT`.

4.  **Add program counter and stack pointer information to your
    output.** A CPU uses a program counter (PC) and a stack to run C
    programs. The PC holds the address in memory of the instruction the
    CPU is executing. The stack pointer (SP) holds the address of the
    top of the stack, the region of memory used for function calls: when
    a function is called, the address to return to is saved on the
    stack, so that when the function finishes, execution can continue
    right after the call. On Arm the PC and SP live in CPU registers.
    You will read them and print them along with your message.

    **The exact format of your message** should be
    `hello, world pc sp`, with one space before the `pc` value and one
    space before the `sp` value, followed by `\r\n`. The `pc` should be
    the address of the instruction right after the code used to output
    "hello, world". The `sp` should be the stack pointer value at that
    same point. Each value is a 64-bit number and must be printed as
    exactly 16 lower case hexadecimal digits, with no `0x` and no
    padding. For example:

        hello, world 000000005cb56034 00000000476869e0

    Reading a register requires assembly. We have included the lines
    you need in `main.c`:

        UINT64 pc_value, sp_value;
        __asm__ volatile ("adr %0, ." : "=r" (pc_value));
        __asm__ volatile ("mov %0, sp" : "=r" (sp_value));

    `adr %0, .` computes the address of the instruction it is part of
    (`.` means "here" in assembly) and the compiler stores the result in
    `pc_value`. `mov %0, sp` copies the stack pointer register into
    `sp_value`. Place these lines immediately after the code that prints
    "hello, world", so the values are the ones asked for above.

    To print the values, convert each 4-bit group of the number into a
    hexadecimal digit character yourself, build a `CHAR16` string from
    the digits, and print it with `OutputString`.

    Once you have completed your program, redo steps 2 and 3 to rebuild
    `disk.img` and boot it in QEMU.

5.  **Boot your Hello World OS on your GCP VM's real hardware.** So far
    QEMU has been pretending to be an Arm computer. Your GCP VM runs on
    an actual Arm server, and Google's firmware looks for the same
    `\EFI\BOOT\BOOTAA64.EFI` on the boot disk and sends the console to
    the VM's serial port, which Google records for you. So the same
    `BOOTAA64.EFI` you just built can be the operating system of a GCP
    VM, and you can read what it printed from the GCP console.

    **Do this on a new, throwaway VM created from your own disk image.
    Never copy `BOOTAA64.EFI` onto the boot disk of the VM you do your
    coursework on.** That VM would boot your Hello World OS instead of
    Ubuntu, forever, and you would not be able to log in again.

    GCP requires a bootable disk image to be a raw disk whose size is a
    whole number of gigabytes, with a GPT partition table and a FAT
    partition. Build one from your `BOOTAA64.EFI`:

        sudo apt install parted
        truncate -s 1G disk.raw
        parted -s disk.raw mklabel gpt mkpart ESP fat32 1MiB 100% set 1 esp on
        mformat -i disk.raw@@1M -F ::
        mmd -i disk.raw@@1M ::/EFI ::/EFI/BOOT
        mcopy -i disk.raw@@1M BOOTAA64.EFI ::/EFI/BOOT
        tar --format=oldgnu -Sczf hello-os.tar.gz disk.raw

    (`@@1M` tells `mtools` that the filesystem starts 1 MiB into the
    file, after the partition table. The file inside the tarball must
    be named exactly `disk.raw`.)

    The remaining commands use `gcloud`, which is already installed on
    your VM. By default `gcloud` on a VM acts as the VM's own service
    account, which is not allowed to create images or VMs. Log in as
    yourself once so it acts as you instead:

        gcloud auth login

    and follow the printed link. (Alternatively, run the `gcloud`
    commands below from your own computer, after copying
    `hello-os.tar.gz` to it.)

    Upload the tarball to a Cloud Storage bucket in your project, turn
    it into an image, and create a VM from that image. First set two
    shell variables to your project ID and the zone of your VM, so the
    commands below can be pasted as they are (they must be run in the
    same terminal):

        export PROJECT_ID=your-project-id
        export ZONE=us-central1-c

    Then:

        gcloud storage buckets create gs://$PROJECT_ID-hw1 --location=us-central1
        gcloud storage cp hello-os.tar.gz gs://$PROJECT_ID-hw1/
        gcloud compute images create hello-os \
            --source-uri gs://$PROJECT_ID-hw1/hello-os.tar.gz \
            --architecture=ARM64 --guest-os-features=UEFI_COMPATIBLE,GVNIC
        gcloud compute instances create hello-os-vm --zone=$ZONE \
            --machine-type=n4a-standard-1 --image=hello-os \
            --boot-disk-size=10GB --no-address

    The `GVNIC` feature declares that the image can use GCP's network
    card; GCP refuses to create the VM without it, even though your OS
    never touches the network. The boot disk must be at least 4 GB even
    though your image is smaller. If the last command fails with "does
    not have enough resources", that zone is out of Arm machines right
    now: try another zone in the same region, or
    `--machine-type=c4a-standard-1`.

    Wait about half a minute for the VM to power on, then read its
    serial port:

        gcloud compute instances get-serial-port-output hello-os-vm --zone=$ZONE

    You can also open the VM in the GCP console and click "Serial port
    1 (console)" under Logs. You should see the firmware's boot
    messages, then your line, then nothing more, since your OS never
    returns:

        UEFI firmware (version  built at 09:00:00 on Jan 10 2025)
        ...
        BdsDxe: starting Boot0001 "UEFI Misc Device" from PciRoot(0x0)/Pci(0x2,0x0)/NVMe(0x1,...)
        UEFI: Attempting to start image.
        ...
        hello, world 000000013c742034 00000000477e0b40

    That is your operating system running on a real Arm machine with
    nothing underneath it. The `pc` and `sp` values differ from the
    ones you saw in QEMU: they depend on where this firmware chose to
    load your program and place its stack.

    The throwaway VM costs money while it exists. Delete it, the image,
    and the bucket when you are done:

        gcloud compute instances delete hello-os-vm --zone=$ZONE
        gcloud compute images delete hello-os
        gcloud storage rm -r gs://$PROJECT_ID-hw1

    **Copy the serial port output from your GCP boot, from the first
    firmware line through your `hello, world` line, into your README**
    under a heading `Part 3 GCP boot`. Steps 1 to 4 are graded by
    booting your `disk.img` in QEMU; this step is graded from your
    README.

    Your `part3` directory must contain a `Makefile` whose default
    target builds `disk.img`. Do not commit `disk.img`, `BOOTAA64.EFI`,
    `disk.raw`, or any other build products.

### Additional Requirements

1.  All code (including test programs) must be written in C.
2.  Make at least ten commits with Git. The point is to make incremental
    changes and use an iterative development cycle.
3.  Follow the following coding style rules:
    - Tab size: 8 spaces.
    - Do not have more that 3 levels of indentations (unless the
      function is extremely simple).
    - Do not have lines that goes after the 80th column (with rare
      exceptions).
    - Do not comment your code to say obvious things. Use /\* \... \*/
      and not // \...
    - Follow the [Linux kernel coding
      style](https://www.kernel.org/doc/html/latest/process/coding-style.html).
      Use
      [checkpatch](https://github.com/torvalds/linux/blob/master/scripts/checkpatch.pl),
      a script which checks coding style rules.
4.  Use a makefile to control the compilation of your code. The makefile
    should have at least a default target that builds all assigned
    programs.
5.  When compiling your code with `gcc` or `clang`, use the `-Wall`
    switch to ensure that all warnings are displayed. **Do not** be satisfied with
    code that merely compiles; it should compile with no warnings. You
    will lose points if your code produces warnings when compiled.
6.  Check the return values of all functions utilizing system resources
    for all parts of the programming assignment.
7.  Your code should not have memory leaks and should handle errors
    gracefully.

### Tips

1.  For this assignment, your primary reference will be [Programming in
    C](http://users.cs.cf.ac.uk/dave/C/). You might also find the [Glibc
    Manual](http://www.gnu.org/software/libc/manual/) useful.
2.  Many questions about functions and system behaviour can be found in
    the system manual pages; type in `man` *function* to get more
    information about *function*. If *function* is a system call,
    `man 2` *function* can ensure that you receive the correct man page,
    rather than one for a system utility of the same name.
3.  If you are having trouble with basic Unix/Linux behavior, you might
    want to check out the resources section of the class webpage.
4.  A lot of your problems have happened to other people. If you have a
    strange error message, you might want to try searching for the
    message on [Google](http://www.google.com/).

## Submission Checklist

Include the following in your main branch. Only include source code (ie
\*.c,\*.h) and text files, do **not** include compiled objects.

- README file
- [references.txt](https://nieh.net/teaching/w4118_f26/homeworks/references.txt) file with a list of
  references to materials that you used to complete your assignment,
  including URLs to websites and names of other students you asked for
  help.
- Implementation of simple shell in [part1](#part-1-the-simple-shell) with Makefile
  generating w4118_sh executable
- Implementation of simple shell using only system calls in
  [part2](#part-2-simple-shell-directly-calling-system-calls) with Makefile generating w4118_sh2 executable
- Implementation of hello world OS in [part3](#part-3-bare-metal-hello-world-os) with Makefile
  generating the disk.img bootable UEFI disk image, and the GCP serial
  console output in your README

