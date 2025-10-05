# Kernel Debugging using QEMU and GDB

This (very rough) guide will tell you how to setup kernel debugging using QEMU
and GDB/LLDB. This is _not_ intended to be used to solve _all_ kernel debugging
-- if you can boot using your kernel normally and it can do most things fairly
stably, you could probably still debug just fine using the old `printk` and
`dmesg` combo.

However there comes inevitably a time when you are unable to even boot your
kernel and are unable to get any information even from serial ports, possibly
because the issues occur in your system in an early enough or critical enough
code path such that none of these methods even egress any information. Then
QEMU/GDB might be your best way forward.

We introduce a relatively simple process to debug a crashing kernel during boot,
as well as a
[slightly more sophisticated process](#debugging-with-custom-binary-in-initramfs-image)
that will allow you to include custom binaries to your init image, so that you
can invoke custom system calls or do more advanced things and observe your
kernel's responses.

## One-time Setup

```sh
# for arm
$ sudo apt install qemu-system-aarch64 gdb
# for x86
$ sudo apt install qemu-system-x86 gdb
```

## Per-run Setup

Before you start, I recommend you setup some sort of terminal multiplexer that
will let you be able to run multiple processes at once and see all their
consoles simultaneously. I recommend using `tmux` or `zellij` for terminal
multiplexing; both should be available on `apt`, but I suppose the built-in one
from VS Code also works.

For your first terminal, go into your `linux` directory, and then run:

```sh
# both assumes you are currently in the `linux` directory
# in your homework repo. Pick the one appropriate for you.

# for arm
$ qemu-system-aarch64 \
  -M virt \
  -cpu cortex-a57 \
  -m 2G \
  -kernel arch/arm64/boot/Image \
  -initrd /boot/initrd.img-6.14.0-cs4118 \
  -append "console=ttyAMA0 nokaslr" \
  -nographic \
  -s -S

# for x86 (untested)
$ qemu-system-x86 \
  -m 2G \
  -kernel arch/x86/boot/Image \
  -initrd /boot/initrd.img-6.14.0-cs4118 \
  -append "console=ttyS0 nokaslr" \
  -nographic \
  -s -S
```

This might not seem to do anything, which is expected; `-S` will pause the
execution so that you can connect and resume at your leisure.

In a separate window, run

```sh
$ gdb vmlinux -ex "target remote :1234"
```

If you got something like the below or something even more meaningful:

```
Remote debugging using :1234
0x0000000040000000 in ?? ()
```

![](/images/gdb.png)

Then you are officially in.

## Typical Workflow

Before diving in, there are some things you can do to make the experience
slightly better:

```
layout split
```

This will give you three separate panels -- the top one for viewing source code,
the middle one for assembly, and the bottom one for interacting with the
debugger using a REPL.

Now you can do stuff like setting a breakpoint to a function that you know the
name of, and `continue`/`c` until the kernel reaches that point.

### Breakpoints

Adding breakpoint:

```
break my_func
```

Regex breakpoint:

```
rbreak kernel/sched/oven.c:.*_oven
```

### Context

```
print var  # prints variables visible in current scope
print var.field

backtrace  # shows stack trace
```

### Continue

```
c  # continue
s  # step, diving into functions if any
n  # continue on current level, no function diving
finish  # step out of current function
```

`Enter` key press will automatically re-execute last command.

`ctrl-c` will stop execution directly.

### Tips

Working with `list_head`s:

```gdb
print *$container_of(rq.wfq.tasks.next, "struct task_struct", "wfq_list_node")
```

Setting up gdb to TUI mode and to auto-accept linux's gdb python scripts:

- create `~/.gdbinit`;
- Add as content:

  ```
  layout split

  add-auto-load-safe-path .
  ```

### Beyond these...

[gdb cheatsheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf)

## Alternative Debugger - LLDB

You can use `lldb` instead of `gdb` and then connect to QEMU using
`gdb-remote localhost:1234`.

You can use some of the same or similar commands like `b some_func`, `c` to
continue, etc. It also has a fancier `gui` command that gives you a terminal
interface that gives you an interactable source code window, a variable panel,
and a list of threads.

![](/images/lldb.png)

You can move from one panel to another using `<tab>`, toggle breakpoints in the
source code panel using `b`, or run to highlighted line with `<enter>`.

`h` will give you the set of keymaps you can use in the current space.

Once you are done with the gui, `<esc>` will put you back to the command screen.

You can find out more at https://lldb.llvm.org/use/tutorial.html.

## Teardown

```sh
$ killall qemu-system-aarch64
# or -x86
```

Will stop the running QEMU process. `lldb` will turn off QEMU for you when you
exit.

## Debugging with custom binary in Initramfs Image

You will realize that the mini-OS running with the initramfs that we specified
doesn't let you do a whole lot. It has only a few binaries, and seems pretty
restrictive overall, as is expected. However, what if you want to test your new
syscalls?

Fortunately, we can bring in custom binaries into

Before other things, you want to make sure to compile your binary with the
`-static` flag as the initramfs environment won't have the usual dynamically
linked libraries that we need. You can likely just reuse the `Makefile` and add
the flag to `CFLAGS`.

Then, in some random location:

```sh
# if you are currently in the linux directory inside your team homework repo:
$ cd ../..

# create a working dir for the temporary RAM-based filesystem that QEMU uses
# for booting. Make it ouside your repo so we don't interfere unnecessarily.
$ mkdir initrd

# unpacks the version created normally using `make`
$ zstd -d /boot/initrd.img-6.14.0-cs4118 -o initrd.img.cpio

# go into the working dir
$ cd initrd
# extract content to current dir
$ sudo cpio -idm < ../initrd.img.cpio

# create tmp dir and move desired binary here
$ mkdir tmp
$ cp ${BINARY_PATH} tmp/

# (optional) make sure you are still in `initrd`
$ pwd

# bundle custom image file
$ find . | cpio -o -H newc | gzip -c > ../custom.img

# go to linux dir
$ cd ../
$ cd f25-hmwkN-teamM  # sub with your appropriate local version
$ cd linux

# run QEMU with revised command using the custom init image
$ qemu-system-aarch64 \
  -M virt \
  -cpu cortex-a57 \
  -m 2G \
  -kernel arch/arm64/boot/Image \
  -initrd ../../custom.img \
  -append "console=ttyAMA0 nokaslr" \
  -nographic \
  -s -S
```

You should now be able to set a breakpoint to a custom kernel function that you
implemented and walk through it using your debugger of choice. Once you setup
QEMU and the debugger like before, click `c` to let QEMU resume booting, and it
will go through the booting process until getting to a shell:

```
(initramfs)
```

Now you can run your custom binary from the place you put it:

```
(initramfs) ./tmp/test
```

Sample GDB: ![](/images/gdb_custom.png)

Sample LLDB: ![](/images/lldb_custom.png)

Your workflow now gets a bit more interesting: you would likely alternate
between the two terminals: set some breakpoint on the functions you want to see,
running `c` in `gdb` to get back to the QEMU shell, run your custom binary which
could possibly invoke your new syscall, wait for QEMU to get stuck on the
breakpoint, return to `gdb` to analyze and walk the instructions, and then rinse
and repeat.

## Editor Integration - Neovim

(Sorry I don't know how regular `vim` should be setup)

Include the following in your `lazy` configs:

```lua
local pretty_print = {
  {
    text = "-enable-pretty-printing",
    description = "enable pretty printing",
    ignoreFailures = false,
  },
}

return {
  {
    "mfussenegger/nvim-dap",
    cmd = { "DapNew" },
    config = function()
      local dap = require("dap")

      -- uses `cpptools` from vsc; install using mason -- you have mason, right?
      dap.adapters.cppdbg = {
        id = "cppdbg",
        type = "executable",
        command = "OpenDebugAD7",
      }

      dap.configurations.c = {
        {
          name = "Launch file",
          type = "cppdbg",
          request = "launch",
          program = function()
            return vim.fn.input("Path to executable: ", vim.fn.getcwd() .. "/", "file")
          end,
          cwd = "${workspaceFolder}",
          stopAtEntry = true,
          setupCommands = pretty_print,
        },
        {
          name = "Attach to gdbserver :1234",
          type = "cppdbg",
          request = "launch",
          MIMode = "gdb",
          miDebuggerServerAddress = "localhost:1234",
          miDebuggerPath = "/usr/bin/gdb",
          cwd = "${workspaceFolder}",
          program = function()
            return vim.fn.input("Path to executable: ", vim.fn.getcwd() .. "/", "file")
          end,
          setupCommands = pretty_print,
        },
      }
      dap.configurations.cpp = dap.configurations.c
    end,
  },
  {
    "rcarriga/nvim-dap-ui",
    cmd = { "DapUI" },
    dependencies = {
      "mfussenegger/nvim-dap",
      "nvim-neotest/nvim-nio",
    },
    config = function()
      local dap = require("dap")
      local dapui = require("dapui")
      dapui.setup()

      -- manual cmds for dapui
      vim.api.nvim_create_user_command("DapUIOpen", dapui.open, {})
      vim.api.nvim_create_user_command("DapUIClose", dapui.close, {})
      vim.api.nvim_create_user_command("DapUI", dapui.toggle, {})

      vim.api.nvim_create_user_command("Debugger", function()
        dapui.open()
        dap.continue()
      end, {})
    end,
  },
}
```

This gives you a wrapper command "Debugger". Once you start QEMU the way
described above, instead of running `gdb`, you open the files you want to edit
in the linux kernel with `neovim`, then run `Debugger`, choose
`Attach to gdbserver`, and input the path to `vmlinux` (you might need to input
a few `../`s). You should now be able to interact with `gdb` using the DAP
protocol, though I would also recommend having some number of custom keybinds to
the various `Dap` operations, e.g. `DapToggleBreakpoint`.

I personally use [hydra](https://github.com/nvimtools/hydra.nvim) to create a
special debug mode for debugging-specific keybinds.

## Further Reading

[Qemu's gdb guide](https://qemu-project.gitlab.io/qemu/system/gdb.html)
