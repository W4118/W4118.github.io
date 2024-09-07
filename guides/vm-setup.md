Setting up an Ubuntu Linux VM in VMware
======================================

You will be doing development and testing on a Linux system. To make this easy to do in an isolated environment running on your own personal computer, you will create a virtual machine (VM) in which you will install Linux. This guide provides instructions for installing Ubuntu Linux in a VMware VM. You will download and install VMware, download Ubuntu Linux, create a VM using VMware, then install Ubuntu Linux in your VM.

Download and Install VMware
---------------------------

Download and install the latest version of [VMware's desktop hypervisor](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion). Once you're in the Broadcom portal, go to the `My Downloads` tab. Then, you may need to click on the button in the top right corner beside your name, and select `VMware Cloud Foundation`.

Use:

- VMware Workstation if you use Windows or Linux
- VMware Fusion if you use macOS

Select the version labeled with `for Personal Use`.

Download Ubuntu Linux
---------------------

There are many GNU+Linux distributions, but we will use [Ubuntu](https://ubuntu.com/). Ubuntu is a distribution derived from [Debian](https://www.debian.org/). This class used to use Debian, but Ubuntu provides more recent kernel versions.

We will be using Ubuntu 24.04.1. All files mentioned below can be found [here](https://cdimage.ubuntu.com/noble/daily-live/current/).

- If your computer has an x86 CPU, download `noble-desktop-amd64.iso`
- If your computer has an Arm CPU, download `noble-desktop-arm64.iso`

Always verify the checksums of your downloads. You can follow the instructions [here](https://ubuntu.com/tutorials/how-to-verify-ubuntu#1-overview). Try to understand what each of these steps does. You will need to use the checksum files `SHA256SUMS` and `SHA256SUMS.gpg`.

Create a VM
-----------

Create a new virtual machine in VMware using the Ubuntu ISO image you just downloaded. For example, on VMware Fusion, this can be done by selecting `New` from the `File` menu. Select the `Install from disc or image` option and choose the relevant ISO file.

For the OS for the VM, choose `Ubuntu 64-bit ...` if available. Depending on your platform, it may not be an option yet. In that case, choose `Other Linux 6.x kernel 64-bit ...`. On some systems, you may need to click on `Go Back` after uploading your ISO file to set the OS.

For the boot firmware, choose `Legacy BIOS`. If you are using an M1/M2/M3 Mac, this option may not be available.

The default settings for memory and storage are probably too small. Customize your VM as detailed below. Note that on some platforms, you may need to press `Finish` and actually create the VM before you can edit these settings. Once the VM is created, click on `Virtual Machine` and then `Settings`.

- **Name**: (your choice; whatever name you want for saving the VM so you can open it again later in VMware)
- **CPU**: 2 processor cores
- **RAM**: 2 GB minimum; 4 GB or more highly recommended, provided that your host machine has 8 GB of RAM or more
- **Hard disk**: 64 GB minimum, 128 GB or more recommended

Installing and Setting Up Ubuntu
--------------------------------

After configuring the VM, start the VM. The VM will now boot from its CD-ROM drive containing the Ubuntu GNU/Linux install disk, which is virtually mapped to the ISO file that you downloaded.

Select `Try or Install Ubuntu`. You will then be guided through a somewhat lengthy installation process. Most of the default settings are acceptable – below is an outline for some of the choices you should be making:

- **Language**: English
- **Keyboard configuration**: English (US)
- **Connect to the internet**: Use wired connection
- **What do you want to do with Ubuntu?**: Install Ubuntu
- **How would you like to install Ubuntu?**: Interactive installation
- **Which apps would you like to install to start with?**: Default selection
- **Install recommended proprietary software?**: No need to select anything
- **How do you want to install Ubuntu?**: Erase disk and install Ubuntu
- **Create your account**:
    - Your name: This will be displayed on startup
    - Your computer's name: This will be the hostname you use for SSH
    - Your username: This will be the username you use for SSH
    - Password: This will be the password you use for SSH
- **Timezone**: Select your desired timezone

Then, select `Restart now`. This might take awhile. It's okay if you see a message on removing installation media. You can simply click `Ok`.

After your system restarts, you may also be prompted to configure the following additional settings:

- **Enable Ubuntu Pro**: Skip for now
- **Help improve Ubuntu**: No, don't share system data

Using your Ubuntu VM
--------------------

One tip you may find useful when using your VM is [using SSH to connect to your VM](./ssh.md).
