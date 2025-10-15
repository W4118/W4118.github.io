# HW3 (W4118 Fall 2025)

> <span style="color:red">**DUE: Friday 10/17/2025 at 11:59pm ET**</span>

## General instructions

All homework submissions are to be made via [Git][Git]. You must submit a detailed list of references as part of your homework submission indicating clearly what sources you referenced for each homework problem. You do not need to cite the course textbooks and instructional staff. All other sources must be cited. Please edit and include this [file][file] in the top-level directory of your homework submission in the `main` branch of your team repo. **Be aware that commits pushed after the deadline will not be considered.** Refer to the homework policy section on the [class website][class-web-site] for further details.

[Git]: https://git-scm.com/
[file]: https://www.cs.columbia.edu/~nieh/teaching/w4118/homeworks/references.txt
[class-web-site]: https://www.cs.columbia.edu/~nieh/teaching/w4118/

Group programming problems are to be done in your assigned groups. We will let you know when the Git repository for your group has been set up on GitHub. It can be cloned using the following command. Replace `teamN` with your team number, e.g. `team0`. You can find your group number [here](https://docs.google.com/spreadsheets/d/1V2ytwTpXo2rhVop9pYt46M74Bk4lOsXeB2vSqylVLMg/edit?gid=690031232#gid=690031232).

```
$ git clone git@github.com:W4118/f25-hmwk3-teamN.git
```

> **IMPORTANT**: You should clone the repository **directly in the terminal of your VM**, instead of cloning it on your local machine and then copying it into your VM. The filesystem in your VM is case-sensitive, but your local machine might use a case-insensitive filesystem (for instance, this is the default setting on macs). Cloning the repository to a case-insensitive filesystem might end up clobbering some kernel source code files. See [this post](https://unix.stackexchange.com/questions/753038/is-building-the-linux-kernel-on-a-case-insensitive-filesystem-possible) for some examples.

This repository will be accessible to all members of your team, and all team members are expected to make local commits and push changes or contributions to GitHub equally. You should become familiar with team-based shared repository Git commands such as [git-pull][git-pull], [git-merge][git-merge], [git-fetch][git-fetch]. For more information, see [this guide](../guides/git.md).

There should be at least five commits **per member** in the team's Git repository. The point is to make incremental changes and use an iterative development cycle. Follow the [Linux kernel coding style](https://www.kernel.org/doc/html/v6.14/process/coding-style.html). You **must** check your commits with the `run_checkpatch.sh` script provided as part of your team repository. Errors or warnings from the script in your submission will cause a deduction of points.

For students on Arm computers (e.g. macs with M1/M2/M3 CPUs): if you want your submission to be built/tested for Arm, you must create and submit a file called `.armpls` in the top-level directory of your repo; feel free to use the following one-liner:

```
$ cd "$(git rev-parse --show-toplevel)" && touch .armpls && git add .armpls && git commit -m "Arm pls"
```

You should do this first so that this file is present in any code you submit for grading.

For all programming problems, you should submit your source code as well as a single README file documenting your files and code for each part. Please do NOT submit kernel images. The README should explain any way in which your solution differs from what was assigned, and any assumptions you made. You are welcome to include a test run in your README showing how your system call works. **It should also state explicitly how each group member contributed to the submission and how much time each member spent on the homework.** The README should be placed in the top level directory of the main branch of your team repo (on the same level as the `linux/` and `user/` directories).

[git-pull]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-pull.html
[git-merge]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-merge.html
[git-fetch]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-fetch.html

## Assignment overview

As you have seen in homework assignment 2, the kernel maintains the state for each process and records that state in the `__state` field of the `task_struct` of the process. The state indicates whether the process is runnable or running (`TASK_RUNNING`), sleeping (`TASK_INTERRUPTIBLE`, `TASK_UNINTERRUPTIBLE`), stopped (`__TASK_STOPPED`), dead (`TASK_DEAD`), etc. When a process is dead, the `exit_state` field of the task_struct of the process indicates whether the process is zombied (`EXIT_ZOMBIE`) or really dead (`EXIT_DEAD`).

In this homework, you will be **tracing the state changes of processes, and recording them in a ring buffer.**

A ring buffer is a [circular buffer](https://en.wikipedia.org/wiki/Circular_buffer) that has no real end. It can be represented using a fixed size array that keeps wrapping around. After writing to the last array location, new entries will start writing at the beginning of the array again, overwriting any records that were previously there. You will write a system call to copy the contents of the ring buffer to user space so that a user program can dump the contents of the buffer. In addition, you will need to write a synchronization mechanism to copy the contents of the ring buffer to user space at the occurrence of some future event.

Your changes should include the following (note that all paths are relative to the root of your kernel source tree):

- A file `pstrace.c` in the `kernel/` directory, i.e. `kernel/pstrace.c`.
- A file `pstrace.h` in the kernel uapi include directory, i.e. `include/uapi/linux/pstrace.h`.
- A file `pstrace.h` in the kernel include directory, i.e. `include/linux/pstrace.h`.
- Additional changes to existing files in the kernel source code as necessary.

You should not change any existing kernel data structures. Almost all your kernel changes should be in `pstrace.c`. `pstrace.h` should only contain the necessary data structure and macro definitions. The only changes to existing files in the kernel source code should be those required for the system call table modifications and calls to record process state changes in various kernel code paths. You will also need to change the kernel `Makefiles`.

## Part 1: Trace a process's state changes in the ring buffer

**STEP 1. Set up system calls for enabling and disabling tracing**

In this part, we will set up the relevant system calls to enable/disable the tracing of a process’s state change. We only want to record the state changes of processes that have had tracing enabled by these system calls. The interface for these functions are:

```c
/*
 * Syscall No. 467
 * Enable the tracing for @pid. If -1 is given, trace all processes.
 */
long pstrace_enable(pid_t pid);

/*
 * Syscall No. 468
 * Disable tracing.
 */
long pstrace_disable();
```

In other words, your tracing system call will either trace a **single** process or trace **all** processes. You should use a suitable data structure to keep track of what processes are being traced. Note that this system call simply __enables/disables__ the tracing - the actual writing to the ring buffer will happen in a subsequent function.

**Tasks**

- Implement `pstrace_enable()` and `pstrace_disable()`.
- Modify any relevant files in the kernel to reflect the newly added system calls.
  
**Additional requirements**

- Tracing a process includes tracing all threads in the same thread group.
- If the pid does not exist, an appropriate error should be returned.
- Do not modify the `task_struct` (for the entire assignment).
- If tracing is already enabled, `pstrace_enable()` will **replace** the set of processes being traced with what is specified in the newer `pstrace_enable()` call.
- Calling `pstrace_disable()` before the first `pstrace_enable()` call should be a nop.
  

**STEP 2. Set up the ring buffer**

Once you can successfully indicate which processes should be traced, you should set up the structure of the ring buffer. Here are some data structures that are relevant for this part of the assignment:

```c
#define PSTRACE_BUF_SIZE 500	/* The maximum size of the ring buffer */

/* The data structure representing the global ring buffer */
struct pstrace_kernel {
    struct pstrace entry[PSTRACE_BUF_SIZE]; /* State changes */
    long counter;                           /* Number of records */

    /* Include other fields that you may need */
};

/* The data structure used to record a state change */
struct pstrace {
	int   cpu;		/* The CPU that this process is on when the state change is recorded */ 
	char comm[16];	/* Name of the process */
	long state;		/* NEW state of the process */
	pid_t pid;		/* PID of the process; i.e. returned by getpid() */
	pid_t tid;		/* TID of the thread; i.e. returned by gettid() */
};
```

Note that we want our entry to record the **NEW** state of the process. Note also that these data structures may need to be spread across multiple header files, depending on whether they're part of the user space API.

You are free to modify the `pstrace_kernel` structure, and you will indeed need to add additional fields to the structure. Note that in addition to the array for the ring buffer itself, `pstrace_kernel` has a counter, which should be initialized to zero and be incremented for each record that is added to the ring buffer. It is a **count of how many trace records have ever been added** to the ring buffer and **should never be reset**.

**Tasks**

- Define the provided data structures.
- Initialize the provided data structures as necessary.

**STEP 3. Recording one type of state change**

For now, we want to focus on tracing only one type of state change: when a process that is running becomes blocked but interruptible. This will result in the process state changing from `TASK_RUNNING` to `TASK_INTERRUPTIBLE`. When a process blocks in this manner, you should call `pstrace_add()` to create a new entry that records the process's updated state `TASK_INTERRUPTIBLE` in the `state` field of a `struct pstrace`, and add this new entry to the global ring buffer. The interface of the function is:

```c
/* Add a record of a state change into the ring buffer. */
void pstrace_add(struct task_struct *p, long stateoption);
```

`pstrace_add()` should only record the state change if the given process has tracing enabled. The `stateoption` argument is an additional argument passed to `pstrace_add()` that you can use as you see fit. For example, you could use the argument to pass in the state that you want recorded or any other information that would be helpful, but you do not have to use the argument for this part of the assignment. We will use this argument in later parts.

Since the ring buffer is shared by all CPUs, multiple state changes may occur simultaneously. You will need to define a lock to protect the ring buffer from race conditions, and use this lock in `pstrace_add()`.

Your job is to record actual changes in process state, not necessarily whenever the `__state` field is changed. For example, if the Linux code changes `__state` from `TASK_RUNNING` to `TASK_INTERRUPTIBLE` to `TASK_RUNNING` all without actually running another process, the process's state did not really change from `TASK_RUNNING`. A key part of this assignment is figuring out where a process's state actually changes and recording those events. You should carefully consider the discussion in class regarding the lifecycle of a process in Linux, including when a process actually starts and stops running.

**Hints**

You may want to look at the following file when thinking about where to add (a) call(s) to `pstrace_add()`:

- `kernel/sched/core.c`

**Tasks**

- Implement `pstrace_add()` to track when a process's state changes from `TASK_RUNNING` to `TASK_INTERRUPTIBLE`.
- Modify any relevant files in the kernel to record this process state change.

**STEP 4. Return a copy of ring buffer to user space**

At this point, we’ve allowed users to enable/disable the tracing of processes, and have recorded state changes from `TASK_RUNNING` to `TASK_INTERRUPTIBLE` into a buffer located in kernel space. Now, we want to be able to access these records in user space. Write a system call that can copy the records in the ring buffer to user space. The interface of the system call is:

```c
/*
 * Syscall No. 469
 *
 * Copy the pstrace ring buffer into @buf.
 * If @counter points to 0, copy the pstrace ring buffer.
 * If @counter points to a value < 0, return an error.
 *
 * Returns the number of records copied.
 */
long pstrace_get(struct pstrace *buf, long *counter); 
```

If `*counter == 0`, copy the current state of the pstrace global ring buffer into `@buf` and set `*counter` to the value of the buffer counter corresponding to the last record copied. If `*counter < 0`, it is invalid and so return an error. Later in the assignment, you will make use of the `counter` argument more extensively, but for now, just have your system call handle these two cases.

In addition, you should have another system call that clears the ring buffer.

```c
/*
 * Syscall No. 470
 *
 * Clear the pstrace buffer. Cleared records should
 * never be returned by `pstrace_get()`. This function
 * should NOT reset the value of the buffer counter.
 */
long pstrace_clear();
```

Cleared records should no longer be able to be returned by future `pstrace_get()` calls. Do not reset the ring buffer counter value.

**Tasks**

- Implement `pstrace_get()` and `pstrace_clear()`.
- You will be required to write a user test in Part 4, but it may be a good idea to test your current basic `pstrace` implementation. Make sure that you can record this one type of state change and return the buffer results to user space immediately when `pstrace_get()` is called, before moving on to the next sections.

## Part 2: Record additional state changes

Extend the usage of `pstrace_add()` to now record changes in processes’ state throughout the kernel. Specifically, changes to the following seven states should be traced and recorded in the kernel ring buffer:

1. `TASK_RUNNING`
2. `TASK_RUNNABLE`
3. `TASK_INTERRUPTIBLE`
4. `TASK_UNINTERRUPTIBLE`
5. `__TASK_STOPPED`
6. `EXIT_ZOMBIE`
7. `EXIT_DEAD`

Note that you should represent a task that is on the run queue as `TASK_RUNNABLE` while a task that is actually running on the CPU should be `TASK_RUNNING`. However, Linux does not have an actual state `TASK_RUNNABLE`. It denotes both the state of being on the run queue and actually running as `TASK_RUNNING`. 

To get around this, you should introduce a `TASK_RUNNABLE` state for tracing purposes only (i.e. `TASK_RUNNABLE` is not stored in the actual `__state` field of the `task_struct`). 

**You should define `TASK_RUNNABLE` to have a value of 3.** (See [here](https://elixir.bootlin.com/linux/v6.14/source/include/linux/sched.h#L99) for more details.) 

Note that since `TASK_RUNNABLE` is not an actual value stored in the `__state` field of the `task_struct`, you will find it useful to use the `stateoption` argument of `pstrace_add()` to record when a process's state changes to `TASK_RUNNABLE`.

Again, your job is to record actual changes in process state, not necessarily whenever the `__state` field is changed. You should identify where a process's state actually changes and record those events as opposed to trying to insert `pstrace_add()` calls whenever you find the `__state` field is being modified. 

**You should minimize the number of `pstrace_add()` calls required to trace the seven states.**

You will also need to carefully consider the code paths in which you insert `pstrace_add()` since the function does locking. Otherwise, you may deadlock your system. In particular, you should ask yourself whether the code path in which you insert `pstrace_add()` could be executed as a result of an interrupt and what implications that may have on the specific locking primitives you use.

**Tasks**

- Update `pstrace_add()` to now track all specified process state changes.
- Modify any relevant files in the kernel where these changes occur.

**Hints**

You may want to look at the following files when thinking about where to add (a) call(s) to `pstrace_add()`:

- `kernel/exit.c`
- `kernel/sched/core.c`


## Part 3: Waiting to copy the tracing buffer into user space

You will now update the implementation of your `pstrace_get()` system call to support waiting for the ring buffer counter to reach some value before copying the contents of the ring buffer to user space. The interface to `pstrace_get()` remains the same, but the system call should now provide new functionality for the case when the counter argument is positive:

```c
/*
 * Syscall No. 469
 *
 * Copy the pstrace ring buffer into @buf.
 * If @counter points to 0, copy the immediate pstrace ring buffer.
 * If @counter points to a value < 0, return an error.
 * If @counter points to a value > 0, the caller process will wait until
 *   a full buffer can be returned starting from record *counter
 *   (i.e. copy records numbering between *counter and
 *   *counter + PSTRACE_BUF_SIZE - 1 from the pstrace ring buffer to @buf).
 *
 * Returns the number of records copied.
 */
long pstrace_get(struct pstrace *buf, long *counter);
```
Maintain the functionality you programmed for `pstrace_get` in `Part 1`. That is, if `*counter == 0`, you must return the buffer with all valid entries in its current state immediately and set `*counter` to the value of the buffer counter corresponding to the last record copied.

Specifically, a positive value of `*counter` indicates a request for a full buffer starting at ring buffer counter `*counter`. That is, your system call should copy records into `@buf` in chronological order such that the first record is the record corresponding to ring buffer counter `*counter`, and the last record is the record corresponding to ring buffer counter `*counter + PSTRACE_BUF_SIZE - 1`.

If the ring buffer counter has not yet reached record `*counter + PSTRACE_BUF_SIZE - 1` (the last record requested), your system call should sleep until this happens and the full buffer can be returned. When you return, update `*counter` to the value of the ring buffer counter corresponding to the last record that was requested.

> For example, if `pstrace_get()` is called with `*counter = 1000`, it should not return until the ring buffer counter has reached 1499. When it returns, it should return the relevant corresponding buffer records from buffer counter 1000 to 1499, with `*counter` updated to 1499.

If the ring buffer counter has already exceeded record `*counter + PSTRACE_BUF_SIZE - 1` (the last record requested), you will only be able to partially fill the buffer. In particular, only copy relevant records within the requested range. You should NOT copy in any other records to fill the buffer.

> For example, if `pstrace_get()` is called with `*counter = 800`, then the last record that is copied over should be the record corresponding to the counter 1299. However, if the ring buffer counter is already at 1399, then it means that it only contains records corresponding to counters 900 to 1399. Therefore, you should only return 400 buffer records (corresponding to counters 900 to 1299) instead of a full buffer, since those are the only relevant records within the requested range that are available.

If all the records within the requested range have already been evicted from the ring buffer, that is if `*counter + PSTRACE_BUF_SIZE - 1` (the last requested record) is less than the earliest record left in the ring buffer, set `*counter` to be the current value of the ring buffer counter and return immediately. Do not copy any records from the ring buffer in this case.

> For example, if `pstrace_get()` is called with `*counter = 100` when the ring buffer counter is already at 1400, no records can be copied and `*counter` is updated to 1400.

**Hints**

- You will need to implement an appropriate synchronization mechanism to wake up waiting processes when their desired counter condition has been met. Waiting processes should be blocked with state `TASK_INTERRUPTIBLE`. **You may NOT let the system call spin when the counter condition has not been met.**
- You should plan to use [Linux wait queues](https://embetronicx.com/tutorials/linux/device-drivers/waitqueue-in-linux-device-driver-tutorial/). A wait queue is a list of processes waiting for some event to occur. The functions that use wait queues are designed to either wake up a single process or all processes on a wait queue. All the processes on a wait queue should be waiting for the same event. If the desired functionality is for processes to wait for different events to occur, the typical approach is to use a separate wait queue for each event. The functions that use wait queues are not designed to search a wait queue to identify processes waiting for different events to occur.
- You should think carefully about how and when you will copy the contents of the ring buffer in response to a `pstrace_get()` to provide the expected functionality. In particular, there may be an arbitrary amount of time that elapses between when a process is woken up and when the process is actually run to complete the system call and return the records to the calling process. Be sure to return the correct records requested when possible.
- You should also think carefully about what happens if you are tracing a process that is waiting on a `pstrace_get()` call, to ensure that you do not deadlock your system. How does that process wake up? What trace records are generated as a result of waking up that process? What locks are held when you are recording trace records?
- One possible edge-case is that you have a `pstrace_get` call waiting for a certain counter but the target process exits before that counter is reached. **You do not need to handle this case.**
- Also make sure that you do not deadlock your system when handling wait queues and interrupts.
  
**Additional requirements**

- If the buffer provided for copying into is not large enough to hold the records requested, you should return an error.
- A process that is sent a signal while it is waiting on `pstrace_get()` should be woken up and return an appropriate error.
- `pstrace_clear()`should also wake up all processes waiting on `pstrace_get()`. The processes that are woken up should copy any relevant records currently in the buffer and return as opposed to waiting for their respective buffer full conditions to be met. `*counter` should still remain as the value of the last record that was originally requested.

**Tasks**

- Update `pstrace_get()` to support positive values of `*counter`.
- Update `pstrace_clear()` to wake up any waiting processes.

## Part 4: Test your pstrace

Write a program named **`test`** that calls the the `pstrace_get()` function repeatedly to return the records in the buffer over time. Show how you can use the counter value so that successive calls to `pstrace_get()` return a chronological ordering of all records. The program should be in the `user/part4/` directory of your team repo.

For testing purposes, you should also write another program named **`seven_states`** that changes its states between running and sleeping a certain number of times and then exits. Use your `test` program to trace the process of this second program. This should be done by modifying `test` so that it forks a child process that execs the second program. In other words, we should be able to see the results of your second program by simply doing `./test`, **without** needing to start your `seven_states` program manually in another terminal.

**Your `Makefile` must have a distinct rule to build `test`, and a distinct rule to build `seven_states`.** In other words, we should be able to call `make test` and `make seven_states` separately to compile each of the executables. **Calling `make` without any specified target should compile both executables.** In other words, we should be able to see all your results by simply doing `make && ./test`.

You should be able to observe how the second program turns from running to sleeping and finally, to zombie, and exits. Your testing should generate at least one record for each of the seven distinct process states we have asked you to record, and you should include the resulting output in your submission in a file `user/part4/pstrace_output.txt`.

> As was the case in HW2, you will have to run `sudo make headers_install INSTALL_HDR_PATH=/usr` in the root of your kernel source tree to make your new kernel header files available to user space programs.



We have provided a sample program to showcase a traced process in `user/part4/sample_programs/` for you to test the progress of homework 3. You may use this as a helper to see how your process behaves in comparison to the output.txt file and also as an aid to develop your testing program. Note that it is possible to have a different output despite a correct solution as the output of your team's run is dependent on when exactly the traces begins, but you may still use this program to get an idea of whether you are correctly reporting your states.

**Importantly, the output for your programs in this part must follow the same format as the sample output we provided.**


**Hints**
- Consider how you may force a task to visit all states. Disk I/O is an example of a task going into the `TASK_UNINTERRUPTIBLE` state, but you must ensure that the target data for `read` is not cached in memory. For this, you may consider ([clearing memory caches](https://linuxconfig.org/clear-cache-on-linux)).


## Part 5: Process lifecycle

Write answers to the following questions in the `user/part5.txt` text file, following the provided template **exactly**. Make sure to include any references you use in your `references.txt file`.

For question 1, make sure that you are referencing the correct kernel version in bootlin (**[v6.14](https://elixir.bootlin.com/linux/v6.14/source)**). For reference, the URLs you answer with should be in the following format: https://elixir.bootlin.com/linux/v6.14/source/kernel/sched/core.c#L1234

> **IMPORTANT**: For each of the URLs you give, make sure the line numbers correspond to the line numbers on bootlin. In other words, they should be the line numbers **BEFORE** you made any changes to the kernel source code.

For questions, 2, 3, and 4, give your answer in the format `[STATE] -> [STATE]`, replacing `STATE` with the relevant state. For example: `[TASK_RUNNING] -> [TASK_INTERRUPTIBLE]`. Do NOT remove the brackets around the state names.

1. Give bootlin URLs showing the file and line number at which you inserted a `pstrace_add()` call to trace the state change to:

    - `TASK_RUNNING`
    - `TASK_RUNNABLE`
    - `TASK_INTERRUPTIBLE`
    - `TASK_UNINTERRUPTIBLE`
    - `__TASK_STOPPED`
    - `EXIT_ZOMBIE`
    - `EXIT_DEAD`

2. Which state change(s) can be directly caused by interrupts (i.e. an interrupt handler will change the `__state` field)?

3. Which state change(s) can only be caused by the running process itself (i.e. `current` will change its own `__state` field)?

4. Which state change(s) always involves or results in the current running process calling `schedule()`?

5. There are 3 tasks with respective pid values 1150, 1152, and 1154. They are run in the following order on a CPU: pid 1150, 1152, 1154, 1152, 1150, 1154. In `__schedule()`, there are two variables, `prev` and `next`. Each of these variables is a pointer to a `struct task_struct`.

    - When each task starts running, what will be the respective PIDs for `prev` and `next`? For any `prev` or `next` that cannot be known from the information provided, indicate your answer as `UNKNOWN`.
    - When each task stops running, what will be the respective PIDs for `prev` and `next`? For any `prev` or `next` that cannot be known from the information provided, indicate your answer as `UNKNOWN`.

## Submission checklist

Include the following in your `main` branch. Only include source code (ie `*.c`, `*.h`) and text files. Do not include compiled objects.

- `.armpls` file for teams with Arm CPUs
- `README` file
- `references.txt` file
- Implementation of system calls and additional functions added to `linux/kernel/pstrace.c`
- Appropriate header files, in particular `linux/include/uapi/linux/pstrace.h` and `linux/include/linux/pstrace.h`
- Other changes to kernel code as necessary
- Implementation of `test.c` in `user/part4/`
- Output of `test.c` in `user/part4/pstrace_output.txt`
- Answers to written questions in `user/part5.txt`
