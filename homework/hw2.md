# HW2 (W4118 Fall 2024)

> <span style="color:red">**DUE: Wednesday 10/2/2024 at 11:59pm ET**</span>

## General instructions

All homework submissions are to be made via [Git][Git]. You must submit a detailed list of references as part of your homework submission indicating clearly what sources you referenced for each homework problem. You do not need to cite the course textbooks and instructional staff. All other sources must be cited. Please edit and include this [file][file] in the top-level directory of your homework submission in the `main` branch of your team repo. **Be aware that commits pushed after the deadline will not be considered.** Refer to the homework policy section on the [class website][class-web-site] for further details.

[Git]: https://git-scm.com/
[file]: https://www.cs.columbia.edu/~nieh/teaching/w4118/homeworks/references.txt
[class-web-site]: https://www.cs.columbia.edu/~nieh/teaching/w4118/

Group programming problems are to be done in your assigned groups. We will let you know when the Git repository for your group has been set up on GitHub. It can be cloned using the following command. Replace `teamN` with your team number, e.g. `team0`. You can find your group number [here](https://docs.google.com/spreadsheets/d/1sWSEjtqSqSsl2dJ3R3YZKYlw4WG4GtfpHNECpqFJiks/edit?gid=0#gid=0).

```
$ git clone git@github.com:W4118/f24-hmwk2-teamN.git
```

> **IMPORTANT**: You should clone the repository **directly in the terminal of your VM**, instead of cloning it on your local machine and then copying it into your VM. The filesystem in your VM is case-sensitive, but your local machine might use a case-insensitive filesystem (for instance, this is the default setting on macs). Cloning the repository to a case-insensitive filesystem might end up clobbering some kernel source code files. See [this post](https://unix.stackexchange.com/questions/753038/is-building-the-linux-kernel-on-a-case-insensitive-filesystem-possible) for some examples.

This repository will be accessible to all members of your team, and all team members are expected to make local commits and push changes or contributions to GitHub equally. You should become familiar with team-based shared repository Git commands such as [git-pull][git-pull], [git-merge][git-merge], [git-fetch][git-fetch]. For more information, see [this guide](../guides/git.md).

There should be at least five commits **per member** in the team's Git repository. The point is to make incremental changes and use an iterative development cycle. Follow the [Linux kernel coding style](https://www.kernel.org/doc/html/v6.8/process/coding-style.html). You **must** check your commits with the `run_checkpatch.sh` script provided as part of your team repository. Errors from the script in your submission will cause a deduction of points. (Note that the script only checks the changes up to your latest commit. Changes in the working tree or staging area will not be checked.)

The kernel programming for this assignment will be run using your Linux VM. As part of this assignment, you will be experimenting with Linux platforms and gaining familiarity with the development environment. Linux platforms can run on many different architectures, but the specific platforms we will be targeting are the X86_64 or Arm64 CPU families. All of your kernel builds will be done in the same Linux VM from homework 1. You will be developing with the Linux 6.8 kernel.

**For this assignment, you will write a system call to dump the process tree and a user space program to use the system call.**

For students on Arm computers (e.g. macs with M1/M2/M3 CPU): if you want your submission to be built/tested for Arm, you must create and submit a file called `.armpls` in the top-level directory of your repo; feel free to use the following one-liner:

```
$ cd "$(git rev-parse --show-toplevel)" && touch .armpls && git add .armpls && git commit -m "Arm pls"
```

You should do this first so that this file is present in any code you submit for grading.

For all programming problems, you should submit your source code as well as a single README file documenting your files and code for each part. Please do NOT submit kernel images. The README should explain any way in which your solution differs from what was assigned, and any assumptions you made. You are welcome to include a test run in your README showing how your system call works. **It should also state explicitly how each group member contributed to the submission and how much time each member spent on the homework.** The README should be placed in the top level directory of the main branch of your team repo (on the same level as the `linux/` and `user/` directories).

[git-pull]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-pull.html
[git-merge]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-merge.html
[git-fetch]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-fetch.html

Part 1: Build your own Linux 6.8.0 kernel and install and run it in your Linux VM
------

You will need to install your own custom kernel in your VM to do this assignment. The source code for the kernel you will use is located in your team repository on GitHub. Follow the instructions provided [here](../guides/kernel-compilation.md) to build and install the kernel in your VM.

Part 2: Write a new system call in Linux
------

**General description**

The system call you write will retrieve information from each thread associated with each process in some subset of the process tree. That thread information will be stored in a buffer and copied to user space.

Within the buffer, threads associated with the same process (main thread) should be grouped together. These thread groups should be sorted in breadth-first-search (BFS) order of the associated process within the process tree. Within each thread group, the threads should be sorted in ascending order of PID. You may ignore PID rollover for the purposes of ordering threads. See the hint on `PID vs TGID` below for more information on what we mean when we say the PID of a thread. See also `Additional requirements` below for an example ordering.

The prototype for your system call will be:

```c
int ptree(struct tskinfo *buf, int *nr, int root_pid);
```

You should define `struct tskinfo` as:

```c
struct tskinfo {
    pid_t pid;              /* process id */
    pid_t tgid;             /* thread group id */
    pid_t parent_pid;       /* process id of parent */
    int level;              /* level of this process in the subtree */
    char comm[16];          /* name of program executed */
    unsigned long userpc;   /* pc/ip when task returns to user mode */
    unsigned long kernelpc; /* pc/ip when task is run by schedule() */
};
```

You should put this definition in `include/uapi/linux/tskinfo.h` as part of your solution. Note that this path is relative to the root directory of your kernel source tree. The `uapi/` directory contains the user space API of the kernel, which should include structures used as arguments for system calls. As an example, you may find it helpful to look at how `struct tms` is defined and used in the kernel. This is another structure that is part of the user space API and is used for getting process times.

To ensure that user space programs have access to your updated set of header files, you must do the following in the root directory of your kernel source tree:

```
$ sudo make headers_install INSTALL_HDR_PATH=/usr
```

This copies the UAPI header files to `/usr/`. You should now be able to see your new header file as `/usr/include/linux/tskinfo.h`. You should do this every time you update a header file that is part of the user space API of the kernel.

**Description of parameters**

* **buf** points to a buffer to store the thread data from the process tree. The thread data stored inside the buffer should be sorted first according to the associated main process in BFS order, where processes at a higher level (level 0 is considered to be higher than level 10) should appear before processes at a lower level. Then, when listing threads within a process's thread group, the threads should be stored in ascending order of PID.
* **nr** points to an integer that represents the size of this buffer (number of entries). The system call copies at most `*nr` entries of thread data to the buffer, and stores the number of entries actually copied in `*nr`. Note that the actual size of the buffer will be `*nr * sizeof(struct tskinfo)` bytes.
* **root_pid** represents the PID of the thread that will serve as the root of the subtree you are required to traverse. See the hint below on `PID vs TGID` for more information on what we mean when we say the PID of a thread. Information of nodes outside this subtree shouldn't be put into `buf`. **If the thread with PID `root_pid` is part of a thread group with multiple threads, they (and all of their children) should be included in `buf`, unless the maximum number of entries is reached.** Note that this means you might end up including threads whose PID is smaller than `root_pid`, as long as these threads are in the same thread group as the thread with PID `root_pid`.
* **Return value:** The function defining your system call should return 0 on success, and return an appropriate error on failure such that `errno` contains the correct error code.

**Additional requirements**

* You should fill the buffer when traversing the tree primarily in **BFS order** of process (main thread), then for each thread within a process in **ascending order of PID** until either every thread is stored, or the number of threads reaches `nr`. E.g. If we have a tree as below:

	```
	       0
	     /   \
	    1,2   3,4,5
	   /   \   / \
	  6     7 8   9
	```
                
	and you are given a buffer where `*nr` is `10`, the buffer should be filled as follows (excluding program counters):

	```
	[
	tskinfo {pid-0, tgid-0, ppid-0, level-0, … },
	tskinfo {pid-1, tgid-1, ppid-0, level-1, … },
	tskinfo {pid-2, tgid-1, ppid-0, level-1, … },
	tskinfo {pid-3, tgid-3, ppid-0, level-1, … },
	tskinfo {pid-4, tgid-3, ppid-0, level-1, … },
	tskinfo {pid-5, tgid-3, ppid-0, level-1, … },
	tskinfo {pid-6, tgid-6, ppid-1, level-2, … },
	tskinfo {pid-7, tgid-7, ppid-2, level-2, … };
	tskinfo {pid-8, tgid-8, ppid-4, level-2, … };
	tskinfo {pid-9, tgid-9, ppid-4, level-2, … };
	]
	```

	Note that in this example, there are 7 processes (0, 1, 3, 6, 7, 8, and 9) and 3 threads (2, 4, and 5). Also, 7 is a child of 2 not 1, and 8 and 9 are children of 4. In this scenario, commands like `ps` might differ from ours in output (see the hint below on `PID vs TGID` for an explanation). Note also that the levels are zero-indexed, and level 0 refers to tasks at the same level as the task with PID `root_pid` (this might not always be the task with PID 0).

* There should be no duplicate information of processes inside the buffer.

* If a value to be set in `tskinfo` is accessible through a pointer which is null, set the value in `tskinfo` to 0.

* Your system call should be assigned the number **462** and be implemented in a file `ptree.c` in the `kernel/` directory, i.e. `kernel/ptree.c`. Note again that this path is relative to the root directory of your kernel source tree. You will need to modify the appropriate kernel `Makefile` so that it is aware of your new source code file, or it will not know to compile your source code file with the rest of the kernel code.

* Retrieving some of the information for your `struct tskinfo` will be architecture-specific (i.e. it will be implemented differently depending on whether your platform is x86-64 or ARM64). You should call generic functions from your main `kernel/ptree.c` file for retrieving these values, but implement them in `arch/[x86 or arm64]/kernel/ptree.c` (and make sure you modify the `Makefile` in the same folder appropriately). This ensures that only the correct retrieval function is included when your kernel is compiled. Although you are only required to complete solutions for one architecture, defining these functions in both `arch/x86` and `arch/arm64` will allow you to develop alongside students with a different platform. Note that you can put declarations for these functions in `include/linux/ptree.h`.

* Your algorithm shouldn't use recursion since the size of the function stack in the kernel is quite small, typically only **8KB**.

* Your code should handle errors that could occur. For example, some error numbers your system call should detect and return include:
	* `-EINVAL`: if `buf` or `nr` are null, or if the number of entries is less than 1.
	* `-EFAULT`: if `buf` or `nr` are outside the accessible address space.

**Hints**

* This syscall implementation has a few different components. We recommend that you approach the problem in steps, doing incremental development and testing to check the correctness of the functionality along the way:

	* To understand how to fill a `struct tskinfo` with the relevant information, you should first write a simple version of the system call which returns the information for one process only, given its PID.
	* Now expand the functionality to allow listing all/any process in the given subtree of the process tree (not including threads). This will help you understand how to implement BFS over the process tree without worrying about the nuances of listing threads versus processes.
	* Finally, write the system call that works for both processes and threads. Think carefully about how threads intersect with the process tree, and how you must modify your BFS algorithm to include them. You may find [this blog post][this-blog-post] helpful for relating processes and threads.

* Linux maintains a list of all processes in a doubly linked list. Each entry in this list is a `task_struct` defined in [include/linux/sched.h][include/linux/sched.h]. You may find these functions/macros (as well as others you run across) in the Linux kernel useful for translating between PIDs and `task_structs`, and walking the list of processes to find the `task_struct` corresponding to your root task:

	* `for_each_process` to walk through the list of processes and `for_each_thread` to walk through the list of threads
	* `task_pid_nr` to translate from a `task_struct` to its PID
	* `task_tgid_nr` to translate from a `task_struct` to its TGID

* When traversing the process tree data structures, it is necessary to prevent the data structures from changing to ensure consistency. For this purpose the kernel relies on RCU, which involves calling the `rcu_read_lock()` and `rcu_read_unlock()` primitives before you begin traversal and after the traversal is completed respectively. You do not need to worry about the details of how RCU works for now, but between the calls to `rcu_read_lock()` and `rcu_read_unlock()`, your code must not perform any operations that may block the executing process, such as memory allocation, copying of data into and out from the kernel etc. Your code should look something like:

	```c
	rcu_read_lock();
	do_some_work();
	rcu_read_unlock();
	```

	If your code needs to do memory allocation or copying of data into and out from the kernel, such code should be before `rcu_read_lock()` or after `rcu_read_unlock()`.

* You can use the `ps -eLf` command to display a list of current threads. However, when using this and other user commands that reference threads, you might notice that they list all the threads within a process as having the same "PID", and differentiate those threads with some other field (maybe called TID or LWP). This can be confusing, because this is **not** how the kernel represents threads internally. In the kernel, threads have unique PIDs, and use a thread group id (TGID) to identify which process the thread is associated with. So why do some user programs identify threads differently? It has to do with historical differences between how the POSIX standard defines threads/processes, and how the Linux kernel implements them. See [this stackoverflow answer][this-stackoverflow-answer] for more information on how these two representations differ. In your program, you should return the **Linux kernel representation** of PID, so each thread `tskinfo` struct you return should have a unique PID.

	* If you're interested, [this (optional) paper from 2002][this-(optional)-paper-from-2002] provides some historical context for the theoretical differences between POSIX and Linux when it comes to threads/processes. Note that it is quite old, so use it as historical context and not a reference for programming.

* To learn about system calls, you may find it helpful to search the Linux kernel for other system calls and see how they are defined. Take a look at at [`include/linux/syscalls.h`][include/linux/syscalls.h] and [`kernel/sys.c`][kernel/sys.c].

* Your system call should not blindly trust the arguments that are passed in, especially pointers. You should make use of functions such as `copy_from_user`, `copy_to_user`, `get_user`, and `put_user`.

* The kernel provides various data structures as discussed in your reading, especially Ch 6 of the Linux Kernel Development book. In addition to the kernel implementation of linked lists, you may also find the kernel implementation of queues helpful.

* Remember that the standard C library is not available in the kernel, but the kernel often provides similar functionality. For memory allocation, instead of malloc and free, you should use the [kernel memory allocation][kernel-memory-allocation] functions, `kmalloc` and `kfree`; the former requires a GFP flag, which should be `GFP_KERNEL` for normal memory allocation. For debugging purposes, instead of `printf`, you will find it helpful to use [printk][printk], a robust mechanism to print information that is designed to be callable from almost any C code in the kernel. However, do not leave extraneous debugging messages enabled in your code when submitting your homework.

[this-blog-post]: https://chengyihe.wordpress.com/2015/12/29/kernel-thread-and-thread-group/
[include/linux/sched.h]: https://elixir.bootlin.com/linux/v6.8/source/include/linux/sched.h#L737
[this-stackoverflow-answer]: https://stackoverflow.com/questions/9305992/if-threads-share-the-same-pid-how-can-they-be-identified/9306150#9306150
[this-(optional)-paper-from-2002]: https://www.kernel.org/doc/ols/2002/ols2002-pages-330-337.pdf
[include/linux/syscalls.h]: https://elixir.bootlin.com/linux/v6.8/source/include/linux/syscalls.h
[kernel/sys.c]: https://elixir.bootlin.com/linux/v6.8/source/kernel/sys.c
[kernel-memory-allocation]: https://www.kernel.org/doc/html/latest//core-api/memory-allocation.html
[printk]: https://www.kernel.org/doc/html/v5.13/core-api/printk-basics.html


Part 3: Test your new system call
------

**General description**

Write a simple C program which calls `ptree`. The program should be able to take in a single command line argument and use it as `root_pid`. If no argument is provided, your program should return the entire process tree. The program should be in the `user/part3/` folder of your team repo, and your makefile should generate an executable named **`test`**.

Since you do not know the tree size in advance, you should start with some reasonable buffer size for calling `ptree`, then if the buffer size is not sufficient for storing the tree, repeatedly double the buffer size and call `ptree` until you have captured the full process tree requested. Print the contents of the buffer from index 0 to the end. For each process, you must use the following format for program output:

```c
printf("%s,%d,%d,%d,%p,%p,%d\n", buf[i].comm, buf[i].pid, buf[i].tgid,
    buf[i].parent_pid, (void *)buf[i].userpc, (void *)buf[i].kernelpc, buf[i].level);
```

Example program output (yours will likely be different depending on the processes running in your VM):

```
$ ./test
swapper/0,0,0,0,(nil),0xffff8000815be794,0
systemd,1,1,0,0xfbe1d5d2bd20,0xffff8000815be794,1
kthreadd,2,2,0,(nil),0xffff8000815be794,1
systemd-journal,362,362,1,0xf19f7d72bd74,0xffff8000815be794,2
systemd-udevd,408,408,1,0xfdf0f485bd20,0xffff8000815be794,2
...
bash,3073,3073,2989,0xedbbabba7a70,0xffff8000815be794,11
test,3239,3239,3073,0xe1d2a6e496a8,0xffff8000815be794,12
```

**Hints**

* The `ps` command in the VM will help in verifying the accuracy of information printed by your program. In particular, you can use `ps -eLf`(however, see the note from part 2 about why this returns a "PID" for threads that is different from the kernel’s idea of a PID).
* Although system calls are generally accessed through a library (libc), your test program should access your system call directly. This is accomplished by utilizing the generall purpose `syscall(2)` system call. You can consult its man page for more details: `man 2 syscall`.

Part 4: Investigate kernel source code
------

Write answers to the following questions in the `user/part4.txt` text file, following the provided template **exactly**. Make sure to include any references you use in your `references.txt` file.

> <span style="color:red">IMPORTANT: The skeleton code pushed to the assignment repo has an error in question two. The original skeleton code asks about the init process. The question should be about the **PROCESS WITH PID 0**, as is the case below.</span>

1. There are a few PIDs that are reserved for system processes and kernel threads. These include PIDs 0, 1, and 2. What is the process name associated with each of these three PIDs (some may have multiple acceptable names)?

2. Give the exact URL on https://elixir.bootlin.com/linux/v6.8/source pointing to the file and line number at which the data structure describing the process with PID 0 is defined. Note: make sure you use v6.8.

3. Give the exact URL on https://elixir.bootlin.com/linux/v6.8/source pointing to the file and line number at which the function that executes instructions to context-switch from one task to another is defined. Please provide an answer for both arm64 and x86-64. The function you identify, which may be in assembly code, should be the one that contains the actual instruction that switches the CPU's program counter register to the task so it can run. Note: use v6.8.

4. Give the exact URL on https://elixir.bootlin.com/linux/v6.8/source pointing to the file and line number at which the process with PID 1 starts running as the currently running process. Please provide an answer for both arm64 and x86-64. Note: use v6.8.

For reference, the URLs you answer with should be in the following format: https://elixir.bootlin.com/linux/v6.8/source/kernel/sched/core.c#L6607

[elixir.bootlin.com]: https://elixir.bootlin.com/linux/v6.8/source

Part 5: Create your own process tree
------

Write another program `foo` that creates processes and/or threads corresponding to the following process tree:

```
         5000,5001
          / \
       5002 5003 
        |     |
       5004 5005
```

In other words, with `foo` running in another shell, you should be able to use the program you wrote in part 3 to print the following console output:

```
$ ./test 5000
foo,5000,5000,1,x,y,0
foo,5001,5000,1,x,y,0
foo,5002,5002,5000,x,y,1
foo,5003,5003,5000,x,y,1
foo,5004,5004,5002,x,y,2
foo,5005,5005,5003,x,y,2
```

where `x` represents the `userpc`, and `x` represents the `kernelpc`. Any valid values of `x` and `y` are okay. Otherwise, all of the other fields shown in the output above should **exactly** match the strings and integers shown. In particular, the PID etc **must** match.

This program should be in the `user/part5/` directory of your team repo, and your `Makefile` should generate the `foo` executable. While we will be testing your code on a freshly booted system, you may find it helpful to [change the maximum possible PID value][change-the-maximum-possible-pid-value] to make it easier to test your program (this allows PID values to rollover more quickly). For example, using the following commands may be helpful: `echo 10000 | sudo tee /proc/sys/kernel/pid_max`

**Hints**

* To create threads, you should use the pthreads [pthread_create()][pthread_create()] call.

* As with part 2, it may be helpful to break the problem into pieces:

    1. First, write a program that does not involve creating threads to better understand when/where to fork. That program should produce the following process tree:

        ```
            5000
            /  \
          5001 5002
           |    |
          5003 5004
        ```

    2. Once you have this intermediate program, you should be able to make the target process tree easily by calling `pthread_create()` from a certain process.

[change-the-maximum-possible-pid-value]: https://stackoverflow.com/questions/6294133/maximum-pid-in-linux
[pthread_create()]: https://man7.org/linux/man-pages/man3/pthread_create.3.html

Submission Checklist
------
Include the following in your `main` branch. Only include source code (ie `*.c`,`*.h`) and text files, do **not** include compiled objects.

* `.armpls` file for teams with M1/M2/M3 CPUs
* `README` file
* `references.txt` file
* Implementation of `ptree` system call added to `linux/kernel/`, as well as changes to other parts of the kernel as necessary
* Implementation of `test.c` in `user/part3/`
* Answers to written questions in `user/part4.txt`
* Implementation of `foo.c` in `user/part5/`
