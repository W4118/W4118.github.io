# W4118 Fall 2025 - Homework 4

> <span style="color:red">**DUE: Wednesday 11/8/2025 at 11:59pm ET**</span>

## General Instructions

All homework submissions are to be made via [Git][Git]. You must submit a detailed list of references as part of your homework submission indicating clearly what sources you referenced for each homework problem. You do not need to cite the course textbooks and instructional staff. All other sources must be cited. Please edit and include this [file][file] in the top-level directory of your homework submission in the `main` branch of your team repo. **Be aware that commits pushed after the deadline will not be considered.** Refer to the homework policy section on the [class website][class-web-site] for further details.

[Git]: https://git-scm.com/
[file]: https://www.cs.columbia.edu/~nieh/teaching/w4118/homeworks/references.txt
[class-web-site]: https://www.cs.columbia.edu/~nieh/teaching/w4118/

Group programming problems are to be done in your assigned groups. We will let you know when the Git repository for your group has been set up on GitHub. It can be cloned using the following command. Replace `teamN` with your team number, e.g. `team0`. You can find your group number [here](https://docs.google.com/spreadsheets/d/1sWSEjtqSqSsl2dJ3R3YZKYlw4WG4GtfpHNECpqFJiks/edit?gid=260033287#gid=260033287).

```
$ git clone git@github.com:W4118/f24-hmwk4-teamN.git
```

> **IMPORTANT**: You should clone the repository **directly in the terminal of your VM**, instead of cloning it on your local machine and then copying it into your VM. The filesystem in your VM is case-sensitive, but your local machine might use a case-insensitive filesystem (for instance, this is the default setting on macs). Cloning the repository to a case-insensitive filesystem might end up clobbering some kernel source code files. See [this post](https://unix.stackexchange.com/questions/753038/is-building-the-linux-kernel-on-a-case-insensitive-filesystem-possible) for some examples.

This repository will be accessible to all members of your team, and all team members are expected to make local commits and push changes or contributions to GitHub equally. You should become familiar with team-based shared repository Git commands such as [git-pull][git-pull], [git-merge][git-merge], [git-fetch][git-fetch]. For more information, see [this guide](../guides/git.md).

There should be at least five commits **per member** in the team's Git repository. The point is to make incremental changes and use an iterative development cycle. Follow the [Linux kernel coding style](https://www.kernel.org/doc/html/v6.8/process/coding-style.html). You **must** check your commits with the `run_checkpatch.sh` script provided as part of your team repository. Errors from the script in your submission will cause a deduction of points. (Note that the script only checks the changes up to your latest commit. Changes in the working tree or staging area will not be checked.)

For students on Arm computers (e.g. macs with M1/M2/M3 CPU): if you want your submission to be built/tested for Arm, you must create and submit a file called `.armpls` in the top-level directory of your repo; feel free to use the following one-liner:

```
$ cd "$(git rev-parse --show-toplevel)" && touch .armpls && git add .armpls && git commit -m "Arm pls"
```

You should do this first so that this file is present in any code you submit for grading.

For all programming problems, you should submit your source code as well as a single README file documenting your files and code for each part. Please do NOT submit kernel images. The README should explain any way in which your solution differs from what was assigned, and any assumptions you made. You are welcome to include a test run in your README showing how your system call works. **It should also state explicitly how each group member contributed to the submission and how much time each member spent on the homework.** The README should be placed in the top level directory of the main branch of your team repo (on the same level as the `linux/` and `user/` directories).

[git-pull]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-pull.html
[git-merge]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-merge.html
[git-fetch]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-fetch.html

### Programming Problems:

Having completed the first half of your operating systems training, you and your team are now well-versed OS developers, and have been hired by W4118 Inc. to work on their next-generation operating systems. In this next-gen OS, W4118 Inc. has tasked your team to improve the Linux kernel's scheduling capabilities and optimize for workloads typical of W4118 Inc.'s customers.

Data privacy is important in modern computing, so instead of real customer programs, W4118 Inc. has provided you with a way to simulate the typical workloads submitted by end users using a Fibonacci calculator. The following links provide two task set workloads, each consisting of a list of jobs, with each line representing the N-th Fibonacci number to compute:

1.  [TASK SET 1](https://www.cs.columbia.edu/~nieh/teaching/w4118_f24/homeworks/taskset1.txt) <!-- TODO: update these links to f25 -->
2.  [TASK SET 2](https://www.cs.columbia.edu/~nieh/teaching/w4118_f24/homeworks/taskset2.txt)

**Note:** The Fibonacci calculator has been provided as user/fibonacci.c. It uses an inefficient algorithm by design to produce differing job lengths. You can modify the file, but **do not modify the `fib` function**.

Given these two task sets, W4118 Inc. is interested in a scheduler (let's call it "Oven") that can optimize for the **completion time** of the tasks. Specifically, the scheduler should provide the minimum average completion time for each task set across all tasks in the respective task set.


Part 1: Measure scheduler performance with eBPF
------
	
To see how well a scheduler performs, it is necessary to first have a way to measure performance, specifically completion time of tasks. W4118 Inc. is looking for a way to trace scheduling events and determine how well a scheduler functions by measuring tasks' run queue latency and total duration from start to finish.

Extended Berkeley Packet Filter ([eBPF](https://ebpf.io/)) is a powerful feature of the Linux kernel that allows programs to inject code into the kernel at runtime to gather metrics and observe kernel state. You will use eBPF to trace scheduler events. Tracepoints for these scheduler events are already available in the scheduler (for example, you can search in core.c for trace\_sched\_), so your job is to write code that will be injected into the kernel to use them. You should not need to modify any kernel source code files for this first part of the assignment.

bpftrace is a Linux tool that allows using eBPF with a high level, script-like language. Install it on your VM with:

```
sudo apt install bpftrace
```


Instead of searching through source code, you can run bpftrace -l to see what available tracepoints there are in the kernel.

Write a bpftrace script named trace.bt to trace how much time a task spends on the run queue, and how much time has elapsed until the task completed. Your profiler should run until stopped with Ctrl-C. While tracing, your script should, at tasks' exits, print out a table of the task name, task PID, milliseconds spent on the run queue (not including time spent running), and milliseconds until task completion. Note that task PID here is the actual pid field in a task\_struct, not what is returned by getpid. The output should be comma-delimited and follows the format:

```
COMM,PID,RUNQ_MS,TOTAL_MS
myprogram1,5000,2,452
myprogram2,5001,0,15
...
```


Ensure that the output of your eBPF script is synchronous. That is, it should print each line immediately after a task completes, and not when Ctrl-C is sent to the eBPF script. You should also only print traces from processes that have started after your script begins running. You should trace all such process and perform no additional filtering.

Test your script by running sudo bpftrace trace.bt in one terminal. In a separate terminal, run a few commands and observe the trace metrics in your first terminal. Experiment with different task sizes. Submit your eBPF script as user/trace.bt.

Now that you have an eBPF measurement tool, use it to measure the completion times for the the two task set workloads when running on the default CFS scheduler in Linux. What is the resulting average completion time for each workload? Write the average completion times of both workloads in your README. You may find this [shell script](https://www.cs.columbia.edu/~nieh/teaching/w4118_f24/homeworks/run_tasks.sh) helpful for running your tasks. You should perform your measurements for two different VM configurations, a single CPU VM and a four CPU VM. We are using the term CPU here to mean a core, so if your hardware has a single CPU but four cores, that should be sufficient for running a four CPU VM. If your hardware does not support four cores, you may instead run with a two CPU VM. Specify in your README the number of CPUs used with your VM for your measurements.

**Hint:** You may find it useful to reference the bpftrace [GitHub project](https://github.com/iovisor/bpftrace), which contains a manual and a list of sample scripts. A useful example script to start with is [runqlat.bt](https://github.com/bpftrace/bpftrace/blob/aa041d9d85f9ec11235c39fdcb5833412ec27083/tools/runqlat.bt). <!-- TODO: make sure this is the right version of bpftrace -->

**Note:** Since your profiler will run on the same machine as the test workloads, you will need to ensure that the profiler gets sufficient CPU time to record data. What could you do to the profiler's scheduling class and priority to ensure it can run over the test workloads? You may find [`chrt`](https://man7.org/linux/man-pages/man1/chrt.1.html) helpful.

**Note:** It is important to ensure that your trace.bt script is correct so that you are able to rely on the measurements produced by the script when comparing your custom scheduler to the default CFS scheduler. 

### Deliverables
*   trace.bt script in user/trace.bt
*   Average completion times for both tasksets on a single-CPU VM and on a VM configured with either two or four CPUs, using the Oven scheduling policy, in your README 
    * Make sure to specify whether you are using a two or four-CPU configuration for your multi-CPU measurement in your README

Oven Overview
------
	
W4118 Inc. has tasked you with creating a new scheduling policy that provides better average completion time than the default Linux scheduling policy. You should begin by implement a new scheduler policy. Call this policy OVEN. For this stage, the scheduler should work as follows:

1.  Only tasks whose policy is set to SCHED\_OVEN should be considered for selection by your new scheduler.
2.  Every task will have an assigned weight. Weights should be configured when a process calls the sched\_setscheduler system call, and passed as the `sched_priority` field of `struct sched_param`.
3.  By default, tasks have a weight set to a MAX\_WEIGHT of 10000. Tasks with a weight equal to the MAX\_WEIGHT of 10000 should use an unweighted round-robin scheduling algorithm. Each such task should get a time quantum of 1 tick.
4.  Tasks with a weight less than 10000 should be prioritzed over weights equal to MAX\_WEIGHT, and use your choice of scheduling algorithm and time quantum to optimize the overall average completion time metric for running the Fibonacci workload. Essentially, your scheduling class defaults to round-robin but should have a special mode that is optimized for running the Fibonnaci workload, where the optimized scheduling algorithm is up to you to decide.
5.  Tasks using the SCHED\_OVEN policy should take priority over tasks using the SCHED\_NORMAL policy, but _not_ over tasks using the SCHED\_RR or SCHED\_FIFO policies.
6.  The new scheduling policy should operate alongside the existing Linux schedulers. The value of SCHED\_OVEN should be 8.
7.  You should keep track of runtime statistics so commands like [top](https://man7.org/linux/man-pages/man1/top.1.html) work with your scheduling class.

The following sections provide more information on implementing a new scheduler. While there is no strict sequence of steps for implementing a scheduler, you might find it helpful to begin with Part 2 below. Note that it is equally valid to complete Part 3 and Part 4 in either order, meaning you are welcome to complete Part 4 before Part 3 and vice versa. 

Part 2: Create your scheduler / Adding a new scheduler
------
Background readings:
*   [Sched class methods](https://mgouzenko.github.io/jekyll/update/2016/11/24/understanding-sched-class.html)
*   [Medium Article](https://deepdives.medium.com/digging-into-linux-scheduler-47a32ad5a0a8)
*   [Scheduler deep dive](https://helix979.github.io/jkoo/post/os-scheduler/)

**Note:**  some of the information in the links may be outdated (e.g. the second source claims the scheduling classes have ".next" pointers to the scheduling class of next highest precedence, which is no longer accurate), so while they are helpful as a reference, your best "source of truth" is still the 6.14 kernel.

Implementation:
*   The Linux scheduler implements individual scheduling classes corresponding to different scheduling policies. For this assignment, you need to create a new scheduling class, oven\_sched\_class, for the OVEN policy, and implement the necessary functions in kernel/sched/oven.c.
*   A good starting point is defining the essential data structures and macro(s) that you would need for Oven. 
	* Implementing a scheduler and getting everything right is not easy. You should make your implementation as simple as possible. By the same token, you might find it helpful to first implement the case of the unweighted round-robin scheduler before introducing special, optimizied mode for the Fibonacci workload (see the 'Optimizing Oven' section for tips on implementing this mode).
	* Avoid using complex data structures that may provide better theoretical runtime complexity but are harder to implement and debug. In general, lists are fine to use. 
	* You should pay careful attention to how scheduling classes initialize their class-specific run queues and scheduling entities to insert them onto the run queues in kernel/sched/core.c; incorrect initialization can easily cause your system to hang. To identify which parts need to be changed, look for references to existing Linux schedulers, particularly in the files mentioned below. Pay special attention to areas that explicitly reference the current default CFS scheduler (fair\_sched\_class).
	* **Note** while the spec and the Linux kernel refer to default scheduler as the Completely Fair Scheduler (CFS), as of kernel version 6.6 the CFS was merged with the Earliest Virtual Deadline First (EEVDF) scheduler. If you're interested you can read more about these changes [here.](https://lwn.net/Articles/969062/)
*   For implementing the oven\_sched\_class itself, a good place to start is the simple scheduling class for idle tasks, listed in kernel/sched/idle.c. This is the simplest of the scheduling classes. In particular, the functions implemented by this scheduling class are a good indication of the minimum set of functions you need to implement to have an operational scheduling class.
*   Whether it is the scheduler's data structures or data structures, it is helpful to reference additional examples of scheduling classes, such as kernel/sched/rt.c and kernel/sched/fair.c. You may find the former particularly useful because it has its own version of a round-robin scheduler, SCHED\_RR, though you will likely find many parts of the code too complex to use directly for your own scheduling class.
*   Other interesting files that will help you understand how the Linux scheduler works are kernel/sched/core.c, kernel/sched/sched.h, include/linux/sched.h, and include/asm-generic/vmlinux.lds.h. 
	* While there is a fair amount of code in these files, a key goal of this assignment is for you to understand how to abstract the scheduler code so that you learn in detail the parts of the scheduler that are crucial for this assignment and ignore the parts that are not. 
	* You may find it helpful to specifically focus on sections that involve the existing schedulers you are referencing for your implementation. 
*   Although Linux in theory allows you to define new scheduling classes, Linux makes assumptions in various places that the only scheduling classes are the ones that have already been defined. 
	* Carefully review the code in kernel/sched/core.c that makes assumptions about the prioritization of scheduling classes, including sched\_init and \_\_pick\_next\_task. 
	* In order to assign tasks to your scheduling class and and allow your scheduling class to be used to pick the next task to run, other functions may also need to be changed, such as those called by sched\_setscheduler.
*   You may want your scheduler to have a minimum valid weight well above the priority range normally used for `sched_priority` such that any weight assignment less than the minimum valid weight is instead forced to be the MAX\_WEIGHT. The reason for this is that there are system programs that use sched\_setscheduler with lower values for `sched_priority`. If they are scheduled using your scheduling class, you want to detect their invalid weight assignments and schedule them using your round-robin algorithm.

Hints:
*   Check out the [debugging tips](#debugging-tips) provided below.
*   While developing and debugging your initial scheduling class implementation, it will be helpful to initially have SCHED\_NORMAL take priority over your SCHED\_OVEN policy so that bugs in your scheduling class are less likely to prevent other tasks from running. Once you have your scheduling class working, you can then switch the priority of these policies as required by the homework.
*   To verify that your oven\_sched\_class functions are actually invoked, you can add pr_info()'s to your functions and ensure that the prints appear when tasks are run using your scheduler. However, you should not include these prints in your final submission. Excessive logging could also prevent your kernel to boot as the default scheduler. 
*   Test your scheduler on a few runs of fibonacci and ensure it works before moving on. Set the weight to something other than the default and verify its behavior. After verifying that Fibonacci works, try running the [program from homework 3 that tests state changes](https://github.com/W4118/f24-hmwk3-sol/blob/main/user/part4/seven_states.c). The latter will exercise your scheduling code more throughly as it will involve scheduling processes that have a greater variety of state changes.

### Deliverables
*   Implementation of your scheduler in linux/kernel/sched/oven.c
*   Modifications to the Linux source code 

Part 3: Oven as the Default Scheduler
------
### Oven as the default scheduler 
Once your scheduler works for both fibonacci and the state program from homework 3, set it to be the default scheduler of your system. You will need to change various places in the kernel code to allow your kernel to boot with your scheduling class as the default. 
*   Consider how the scheduling policy for the `init_task` is assigned and how a task is assigned to a scheduling class by default. <!-- TODO:is saying the init task giving away too much here -->
*   For a more responsive system, you may want to set the scheduler of kernel threads to be SCHED\_OVEN as well (otherwise, SCHED\_OVEN tasks can starve the SCHED\_NORMAL tasks to a degree). To do this, you can modify kernel/kthread.c and replace SCHED\_NORMAL with SCHED\_OVEN. It is strongly suggested that you do this to ensure that your VM is responsive enough for the test cases, but you should not do this until you are certain your scheduler works properly.

Hints:
*   Check out the [debugging tips](#debugging-tips) provided below.
*   At this point, excessive pr_info() logging may prevent your kernel from booting with Oven as the default scheduler. Hence, you may want to limit the pr_info() logs in your oven\_sched\_class functions by only printing for a limited selection of tasks using if statements.
*   One way to see the scheduling policy number for tasks is by using the following ps command. However, note that just because the policy number is 8 does not mean that the task is necessarily scheduled using Oven– you can confirm that tasks are scheduled using Oven through pr_info() logs. 
```
ps -e --forest -o sched,policy,psr,pcpu,c,pid,user,cmd
```

### Deliverables
*   Modifications to the Linux source code 

Part 4: Optimizing and measuring your new scheduler
------
	
Recall your average completion time measurements from part 1, which used the CFS scheduler. Can you do better?

You should consider how you might modify the fibonacci program to set its OVEN weight. How did you determine when jobs of different weights will be scheduled in relation to each other? What weights and scheduling algorithm will optimize for average completion time? Note that this assignment asks you to optimize your scheduler specifically for the Fibonacci workload. Hence, it may be helpful to first consider what makes the Fibonacci workload unique and how those characteristics might guide your scheduler design. 

Make any changes to fibonacci and your scheduler, then submit eBPF traces of the two task sets running on your scheduler. You should use the same two VM CPU configurations you used earlier to measure performance using CFS. Submit your eBPF traces for the two task sets and CPU configurations as user/taskset1\_average.txt, user/taskset1\_average\_smp.txt, user/taskset2\_average.txt, and user/taskset2\_average\_smp.txt Write the average completion times for each workload in your README and compare against CFS.

**Hint:** Configuring the scheduling class and priority within the Fibonacci program will be too late (why is this the case?). You may want to use chrt while launching your Fibonacci tasks or write a small program to set the scheduler and weight before calling exec on Fibonacci.

**Note:** When launching jobs from the task list, start tasks in the order they are listed, but do not wait for tasks to finish before launching the next one. Also, for all trace submissions in this section, filter out any process that is not fibonacci.

### Deliverables
*   Changes to fibonacci.c
*   Modifications to the Linux source code 
*   user/taskset1\_average.txt, user/taskset1\_average\_smp.txt, user/taskset2\_average.txt, and user/taskset2\_average\_smp.txt
*   Average completion times for both tasksets on a single-CPU VM and on a VM configured with either two or four CPUs, using the Oven scheduling policy, in your README
*   Comparison between the Oven average completion time and CFS average completion time in your README 
	
Part 5: Add load-balancing features
------
	
So far, your scheduler will run a task only on the CPU that it was originally assigned. Let's change that now! For this part you will be implementing idle balancing, which is when a CPU will attempt to steal a task from another CPU when it doesn't have any tasks to run (i.e. when its runqueue is empty).

Load balancing is a key feature of the default Linux scheduler (Completely Fair Scheduler). While CFS follows a rather complicated policy to balance tasks and works to move more than one task in a single go, your scheduler will have a simple policy in this regard.

Idle balancing works as follows:

*   Idle balance is when an idle CPU (i.e. a CPU with an empty runqueue) pulls one task from another CPU.
*   Take a look at the CFS implementation to figure out where this is taking place and how it is called from kernel/sched/core.c.
*   The CPU that you pull a task from should have at least two tasks on its runqueue. Which task you pull off is up to your scheduling algorithm.
*   You should again ensure that the task you are stealing is eligible to be moved and allowed to run on the idle CPU.

Once you add idle load balancing, repeat the performance tests you conducted in the previous part and make notes on any observations. Again, be sure to include actual data. Submit the trace in user/taskset1\_smp\_balance.txt and user/taskset2\_smp\_balance.txt and note any differences to your previous scheduler in your README. You only need to submit results in this case for the same multi-CPU VM configuration as you used previously, but do not need to submit single-CPU VM results for this case. Write the updated average completion times for each workload in your README and compare against CFS.

### Deliverables
*   Modifications to the Linux source code and your Oven scheduler
*   user/taskset1\_smp\_balance.txt and user/taskset2\_smp\_balance.txt
*   Average completion times for both tasksets on a VM configured with either two or four CPUs, using the Oven scheduling policy with idle balancing in your README 
*   Comparison between the Oven (with idle balancing) average completion time and CFS average completion time in your README 

Part 6: Tail completion time
------
	
Thus far you have focused on optimizing the average completion time across all jobs. Another important metric to consider is tail completion time, which is the maximum completion time across 99% of all jobs. Tail completion time focuses on ensuring that the completion time of most jobs is no worse than some amount. Using tail completion time as the performance metric of interest, compare your optimized scheduling class versus the default Linux scheduler for the W4118 Inc. workloads. Which one does better? If the default Linux scheduler has better tail completion time, can you change how you use your OVEN scheduler (without modifying the OVEN scheduler code) to provide tail completion time comparable to the default Linux scheduler?

Make any changes to fibonacci , then submit eBPF traces of the two task sets running on your scheduler as user/taskset1\_tail.txt and user/taskset2\_tail.txt. You only need to submit results in this case for the same multi-CPU VM configuration as you used previously, but do not need to submit single-CPU VM results for this case. Write the tail completion times for each workload in your README and compare against CFS.

### Deliverables
*   Changes to fibonacci.c
*   user/taskset1\_tail.txt and user/taskset2\_tail.txt
*   Tail completion times for both tasksets on a VM configured with either two or four CPUs, using the Oven scheduling policy in your README 
*   Comparison between your tail completion time and CFS tail completion time in your README 

Part 7: Analysis and investigation of kernel source code
------
	
Write answers to the following questions in the user/part6.txt text file, following the provided template exactly. Make sure to include any references you use in your references.txt file.

1.  Give the exact URL on elixir.bootlin.com pointing to the file and line number of the function that initializes the idle tasks on CPUs other than the boot CPU for a multi-CPU system. What is the PID of the task that calls this function? Note: make sure you use v6.14.
2.  Give the exact URL on elixir.bootlin.com pointing to the file and line number at which the TIF\_NEED\_RESCHED flag is set for the currently running task as a result of its time quantum expiring if the task is scheduled using SCHED\_RR. Select the file and line number most related to the SCHED\_RR (i.e. do not select a generic helper function that may be used outside of SCHED\_RR). Note: make sure you use v6.14.
3.  Give the exact URL on elixir.bootlin.com pointing to the files and line numbers at which a timer interrupt occurring for a process running in user mode, whose time quantum has expired, results in schedule being called. Select the line of the call to schedule. This location is different for ARM64 and x86 - provide the answer for both. Note: make sure you use v6.14.
4.  What is the default time period for a tick? Write your answer in milliseconds. Hint: You may need to look in the kernel configuration files to find this answer.

## Debugging Tips

The following are some debugging tips you may find useful as you create your new scheduling class.

*   Before testing, take a snapshot of your VM if you have not done so already. That way, if your kernel crashes or is unresponsive because of scheduler malfunction, you can restore the state of the VM prior to the problem.
*   It is possible that your kernel gets stuck at boot time, preventing you from reading debug messages via dmesg. In this scenario, you may find it helpful to redirect your debug messages to a file on your host machine.
	1.  Right click on your VM and click "Settings"
	2.  Under "Hardware", click "Add" and create a Serial Port.
	3.  Under "Device", click on your new Serial Port.
	4.  Click on "Use output file", and specify the file on your host machine to which your would like to dump the kernel log.
	5.  Turn on your VM, move to your kernel in the GRUB menu, and press e.
	6.  Move toward the bottom until you find a line that looks like (linux   /boot/vmlinuz-6.8-cs4118...).
	7.  At the end of this line, replace quiet with console=ttyS0 (try console=ttyS1 if this doesn't work).
	8.  Hit F10 to boot your kernel. The kernel log should be written to the file on your host machine you specified earlier.
	9.  If neither ttyS0 nor ttyS1 work, you may need to remove the virtual printer hardware in your VMware VM settings.
*   You may find it helpful to use ps, top, or htop to view information such as the CPU usage and scheduling policies of your tasks. These commands rely on execution time statistics reported by certain scheduler functions in the kernel. As a result, bugs in your scheduling class could cause these commands to report inaccurate information. This cuts two ways. First, it is possible that your scheduler is working properly in terms of selecting the right tasks to execute for the right amount of time, but your calculation of execution time statistics in the kernel is wrong, so these commands appear to report that your tasks are not running at all when in fact they are. Second, it is possible that your scheduler is not working properly such that these tools report that tasks are using your scheduling class when in fact they are not.
	
	As a result, you should not exclusively rely on these tools to determine if your scheduling class is working properly. For example, when you make your scheduling class the default scheduling class, the fact that user-level tools claim that all tasks are using your scheduling class may not necessarily mean that this is the case. Instead, to ensure that your scheduling class functions are actually being used, you might add a printk to a function like your class's enqueue\_task() and verify that it appears in dmesg output. Make sure that you do not submit your code with such debugging printk statements as they can cause issues if invoked too frequently.
	

Submission Checklist
--------------------

Include the following in your main branch. Only include source code (ie \*.c,\*.h) and text files, do **not** include compiled objects.

*   .armpls file for teams with M1/M2 CPUs
*   README file that includes a description of your scheduling algorithm and how you do idle load balancing, the number of cores enabled in the VM you used for your measurements, and a discussion of your completion time results compared to CFS.
*   references.txt file
*   Implementation of your scheduler in linux/kernel/sched/oven.c
*   Changes to fibonacci.c
*   Implementation of trace.bt in user/trace.bt
*   Output of running your scheduler on the workloads in user/taskset1\_average.txt, user/taskset2\_average.txt, user/taskset1\_average\_smp.txt, user/taskset2\_average\_smp.txt, user/taskset1\_smp\_balance.txt, user/taskset2\_smp\_balance.txt, user/taskset1\_tail.txt, and user/taskset2\_tail.txt.
*   Answers to written questions in user/part7.txt
