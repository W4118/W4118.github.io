# Kernel compilation in Ubuntu Linux

## Preparation

### Dependencies

First, update the package repository cache by running the following command:

```
$ sudo apt update
```

Then, install the following dependencies:

```
$ sudo apt install build-essential bc python3 bison flex rsync libelf-dev libssl-dev libncurses-dev dwarves
```

### Source code

Clone the source code for the kernel. The source code that you will use is located in the `linux/` folder in the `main` branch of your team repository. This directory will be the root of your kernel tree. All subsequent commands in this guide should be run in that directory.

You should verify the version of the kernel. The first 6 lines of Linux’s top-level `Makefile` will show you the version. For this class, we will be using Linux 6.8.0:

```
$ head -n 6 Makefile
# SPDX-License-Identifier: GPL-2.0
VERSION = 6
PATCHLEVEL = 8
SUBLEVEL = 0
EXTRAVERSION =
NAME = Hurr durr I'ma ninja sloth
```

## Configuring your kernel build

The kernel build is configured using a file called `.config`. This file should be located at the root of the kernel tree.

To create your kernel `.config`, use the following steps:

1. Remove any existing config files:

    ```
    $ make mrproper
    ```

2. Create a config file based on the config file of your current kernel. Make sure that you're running the stock Ubuntu kernel before you do this step. You don't want to copy a bad config! You can verify this by running `uname -r`. You should get something like `6.8.0-41-generic`.

    The config file that was used to build your current kernel is located in the `/boot/` directory. The following command copies over that file, and updates any missing options with default values.

    ```
    $ make olddefconfig
    ```

    It's okay if you see some warnings like this:

    ```
    .config:12661:warning: symbol value 'm' invalid for ANDROID_BINDER_IPC
    .config:12662:warning: symbol value 'm' invalid for ANDROID_BINDERFS
    ```

    You should now see a `.config` file in the root of your kernel tree.

3. Edit your `.config` file. The `.config` file consists of different options like `CONFIG_LOCALVERSION`, each of which is set to a certain value. Try running `cat` on your `.config` file to see some examples. There are two main ways to edit your `.config` file (apart from directly modifying it):

    - Run `make menuconfig`, which will open up an interactive menu.

        A backup of your previous config will be created at `config.old` every time you select `Save` in the interactive menu. If you choose this method, we recommend that you save once, only after making all of the changes listed below. This will allow you to take a clear diff of the updated `.config` vs the original config file saved in `.config.old`.

    - Use the config script that comes with the kernel source code:

        ```
        $ scripts/config --set-str <option> <value>  # Sets <option>
        $ scripts/config --state <option>            # Retrieves <option>
        ```

        Note that this method does not create a `.config.old` file. However, since you're modifying the config file created in step 2, you can find the original config file in `/boot/` if necessary.

    Make the following changes, using either of the above methods:

    - `CONFIG_LOCALVERSION`: This setting gives your custom kernel a unique name to distinguish it from other kernels present in your system. The local version will be appended to your kernel version to form your kernel name. For example, if we build a 6.8.0 kernel with the local version set to `-cs4118`, it will be named `6.8.0-cs4118`. For your pristine kernel build, set this to `-cs4118`.

        In menuconfig, this can be found under `General setup`, in the `Local Version - append to kernel release` option. Alternatively, you can run `scripts/config --set-str CONFIG_LOCALVERSION "-cs4118"` as mentioned above.

    - `SYSTEM_TRUSTED_KEYS`: This is used to bake additional trusted keys directly into the kernel image, which can be used to verify kernel modules before loading them. We don't need this, so set this to the empty string.

        In menuconfig, this can be found by opening the `Cryptographic API` section, then opening the `Certificates for signature checking` section at the bottom. The specific field is `Additional X.509 keys for default system keyring`. Alternatively, you can run `scripts/config --set-str SYSTEM_TRUSTED_KEYS ""` as mentioned above.

    - `SYSTEM_REVOCATION_KEYS`: Set this to the empty string as well.

        In menuconfig, this can be found by opening the `Cryptographic API` section, then opening the `Certificates for signature checking` section at the bottom. The specific field is `X.509 certificates to be preloaded into the system blacklist keyring`.

Take a moment to inspect the contents of the `.config` file. Make sure that the options you configured are set to what you expect them to be.

Note that apart from running `cat` on the config file, Linux also provides the `scripts/diffconfig` utility, which can be used to compare different config files. For example, if you used `make menuconfig`, you could do something like this:

```
$ scripts/diffconfig .config.old .config
LOCALVERSION "" -> "-cs4118"
SYSTEM_TRUSTED_KEYS "debian/canonical-certs.pem" -> ""
SYSTEM_REVOCATION_KEYS "debian/canonical-revoked-certs.pem" -> ""
```

If you used `scripts/config`, you can do a diff against the stock config file in the `/boot/` directory. For instance, run `scripts/diffconfig /boot/config-6.8.0-41-generic .config`. If you do this, you'll probabably see some extra changes besides the three lines listed above. That's okay, because `make olddefconfig` also updates some of the other configs. Just make sure your desired changes are reflected in the output.

## Building the kernel

To build the kernel, run the following as a **non-root user**:

```
$ make -j$(nproc)
```

In the command above, `nproc` evaluates to the number of cores in your VM. You can also set the parallelization level to a different value by using `make -jN`, where N is the number of parallel compilation jobs to run. Note that setting N to greater than `nproc` won't necessarily make things faster, and may even lead to more overhead.

## Installing the kernel

Run:

```
$ sudo make modules_install && sudo make install
```

Make sure that the installation **actually succeeds**, i.e. your output ends with something like `done`. If the above command errors out, i.e. your output ends with something like `make: *** [Makefile:240: __sub-make] Error 2`, try doing the following:

```
$ sudo apt remove initramfs-tools
$ sudo apt clean
$ sudo apt install initramfs-tools
```

Verify that you have the following 3 files in `/boot/`:

```
initrd.img-6.8.0-cs4118
System.map-6.8.0-cs4118
vmlinuz-6.8.0-cs4118
```

## Booting to the new kernel

> **IMPORTANT**: You should **ALWAYS** take a snapshot of your VM before executing this step. If you boot into a buggy kernel and you do not have any snapshots, you will have to set up your VM from scratch again.

If you have not done so before, enable the boot selection screen (called grub) so that you can select the kernel you want to boot into. In particular:

- Open `/etc/default/grub` **as a root user**, for instance with `sudo vim /etc/default/grub`
- Replace `GRUB_TIMEOUT_STYLE=hidden` with `GRUB_TIMEOUT_STYLE=menu`
- Replace `GRUB_TIMEOUT=0` with `GRUB_TIMEOUT=30`
- Close the file and run `sudo update-grub`

Reboot your VM:

```
$ sudo reboot
```

When your VM boots up, select `Advanced options for Ubuntu` and choose the kernel you want. In this case, you'll choose the kernel whose name ends with `-cs4118` (the `CONFIG_LOCALVERSION` identifier you set in `.config`).

Now verify that you’re running your own custom kernel by running:

```
$ uname -r
```

Instead of `6.8.0-41-generic`, you should now see your kernel version string, `6.8.0-cs4118`!

## Optimizing your kernel compilation time

A large amount of time is spent compiling and installing kernel modules you never use. To reduce your kernel compilation time, you can optionally regenerate a `.config` so that it only contains modules you are using by following these instructions:

1. Back up your `.config` to something like `.config.[UNI]-from-lts`. Make sure to keep your `CONFIG_LOCALVERSION` the same; that is, your kernel should still be named 6.8.0-cs4118.

2. Run `make localmodconfig` in your kernel source tree. This will take your current `.config` and turn off all modules that you are not using. It will ask you a few questions. You can hit ENTER to accept the defaults, or just have `yes` do so for you:

    ```
    $ yes '' | make localmodconfig
    ```

    Make sure that `CONFIG_BLK_DEV_LOOP` is still set to `y`. Now you have a much smaller `.config`. Then, build and install this kernel by following the rest of the steps starting from the section `Building the kernel`. Note that you only need to do `make localmodconfig` once, not each time you build the kernel.

## Overall workflow

When you are hacking kernel code, you’ll often make simple changes to only a handful of `.c` files. If you didn’t touch any header files, the modules will not be rebuilt when you run make; thus there is no reason to reinstall all modules every time you rebuild your kernel. In other words, when installing your kernel, you can simply do:

```
$ sudo make install
```

This assumes you have previously run `sudo make modules_install` with the kernel configuration you are currently using.

Just to reemphasize the earlier point, when you are hacking kernel code, the standard workflow will be to modify kernel code, then build the kernel and install the updated kernel using the following two steps:

```
$ make -j$(nproc)
$ sudo make install
```

Then reboot your VM and select your kernel in grub. In other words, there is no need to go through the sections `Configuring your kernel build` or `Optimizing your kernel compilation time` each time you build your kernel. Those steps only need to be done once. Of couse, if you do a fresh clone of the kernel source code, you'll need to go through all of these steps again.
