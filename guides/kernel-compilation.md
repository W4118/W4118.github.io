# Kernel compilation in Ubuntu Linux

## Preparation

### Dependencies

First, update the package repository cache by running the following command:

```
$ sudo apt update
```

Then, install the following dependencies:

```
$ sudo apt install build-essential git bc python3 bison flex rsync libelf-dev libssl-dev libncurses-dev dwarves libdw-dev gawk
```

### Source code

Clone the source code for the kernel. The source code that you will use is
located in the `linux/` folder in the `main` branch of your team repository.
This directory will be the root of your kernel tree. All subsequent commands in
this guide should be run in that directory.

You should verify the version of the kernel. The first 6 lines of Linux’s
top-level `Makefile` will show you the version. For this class, we will be using
Linux 7.0.0:

```
$ head -n 6 Makefile
# SPDX-License-Identifier: GPL-2.0
VERSION = 7
PATCHLEVEL = 0
SUBLEVEL = 0
EXTRAVERSION =
NAME = Baby Opossum Posse
```

## Configuring your kernel build

The kernel build is configured using a file called `.config`. This file should
be located at the root of the kernel tree.

To create your kernel `.config`, use the following steps:

1. Remove any existing config files:

   ```
   $ make mrproper
   ```

2. Create a config file based on the config file of your current kernel. Make
   sure that you're running the stock Ubuntu kernel before you do this step. You
   don't want to copy a bad config! You can verify this by running `uname -r`.

   You should get something like `7.0.0-1010-gcp`.

   The config file that was used to build your current kernel is located in the
   `/boot/` directory. The following command copies over that file, and updates
   any missing options with default values.

   ```
   $ make olddefconfig
   ```

   It's okay if you see some warnings like this:

   ```
   .config:12661:warning: symbol value 'm' invalid for ANDROID_BINDER_IPC
   .config:12662:warning: symbol value 'm' invalid for ANDROID_BINDERFS
   ```

   You should now see a `.config` file in the root of your kernel tree.

3. Edit your `.config` file. The `.config` file consists of different options
   like `CONFIG_LOCALVERSION`, each of which is set to a certain value. Try
   running `cat` on your `.config` file to see some examples. There are two main
   ways to edit your `.config` file (apart from directly modifying it):

   - Run `make menuconfig`, which will open up an interactive menu.

     A backup of your previous config will be created at `config.old` every time
     you select `Save` in the interactive menu. If you choose this method, we
     recommend that you save once, only after making all of the changes listed
     below. This will allow you to take a clear diff of the updated `.config` vs
     the original config file saved in `.config.old`.

   - Use the config script that comes with the kernel source code:

     ```
     $ scripts/config --set-str <option> <value>  # Sets <option>
     $ scripts/config --state <option>            # Retrieves <option>
     ```

     Note that this method does not create a `.config.old` file. However, since
     you're modifying the config file created in step 2, you can find the
     original config file in `/boot/` if necessary.

   Make the following changes, using either of the above methods:

   - `CONFIG_LOCALVERSION`: This setting gives your custom kernel a unique name
     to distinguish it from other kernels present in your system. The local
     version will be appended to your kernel version to form your kernel name.
     For example, if we build a 7.0.0 kernel with the local version set to
     `-cs4118`, it will be named `7.0.0-cs4118`. For your pristine kernel
     build, set this to `-cs4118`.

     In menuconfig, this can be found under `General setup`, in the
     `Local Version - append to kernel release` option. Alternatively, you can
     run `scripts/config --set-str CONFIG_LOCALVERSION "-cs4118"` as mentioned
     above.

   - `SYSTEM_TRUSTED_KEYS`: This is used to bake additional trusted keys
     directly into the kernel image, which can be used to verify kernel modules
     before loading them. We don't need this, so set this to the empty string.

     In menuconfig, this can be found by opening the `Cryptographic API`
     section, then opening the `Certificates for signature checking` section at
     the bottom. The specific field is
     `Additional X.509 keys for default system keyring`. Alternatively, you can
     run `scripts/config --set-str SYSTEM_TRUSTED_KEYS ""` as mentioned above.

   - `SYSTEM_REVOCATION_KEYS`: Set this to the empty string as well.

     In menuconfig, this can be found by opening the `Cryptographic API`
     section, then opening the `Certificates for signature checking` section at
     the bottom. The specific field is
     `X.509 certificates to be preloaded into the system blacklist keyring`. The
     corresponding command is
     `scripts/config --set-str SYSTEM_REVOCATION_KEYS ""`.

Take a moment to inspect the contents of the `.config` file. Make sure that the
options you configured are set to what you expect them to be.

Note that apart from running `cat` on the config file, Linux also provides the
`scripts/diffconfig` utility, which can be used to compare different config
files. For example, if you used `make menuconfig`, you could do something like
this:

```
$ scripts/diffconfig .config.old .config
LOCALVERSION "" -> "-cs4118"
SYSTEM_TRUSTED_KEYS "debian/canonical-certs.pem" -> ""
SYSTEM_REVOCATION_KEYS "debian/canonical-revoked-certs.pem" -> ""
```

If you used `scripts/config`, you can do a diff against the stock config file in
the `/boot/` directory. For instance, run
`scripts/diffconfig /boot/config-$(uname -r) .config`. If you do this,
you'll probabably see some extra changes besides the three lines listed above.
That's okay, because `make olddefconfig` also updates some of the other configs.
Just make sure your desired changes are reflected in the output.

## Building the kernel

Before you start the build, consider running the
[optimization step](#optimizing-your-kernel-compilation-time). It could
**greatly** reduce your first-time compilation time.

To build the kernel, run the following as a **non-root user**:

```
$ make -j$(nproc)
```

In the command above, `nproc` evaluates to the number of cores in your VM. You
can also set the parallelization level to a different value by using `make -jN`,
where N is the number of parallel compilation jobs to run. Note that setting N
to greater than `nproc` won't necessarily make things faster, and may even lead
to more overhead. The first time that you build the kernel, it could take a
couple hours to complete depending on the speed of your computer.

## Installing the kernel

Run:

```
$ sudo make modules_install && sudo make install
```

Make sure that the installation **actually succeeds**, i.e. your output ends
with something like `done`. If the above command errors out, i.e. your output
ends with something like `make: *** [Makefile:240: __sub-make] Error 2`, try
doing the following:

```
$ sudo apt remove initramfs-tools
$ sudo apt clean
$ sudo apt install initramfs-tools
```

If `dpkg` reported errors during the `apt remove` step, you might also need to

```
$ sudo apt remove flash-kernel dracut
```

before doing the `clean` and `install` steps (you still only need to install
`initramfs-tools`).

Verify that you have the following 3 files in `/boot/`:

```
initrd.img-7.0.0-cs4118
System.map-7.0.0-cs4118
vmlinuz-7.0.0-cs4118
```

## Booting to the new kernel

> **IMPORTANT**: You should **ALWAYS** take a snapshot of your VM before
> booting a newly modified kernel. A snapshot provides a recovery point if the
> kernel or boot configuration makes the VM inaccessible.

GCP does not provide the VMware graphical console. You will use GCP's
interactive serial console to access GRUB and select a kernel.

### Configure GRUB

This configuration is required only once per VM. Create the following file:

```
$ sudo vim /etc/default/grub.d/99-cs4118.cfg
```

Add exactly:

```
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=30
GRUB_TERMINAL=console
GRUB_SERIAL_COMMAND=""
```

Then update GRUB:

```
$ sudo update-grub
```

Confirm that the output lists your custom kernel.

The separate `99-cs4118.cfg` file is necessary because Ubuntu's GCP image loads
configuration files in `/etc/default/grub.d/` after `/etc/default/grub`. The
`99-` prefix ensures that these settings take precedence over the cloud-image
defaults. `GRUB_TERMINAL=console` is required because N4A VMs do not expose the
legacy serial device expected by GRUB; GCP captures the UEFI console instead.

### Enable the interactive serial console

1. Go to **Compute Engine > VM instances** and click the VM's name.

2. Click **Edit**.

3. Under **Remote access**, enable **Connect to serial ports**, then click
   **Save**.

If this option is unavailable, add the following under **Custom metadata**:

```
Key: serial-port-enable
Value: TRUE
```

### Reboot into the custom kernel

1. Take a snapshot of the VM's boot disk.

2. Open the VM's details page and click **Connect to serial console**.

3. Keep the serial console open and run the following from your regular SSH
   session:

```
$ sudo reboot
```

4. In the serial console, select `Advanced options for Ubuntu`, then select the
   kernel whose name ends with `-cs4118`.

5. Reconnect through SSH and verify the running kernel:

```
$ uname -r
```

You should see `7.0.0-cs4118`.

If the custom kernel freezes or panics, click **Reset** on the VM's GCP page and
use the serial console to select the stock Ubuntu `-gcp` kernel. Always keep the
stock kernel installed as a recovery option.

## Optimizing your kernel compilation time

A large amount of time is spent compiling and installing kernel modules you
never use. To reduce your kernel compilation time, you can optionally regenerate
a `.config` so that it only contains modules you are using by following these
instructions:

1. Back up your `.config` to something like `.config.[UNI]-from-lts`. Make sure
   to keep your `CONFIG_LOCALVERSION` the same; that is, your kernel should
   still be named 7.0.0-cs4118.

2. Run `make localmodconfig` in your kernel source tree. This will take your
   current `.config` and turn off all modules that you are not using. It will
   ask you a few questions. You can hit ENTER to accept the defaults, or just
   have `yes` do so for you:

   ```
   $ yes '' | make localmodconfig
   ```

   Make sure that `CONFIG_BLK_DEV_LOOP` is still set to `y`. Now you have a much
   smaller `.config`. Then, build and install this kernel by following the rest
   of the steps starting from the section `Building the kernel`. Note that you
   only need to do `make localmodconfig` once, not each time you build the
   kernel.

## Overall workflow

When you are hacking kernel code, you’ll often make simple changes to only a
handful of `.c` files. If you didn’t touch any header files, the modules will
not be rebuilt when you run make; thus there is no reason to reinstall all
modules every time you rebuild your kernel. In this case, when compiling and
installing your kernel, you can simply do:

```
$ make -j$(nproc)
$ sudo make install
```

**Again, this assumes that you did NOT modify any header files potentially used
by kernel modules.** This also assumes you have not changed your kernel
configuration since you last ran `sudo make modules_install`.

Then, follow the steps in [Booting to the new kernel](#booting-to-the-new-kernel).

In other words, there is no
need to go through the sections `Configuring your kernel build` or
`Optimizing your kernel compilation time` each time you build your kernel. Those
steps only need to be done once. Of couse, if you do a fresh clone of the kernel
source code, you'll need to go through all of these steps again.
