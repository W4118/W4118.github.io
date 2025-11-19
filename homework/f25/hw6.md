# W4118 Fall 2025 - Homework 6

> <span style="color:red">**DUE: Monday 12/8/2025 at 11:59pm ET**</span>

## General Instructions

All homework submissions are to be made via [Git][Git]. You must submit a detailed list of references as part of your homework submission indicating clearly what sources you referenced for each homework problem. You do not need to cite the course textbooks and instructional staff. All other sources must be cited. Please edit and include this [file][file] in the top-level directory of your homework submission in the `main` branch of your team repo. **Be aware that commits pushed after the deadline will not be considered.** Refer to the homework policy section on the [class website][class-web-site] for further details.

[Git]: https://git-scm.com/
[file]: https://www.cs.columbia.edu/~nieh/teaching/w4118/homeworks/references.txt
[class-web-site]: https://www.cs.columbia.edu/~nieh/teaching/w4118/

Group programming problems are to be done in your assigned groups. We will let you know when the Git repository for your group has been set up on GitHub. It can be cloned using the following command. Replace `teamN` with your team number, e.g. `team0`. You can find your group number [here](https://docs.google.com/spreadsheets/d/1V2ytwTpXo2rhVop9pYt46M74Bk4lOsXeB2vSqylVLMg/edit?usp=sharing).

```
$ git clone git@github.com:W4118/f25-hmwk6-teamN.git
```

This repository will be accessible to all members of your team, and all team members are expected to make local commits and push changes or contributions to GitHub equally. You should become familiar with team-based shared repository Git commands such as [git-pull][git-pull], [git-merge][git-merge], [git-fetch][git-fetch]. For more information, see [this guide](../../guides/git.md).

[git-pull]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-pull.html
[git-merge]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-merge.html
[git-fetch]: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/git-fetch.html

There should be at least five commits **per member** in the team's Git repository. The point is to make incremental changes and use an iterative development cycle. Follow the [Linux kernel coding style](https://www.kernel.org/doc/html/v6.14/process/coding-style.html). You **must** check your commits with the `run_checkpatch.sh` script provided as part of your team repository. Errors from the script in your submission will cause a deduction of points. (Note that the script only checks the changes up to your latest commit. Changes in the working tree or staging area will not be checked.)

For students on Arm computers (e.g. macs with M1/M2/M3 CPU): if you want your submission to be built/tested for Arm, you must create and submit a file called `.armpls` in the top-level directory of your repo; feel free to use the following one-liner:

```
$ cd "$(git rev-parse --show-toplevel)" && touch .armpls && git add .armpls && git commit -m "Arm pls"
```

You should do this first so that this file is present in any code you submit for grading.

For all programming problems, you should submit your source code as well as a single README file documenting your files and code for each part. Please do NOT submit kernel images. The README should explain any way in which your solution differs from what was assigned, and any assumptions you made.

For this assignment, the README should explicitly state which parts of the filesystem assignment were completed successfully and which parts are not functional. **It should also state explicitly how each group member contributed to the submission and how many hours each member spent on the homework.**

Finally, since this is the last assignment of the semester, EACH group member should indicate at the top of the README five important pieces of information:

1. The number of hours spent on this assignment
2. A rank ordering of the difficulty of the homework assignments
3. A rank ordering of how much you learned on each homework assignment
4. The extent to which you agree that this assignment has significantly improved your understanding of filesystems (1=strongly disagree, 2=disagree, 3=neutral, 4=agree, 5=strongly agree)
5. Any comments about your educational experience doing this assignment.

The format should be EXACTLY as shown below for each group member:

```
    abc123: 15hrs
    difficulty: hmwk1 < hmwk2 < hmwk4 < hmwk6 < hmwk5 < hmwk3
    learned: hmwk1 < hmwk2 < hmwk4 < hmwk3 < hmwk5 < hmwk6
    rating: 5
    comments: any comments here
```

This would indicate that student with UNI abc123 spent 15 hrs on this assignment, hmwk1 was the easiest and hmwk3 was the hardest, learned the least on hmwk1 and most on hmwk6, and strongly agree that this assignment significantly improved abc123's understanding of filesystems. The README should be placed in the top level directory of the main branch of your team repo. **5% of the grading points for this assignment will be allocated to grading your README.**

### Programming Problems:

In this assignment, you will write your own disk-based filesystem, EZFS. You may find it helpful to first review the [EZFS paper][ezfspaper], but keep in mind that some of the description in that paper is dated and does not apply to the 6.14 Linux kernel implementation you will do for this assignment. In particular, you should not use buffer heads for this assignment because buffer heads have been deprecated.  Instead, you will take advantage of iomaps, modern 64-bit systems including x86 and arm64, and the use of page-size blocks for EZFS to simplify your implementation.  Your filesystem only has to work on 64-bit systems.

You will learn how to use a loop device to turn a regular file into a block storage device, then format that device into an EZFS filesystem. Then you will use EZFS to access the filesystem. EZFS will be built as a kernel module that you can load into the **stock Ubuntu 25.04** kernel in your VM. You do not need to use the 4118 kernel you built for previous homework assignments and there is no need to build the entire Linux kernel tree for this assignment.

[ezfspaper]: https://www.cs.columbia.edu/~nieh/pubs/sigcse2025_ezfs.pdf

## Part 1: Formatting and mounting a disk

A loop device is a pseudo-device that makes a file accessible as a block device. Files of this kind are often used for CD ISO images. Mounting a file containing a filesystem via such a loop mount makes the files within that filesystem accessible. You will do this with EZFS, but to first gain some experience with a loop device, the following gives you a sample session for creating a loop device and building and mounting an ext2 filesystem on it. This session starts from the home directory of a user o_o. You should read man pages and search the Internet so you can understand what is going on at each step.

```console
$ sudo su
# dd if=/dev/zero of=./ext2.img bs=1024 count=224
224+0 records in
224+0 records out
229376 bytes (229 kB, 224 KiB) copied, 0.00198087 s, 116 MB/s
# modprobe loop
# losetup --find --show ext2.img
/dev/loop12
# mkfs -t ext2 /dev/loop12
mke2fs 1.47.0 (1-Jan-2025)
Discarding device blocks: done
Creating filesystem with 56 4k blocks and 32 inodes

Allocating group tables: done
Writing inode tables: done
Writing superblocks and filesystem accounting information: done

# mkdir mnt
# mount /dev/loop12 ./mnt
# df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
...
/dev/loop12    ext2      200K   24K  168K  13% /home/o_o/mnt
# cd mnt
/mnt# ls -al
total 24
drwxr-xr-x  3 root root  4096 Nov 17 19:39 .
drwxr-x--- 24 o_o  o_o   4096 Nov 17 19:41 ..
drwx------  2 root root 16384 Nov 17 19:39 lost+found
/mnt# mkdir sub2
/mnt# ls -al
total 28
drwxr-xr-x  4 root root  4096 Nov 17 19:42 .
drwxr-x--- 24 o_o  o_o   4096 Nov 17 19:41 ..
drwx------  2 root root 16384 Nov 17 19:39 lost+found
drwxr-xr-x  2 root root  4096 Nov 17 19:42 sub2
/mnt# cd sub2
/mnt/sub2# ls -al
total 8
drwxr-xr-x 2 root root 4096 Nov 17 19:42 .
drwxr-xr-x 4 root root 4096 Nov 17 19:42 ..
/mnt/sub2# mkdir sub2.1
/mnt/sub2# ls -al
total 12
drwxr-xr-x 3 root root 4096 Nov 17 19:42 .
drwxr-xr-x 4 root root 4096 Nov 17 19:42 ..
drwxr-xr-x 2 root root 4096 Nov 17 19:42 sub2.1
/mnt/sub2# touch file2.1
/mnt/sub2# ls -al
total 12
drwxr-xr-x 3 root root 4096 Nov 17 19:42 .
drwxr-xr-x 4 root root 4096 Nov 17 19:42 ..
-rw-r--r-- 1 root root    0 Nov 17 19:42 file2.1
drwxr-xr-x 2 root root 4096 Nov 17 19:42 sub2.1
/mnt/sub2# cd ../../
# umount mnt/
# losetup --find
/dev/loop13
# losetup --detach /dev/loop12
# losetup --find
/dev/loop12
# ls -al mnt/
total 8
drwxr-xr-x  2 root root 4096 Nov 17 19:41 .
drwxr-x--- 24 o_o  o_o  4096 Nov 17 19:41 ..
```

## Part 2: Exploring EZFS

Now that you understand how to use a loop device, mount a loop device and format it as EZFS. To do the latter, we have provided you with source code for an EZFS formatting program. First create a disk image and assign it to a loop device:

```console
# dd bs=4096 count=1000 if=/dev/zero of=~/ez_disk.img
# losetup --find --show ~/ez_disk.img
```

This will create the file `ez_disk.img` and bind it to an available loop device, probably `/dev/loop12`. Now, `/dev/loop12` can be used as if it were a physical disk, and the data backing it will be stored in `ez_disk.img`.

Now format the disk as EZFS. The skeleton code for a formatting utility program is in `format_disk_as_ezfs.c`. Compile it, then run it:

```console
# make format_disk_as_ezfs
# ./format_disk_as_ezfs /dev/loop12 1000
```

We have provided you with reference kernel modules that implement EZFS which are designed to work with your stock Ubuntu 25.04 kernel (6.14.0-34-generic). x86 and arm kernel modules are in `ref/ez-x86.ko` and `ref/ez-arm.ko`, respectively. You should familiarize yourself with writing and using Linux kernel modules. You can use the reference kernel module to explore your newly created EZFS by mounting the disk and loading the kernel module:

```console
# mkdir /mnt/ez
# insmod ez-ARCH.ko
# mount -t ezfs /dev/loop12 /mnt/ez
```

where ARCH is either `x86` or `arm`. Now you can create some new files, edit hello.txt, etc. If your kernel name is slightly different (e.g. `6.14.0-49-generic`), you may get a versioning error when you try to load the kernel module. In that case, you can try forcibly inserting the module with `insmod -f`.

## Part 3: Changing the formatting program

The formatting utility creates the new filesystem's root directory and places `hello.txt` in that directory. You can think of the formatting utility as statically creating the filesystem on a disk. You will first create directories and files by modifying the formatting utility, as this will help you later to figure out what EZFS must do to perform these filesystem operations.

Start by reviewing the EZFS [specification][specification], then read the formatting utility source code `format_disk_as_ezfs.c`. Make sure you understand the on-disk format and what each line contributes toward creating the filesystem. **A key simplifying concept in EZFS is how file data is stored, specifically directories are limited to one block in size and regular files may use multiple blocks but the blocks used for storing the data for a given file must be contiguous.**

[specification]: https://w4118.github.io/homework/f25/f25-ezfs-spec.pdf

Now extend the formatting utility program to create a subdirectory called `subdir`. The directory should contain `names.txt` that lists the names of your team members, `big_img.jpeg`, and `big_txt.txt`. The latter two files are in your repo subdirectory `big_files`. `names.txt` should be stored in disk block number 5, `big_img.jpeg` in disk block numbers 6-13, and `big_txt.txt` in disk block numbers 14-15. Be sure to set the directories' link counts correctly.

Create and format a new disk using your modified program. Use the reference EZFS kernel module provided to verify that the new files and directory were created correctly. You can use the `stat` command to see the size, inode number, and other properties. Note that the primary purpose of the reference EZFS kernel module is to provide a way to check that your formatting utility program operates correctly. It does not necessarily implement all of the functionality that you will provide in your own EZFS implementation.

Some Hints:
- As you modify your disk formatter, be mindful that you might not be formatting a fresh disk. Overwriting a disk you already formatted with an older version of your program might corrupt your disk.
- `hexdump` can be a useful tool for inspecting contents at the byte level.

## Part 4: Initializing and mounting the filesystem

### 4.1: Overview
Now that you understand how to manually add files to your filesystem via your formatting utility, the following parts will have you write a filesystem to allow you to use standard file commands. The rest of this assignment is structured to guide you toward incrementally implementing your filesystem functionality, which you will do by implementing `ez_ops.h` and `myez.c` in your team repo.

The parts are structured as follows:
* part 4: mount the filesystem
* part 5,6: list directories
* part 7: read files
* part 8: modify existing files 
* part 9: create new files
* part 10: delete files
* part 11: create and remove directories
* part 12: compile and run executables

In some cases, you may find that what you implemented for a given part is correct enough to get some piece of functionality working, but may not be completely correct such that some later functionality that depends on it ends up not working. Keep that in mind during your debugging. Also keep in mind that your implemented filesystem functionality should be compatible with the formatting utility, not the other way around; other than Part 3, you should not change the formatting utility, and certainly should not change it because your filesystem implementation is not working. Here are some resources that might be useful, though keep in mind that some of the information contained therein may be out of date:

- LKD chapter 13
- LKD chapter 14: pages 289 - 294
- LKD chapter 16: pages 326 - 331
- [Page cache overview slides][slides1]: "Block Devices" - "Block Devices (continued)"
- [Page cache + iomap overview slides][slides2]
- [Linux Kernel Internals chapter 3 (Virtual Filesystem)][lki]
- [Documentation/filesystems/vfs.rst][documentation]
- [Linux VFS tutorial at lwn.net][vfs_tutorial]

[slides1]: https://www.cs.columbia.edu/~nieh/teaching/w4118_f23/homeworks/hmwk6/page-cache-overview.pdf
[slides2]: https://docs.google.com/presentation/d/1_CKUdVg1mhUtbtH3vOLneLH27kzq_auk/edit?usp=sharing&ouid=109402832591562892559&rtpof=true&sd=true 
[lki]: https://tldp.org/LDP/lki/lki-3.html
[documentation]: https://elixir.bootlin.com/linux/v6.14/source/Documentation/filesystems/vfs.rst
[vfs_tutorial]: https://lwn.net/Articles/57369/

Note that the VFS has evolved over the years and some functions exist primarily for backwards compatibility with older filesystem implementations. **In your implementation, you should make sure to use the newer VFS interface functions discussed in class whenever possible.** This includes using filesystem contexts and iomaps, not older deprecated interfaces like buffer heads.  As always, the best source of correct information is the source code, especially other filesystem implementations, some of which were described in class, including [ramfs][ramfs] for a basic filesystem implementation. Other filesystem implementations are also good references to see what functions you have to implement and which ones you do not have to implement, or can implement by leveraging functions already provided by the VFS.  In particular, [bfs][bfs] is useful to see an older filesystem that uses contiguous allocation, [minix][minix] is useful to see a more complete but not too complex older filesystem, and [zonefs][zonefs] is useful to see a more complex filesystem that uses newer VFS interfaces including iomaps.  However, keep in mind that [bfs][bfs] and [minix][minix] use older VFS interfaces such as buffer heads and the mount method ([bfs][bfs]) that you should not use.

[ramfs]: https://elixir.bootlin.com/linux/v6.14/source/fs/ramfs
[bfs]: https://elixir.bootlin.com/linux/v6.14/source/fs/bfs
[zonefs]: https://elixir.bootlin.com/linux/v6.14/source/fs/zonefs
[minix]: https://elixir.bootlin.com/linux/v6.14/source/fs/minix

### 4.2: Initializing and mounting 

This part of the assignment focuses on writing the code that registers the filesystem and enables mounting disks. 

Create the basic functionality for your filesystem to work as a kernel module so that it can be loaded and unloaded from the kernel. After your module is installed, it should be visible (but not usable yet) as a filesystem on your VM. Look at `man 5 filesystems` for info about where filesystems are listed. This should be a good time for you to consider whether your device should be `nodev` or `bdev`.

Having registered your `myezfs` filesystem (nice job!), you'll now need to implement minimal functionality for being able to correctly mount and unmount the disk you just created. We won't be implementing any FS operations yet, but we have to initialize some pointers required by the kernel so that mounting and unmounting doesn't crash it. Besides looking at other filesystems as inspirations, we recommend tracing the system calls for [mount][mount-syscall] and [unmount][unmount-syscall] for what is initialized and which fields are used. Specifically, pay attention to the role of fs_context-related operation(s).

On mounting, you'll need to fill the in-memory kernel superblock data structure. This requires reading and having general access to the on-disk superblock. You'll do this by reading the EZFS superblock and inodes and assigning them to an instance of `struct struct ezfs_sb_pages` (defined in `ezfs.h`) that you'll create. Store this struct in the `s_fs_info` member of the VFS superblock. This way, we can always find the EZFS superblock and inodes by following the trail of pointers from the VFS superblock. This will become very useful later on. [The diagram][fill_super] in the Page Cache + iomaps Overview shows the relationship between these structs after the the superblock is read from disk and its in-memory representation is initialized.

The name attribute of your `struct file_system_type` MUST BE **myezfs**. _Failure to provide the correct naming of your filesystem will result in an automatic zero on your grade_.

Some Hints:

- Use `sb_set_blocksize()` to ensure that the block layer reads blocks of the correct size.
- You will have to fill out some additional members of the VFS superblock structure, such as the magic number and pointer to the ops struct.
- Make sure to handle errors by returning an appropriate error code. For example, what if someone asks you to mount a filesystem that isn't EZFS?
- Use `iget_locked()` to create a new VFS inode for the root directory. Read the kernel source to learn what this function does for you and get some hints on how you're supposed to use it. The only metadata you need to set is the mode. Make the root directory `drwxrwxrwx` for now.
- After creating an inode for the root directory, you need to create a dentry associated with it. Make the VFS superblock point to the dentry. Look at how other filesystems use `d_make_root` for this purpose.
- Remember to take care of any `struct page`s and dynamically allocated pointers in the function(s) called during unmount.

[fill_super]: https://docs.google.com/presentation/d/1_CKUdVg1mhUtbtH3vOLneLH27kzq_auk/edit?usp=sharing&ouid=109402832591562892559&rtpof=true&sd=true 
[mount-syscall]: https://elixir.bootlin.com/linux/v6.14/source/fs/namespace.c#L4088
[unmount-syscall]: https://elixir.bootlin.com/linux/v6.14/source/fs/namespace.c#L2077

## Part 5: Listing the contents of the root directory

In the previous part, you may have created a VFS inode without associating it with the corresponding EZFS inode from disk. Although this may be sufficient for `mount` to work, it will not be enough to properly list the contents of the root directory. You need to update your code to associate the root VFS inode with the root EZFS inode. Use the `i_private` member of the VFS inode to store a pointer to the EZFS inode. All of the EZFS inodes live in the inode store that we read from disk in the previous section. Consult the diagram in the EZFS Specification section.

Now you can add support for listing the root directory. You should be able to run `ls` and `ls -a`. Note that we do not support listing the contents of a subdirectory yet. Here's sample session:

```console
# ls /mnt/ez
ls: cannot access '/mnt/ez/hello.txt': No such file or directory # ok for now
ls: cannot access '/mnt/ez/subdir': No such file or directory    # ok for now
hello.txt  subdir
# ls -a /mnt/ez
ls: cannot access '/mnt/ez/hello.txt': No such file or directory # ok for now
ls: cannot access '/mnt/ez/subdir': No such file or directory    # ok for now
.  ..  hello.txt  subdir
# ls /mnt/ez/subdir
ls: cannot access '/mnt/ez/subdir': No such file or directory
```

Let's take a look at which system calls are behind this functionality.
The following is an excerpt from the output of `strace ls /usr/bin > /dev/null`:

```
[...]
openat(AT_FDCWD, "/usr/bin", O_RDONLY|O_NONBLOCK|...) = 3
[...]
getdents64(3, /* 1003 entries */, 32768) = 32744
[...]
getdents64(3, /* 270 entries */, 32768) = 8888
[...]
getdents64(3, /* 0 entries */, 32768)   = 0
close(3)                                = 0
```

In the example above, the `ls` program first opens the /usr/bin directory file, getting `fd` 3 back (why 3?). Then, it calls `getdents64()` three times to retrieve the list of 1,273 files in `/usr/bin`. Finally, `ls` closes the directory file. 

Currently, in your myezfs implementation, `ls`-ing your root directory should look something like this:

```console
# ls /mnt/ez
ls: cannot open directory '/mnt/ez': Not a directory
```

If we run the same command under `strace`, we see why that happens:

```console
# strace ls /mnt/ez
[...]
openat(AT_FDCWD, "/mnt/ez", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = -1 ENOTDIR (Not a directory)
[...]
```

openat is called to create a file descriptor for `"/mnt/ez"`. However, it fails, retrning `ENOTDIR` as it was given an `O_DIRECTORY` flag but could not determine that `"/mnt/ez"` was a directory. 
For right now, we can fix this by adding a no-op `lookup` member to `struct inode_operations`. We will come back to this member function, but for right now this will do.

With our newly working openat returning a file descriptor, we can lookup the entries in our root directory.
When `getdents64` is called, the VFS framework will call the `iterate_shared` member of the `struct file_operations`. Inside your iterate_shared implementation, use `dir_emit()` to provide VFS with the contents of the requested directory. VFS will continue to call `iterate_shared` until your implementation returns without calling `dir_emit()`. Each call to `getdents64()` will result in one call to `iterate_dir()`, which in turn will call your `iterate_shared` implementation. Consequently, your `iterate_shared` implementation should call `dir_emit()` until the given buffer is full. For now, you can pass in `DT_UNKNOWN` as the type argument for `dir_emit()`. We will revisit this in the next part. You can use the `ctx->pos` variable as a cursor to the directory entry that you are about to emit. 

Note that iterating through a directory using `dir_emit()` will list each directory entry contained in the directory, but what should be done to cause the `.` and `..` to appear in the listing? Some filesystems accomplish this by actually storing separate entries for `.` and `..` so that they will appear just like any other entry, but other filesystems do not, such as the proc filesystem. Look at how the proc filesystem achieves this [behavior][proc], and use a similar approach for your EZFS.

[proc]: https://elixir.bootlin.com/linux/v6.14/source/fs/proc/generic.c

Hints:
* Make sure you implement `iterate_shared`, not `iterate`, as the latter is an older interface.
* Running `ls -l` might print error messages because the `ls` program is unable to `stat` the files. This is the expected behavior for this part.

## Part 6: Accessing subdirectories

Add support for looking up filepaths. You should be able to `cd` into directories and `ls` the contents of directories that aren't the root. As a side effect, the `-l` flag and stat command should work on both files and directories now. Here's a sample session:

```console
# ls /mnt/ez/subdir
big_img.jpeg  big_txt.txt  names.txt
# cd /mnt/ez/subdir
# stat names.txt
    File: names.txt
    Size: 38           Blocks: 8      IO Block: 4096   regular empty file
Device: 7,12	Inode: 4           Links: 1
Access: (0666/-rw-rw-rw-)  Uid: (0 /    root)   Gid: (0 /    root)
Access: 2025-11-11 11:12:27.629345430 -0500
Modify: 2025-11-11 11:12:27.629345430 -0500
Change: 2025-11-11 11:12:27.629345430 -0500
    Birth: -
# stat does_not_exist.txt
stat: cannot stat 'does_not_exist.txt': No such file or directory
# ls -l ..
total 8
-rw-rw-rw- 1 root root    0 Nov 11 11:12 hello.txt
drwxrwxrwx 2 root root 4096 Nov 11 11:12 subdir
```

VFS does most of the heavy lifting when looking up a filepath. To avoid repeated work when looking up similar paths, the kernel maintains a cache called the dentry cache. Learn how the dentry cache works by reading the materials given earlier. A given path is split up into parts and each part is looked up in the dentry cache. If a part isn't in the dentry cache, the VFS will call the filesystem-specific `lookup` function of `inode_operations` to ask the filesystem to add it. For example, given a filepath such as `/a/b/c/d/e/f.txt`, once the kernel knows the inode of c, it will ask for the inode associated with the name d in the directory c. If there is no matching dentry in the cache, the lookup function will be called to retrieve the inode for d from the filesystem. Before you add things to the dentry cache, you are responsible for determining whether the given parent directory contains an entry with the given name.

Make sure your code returns correct metadata for all files and directories. These include size, link count, timestamps, permissions, owner, and group.

- Test by using `ls -l` and `stat`.
- You should also pass the correct type to `dir_emit()` in your `iterate_shared` member. Check out this [StackOverflow post][readdir] for why it matters. **Hint:** you should use `S_DT()`.

[readdir]: https://stackoverflow.com/questions/47078417/readdir-returning-dirent-with-d-type-dt-unknown-for-directories-and

## Part 7: Reading the contents of regular files
Add support for reading the contents of files. There are a number of ways to do this, but you should take advantage of generic functions that are already available as part of the VFS to implement `read_iter`, not `read`. For example, `generic_file_read_iter` handles complex logic to read ahead so that file blocks can be cached in memory by the time they are actually needed to avoid blocking on slow I/O devices. However, generic filesystem functions are unaware of filesystem-specific functionality for deciding what data blocks are actually associated with each file, so the job of the filesystem is to provide that information through appropriate functions that will be called by the generic functions. For this assignment, you will be using iomaps for file I/O operations, meaning you will supply the association between data blocks and files through iomaps. 

Some Hints 
- You should read `generic_file_read_iter` to understand how it interacts with `address_space_operations` to see what functions need to be implemented. What is `read_folio` and how is it used? How does zonefs use iomaps to implement read functionality?
- You may find it helpful to refer to the suggestions in the iomap overview above to understand the role of `iomap_begin` and `iomap_end`.
- In `iomap_begin`, what if the file does not have any blocks associated with it? Alternatively, what if the requested section is beyond what is allocated for that file?

Once you have read support, you should be able to do the following:

```console
# cat /mnt/ez/hello.txt
Hello world!
# cat /mnt/ez/subdir/names.txt
Emma Nieh
Haruki Gonai
Zijian Zhang
# dd if=/mnt/ez/hello.txt
Hello world!
0+1 records in
0+1 records out
13 bytes copied, 4.5167e-05 s, 266 kB/s
# dd if=/mnt/ez/hello.txt bs=1 skip=6
world!
7+0 records in
7+0 records out
7 bytes copied, 5.1431e-05 s, 117 kB/s
```

If you try using other programs to read files, you may encounter some errors. For example, vim by default places swap files in the current directory and seeks through them upon opening a file using `llseek`. You may have noticed an error when trying to open files using vim because your EZFS has no support for `llseek` yet. Fix it. **Hint:** there's already a generic implementation in the kernel for `llseek`.

At this point, you should stress test your EZFS implementation. The rest of this assignment will be easier if you can depend on the reading functionality to report things correctly. Some of the things you should make sure work include:

- Try copying all the files out of your ez using `cp` or `rsync`.
- Extend the formatting program again to create additional files and a more complex directory structure. Be sure to include different file types and larger files. For example, add a small team photo to the `subdir` directory.
- Overwrite the disk with random garbage from `/dev/urandom` (instead of `/dev/zero`). Format it. After formatting, the random data should not affect the normal operation of the filesystem.
- Write a program that requests invalid offsets when reading files or iterating through directories.

## Part 8: Writing to existing files

### 8.1: Writing in memory

So far, we've only been reading what's already on the filesystem. Implement functions for modifying the filesystem contents. Again, you should implement `write_iter` instead of `write`. In order to implement `write_iter`, read through the code for `iomap_file_buffered_write` and examine how it is used by zonefs, xfs, and other filesystems. Try to understand how it helps us to write iteratively, and find out how it interacts with `iomap_ops`. As you consider the different cases of writing to a file in the context of EZFS, add additional functionality to `iomap_begin` with that in mind. Fundamentally, `iomap_begin` identifies the data blocks that correspond to the file region being written to. Make sure to also update `iomap_end` to support the write functionality.

In this part, you should also handle writes that shrink files (i.e. the `truncate` syscall should be supported). 

We recommend you first make sure your write functionality works for a file that requires no more than one data block for its contents. Test for writing the contents of files:

```console
$ cd /mnt/ez
$ echo -ne "4118" | dd of=hello.txt bs=1 seek=7 conv=notrunc
[...]
$ cat hello.txt
Hello w4118!
$ echo "Greetings and salutations, w4118!" | dd of=hello.txt conv=notrunc
[...]
$ cat hello.txt
Greetings and salutations, w4118!
```

Once you have the one block case working, then you should consider what if the file requires more than one block. EZFS only supports contiguous allocation of blocks to a file, so there are two cases to consider. The easier case is if the data block following the last existing data block of this file is empty, in which case you can allocate this block to it.

```console
$ ls -al /mnt/ez/subdir/
total 52
drwxrwxrwx 2 o_o o_o  4096 Nov 17 19:05 .
drwxrwxrwx 3 o_o o_o  4096 Nov 17 19:05 ..
-rw-rw-rw- 1 o_o o_o 29296 Nov 17 19:05 big_img.jpeg
-rw-rw-rw- 1 o_o o_o  4169 Nov 17 19:05 big_txt.txt
-rw-rw-rw- 1 o_o o_o    38 Nov 17 19:05 names.txt
$ cat /mnt/ez/subdir/big_img.jpeg >> /mnt/ez/subdir/big_txt.txt
$ stat /mnt/ez/subdir/big_txt.txt
  File: /mnt/ez/subdir/big_txt.txt
  Size: 33465     	Blocks: 72         IO Block: 4096   regular file
Device: 7,12	Inode: 6           Links: 1
Access: (0666/-rw-rw-rw-)  Uid: ( 1000/     o_o)   Gid: ( 1000/     o_o)
Access: 2025-11-17 19:05:19.000000000 -0500
Modify: 2025-11-17 19:07:06.094968851 -0500
Change: 2025-11-17 19:07:06.094968851 -0500
 Birth: -

$ # big_txt.txt's data_block_range change from [14-15] to [14-22]
```

The harder case is because files are limited to contiguous allocation of data blocks, you should move the existing blocks along with the new block to another position so that there is enough space for the contiguous region of blocks.

```console
$ # reformat the disk img
$ stat /mnt/ez/hello.txt
  File: /mnt/ez/hello.txt
  Size: 13        	Blocks: 8          IO Block: 4096   regular file
Device: 7,12	Inode: 2           Links: 1
Access: (0666/-rw-rw-rw-)  Uid: ( 1000/     o_o)   Gid: ( 1000/     o_o)
Access: 2025-11-17 19:05:19.000000000 -0500
Modify: 2025-11-17 19:05:19.000000000 -0500
Change: 2025-11-17 19:05:19.000000000 -0500
 Birth: -
$ cat /mnt/ez/subdir/big_txt.txt >> /mnt/ez/hello.txt
$ stat /mnt/ez/hello.txt
  File: /mnt/ez/hello.txt
  Size: 4182     	Blocks: 16         IO Block: 4096   regular file
Device: 7,12	Inode: 2           Links: 1
Access: (0666/-rw-rw-rw-)  Uid: ( 1000/     o_o)   Gid: ( 1000/     o_o)
Access: 2025-11-17 19:05:19.000000000 -0500
Modify: 2025-11-17 19:10:09.428618497 -0500
Change: 2025-11-17 19:10:09.428618497 -0500
 Birth: -

$ # hello.txt's data_block_range change from [3-3] to [16-17]
```

You should also be able to edit files with the nano editor, although it will complain about `fsync()` not being implemented. Fix this problem.

If there is not enough space in your filesystem to write what you need to write, you should return an appropriate error, specifically [`ENOSPC`][ENOSPC]. Keep in mind that there may be multiple reasons why there is not enough space.

[ENOSPC]: https://elixir.bootlin.com/linux/v6.14/source/include/uapi/asm-generic/errno-base.h#L32

Now that your filesystem is being modified, you should take care to make sure that concurrent file operations are being handled properly. For example, if two files are being modified at the same time, you want to make sure that you do not accidentally assign the same free data block to both files, which would obviously be an error. Make sure that your EZFS operations work properly when multiple processes or threads are performing those operations at any given time. Keep in mind that page cache operations such as read_mapping_page may block if they need to go to disk. While we are not using buffer heads for this assignment, you may find it helpful to review how synchronization is handled in [BFS][BFS].

[BFS]: https://elixir.bootlin.com/linux/v6.14/source/fs/bfs

Once you can write multi-block files, you should also ensure you can seek to different positions of a file to write data. For example, you should be able to write to the first block of a file, seek 100 blocks ahead and then write to that block of the file. After writing such a file, what should you see when you read the file? Supporting seeking and writing may require additional implementation effort. Note that there is also a `zero_blocks` bit vector in the EZFS superblock in `ezfs.h`; if helpful, you may use that for your implementation.

### 8.2: Writing to disk

While working on the previous subsection, you might have noticed that all your writes don't seem to persist to disk. To verify, try unmounting and remounting a modified disk. You'll get a view of the filesystem as you had never made any changes. Now you'll take the appropriate measures so that your modifications are written to disk. 

Ensure that changes to the VFS inode are written back to disk. You should do this by implementing the `.write_inode` member of your `struct super_operations`. Of course, VFS needs to be informed that the VFS inode is out of sync with the EZFS inode. Upon modifying an in-memory inode, you should signal to the kernel that it requires peristing to disk. If successful, the kernel will later call your `.write_inode` implementation on that inode.

When writing, you also need to signal to the kernel which blocks need to be persisted. This includes data blocks and your superblock.

## Part 9: Creating new files

Implement creating new files. That is, user programs should be able to call `open()` with a mode that includes `O_CREAT`. **Note that an empty file should have 0 data blocks.** Here's a sample session:

```console
cd /mnt/ez/
$ ls
hello.txt  subdir
$ touch world.txt
$ ls
hello.txt  subdir  world.txt
$ stat world.txt
  File: world.txt
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: 7,12	Inode: 7           Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/     o_o)   Gid: ( 1000/     o_o)
Access: 2025-11-17 19:51:03.287875291 -0500
Modify: 2025-11-17 19:51:03.287875291 -0500
Change: 2025-11-17 19:51:03.287875291 -0500
 Birth: -
$ cat > subdir/favorite_memes.txt
doge
chad
BigTime Tommie
https://youtu.be/TiC8pig6PGE  # Ctrl+D to denote EOF
$ cat subdir/favorite_memes.txt
doge
chad
BigTime Tommie
https://youtu.be/TiC8pig6PGE
```

Note that the size of an ezfs data block is 4KB, whereas the VFS block size is 512B. When you `stat` a file, the number of blocks should correspond to the VFS block size. Make sure to correctly convert between the two definitions as needed.

## Part 10: Deleting Files

While testing the previous part, you probably created lots of files that are now cluttering your disk. Let's implement a way to delete those files.

Review how the VFS dentry and inode caches interact with each other using the resources given earlier in this assignment. Implement the `unlink` and `evict_inode` ops so that you can delete files.

You are not required to implement directory removal in this part, that will happen in the next part. Ensure that you are reclaiming data blocks and EZFS inodes when appropriate. To test this, see if you can repeatedly create and remove files.

```
for i in {1..10}; do touch {1..14}; rm {1..14}; done
```

In a Unix-like operating system, what is the correct behavior if one process unlinks a file while another process has the same file open? Here's an experiment you can run on ext4 or the EZFS reference implementation to find out:

- Create a file named `foo`.
- In terminal window A, run `tail -f foo`. This command will open `foo`, print out all the contents, and wait for more lines to be written.
- In terminal B, run `cat > foo`. This reads from stdin and outputs the result to `foo`.
- In terminal C, delete `foo`.
- Back in terminal B, type some text and press return.
- The text should appear in terminal A.

## Part 11: Making and removing directories

Implement creating new directories. That is, user programs should be able to call `mkdir()`. This should be very similar to what you did to support creating regular files. You need to make sure that you're setting a size and link count appropriate for a directory, rather than a regular file. Hint: consider the link count of the parent directory of the newly created directory as well. In this part as well as the preceding ones, you should make sure that whatever robustness tests you did earlier continue to pass.

Implement deleting directories. User programs should be able to call `rmdir()` successfully on empty directories. This should be very similar to what you did in the previous part. Take a look at `simple_rmdir()` for some additional directory-specific steps. Note that `simple_empty()` is not sufficient to check if a directory is empty for our purposes, because the function simply checks the dentry cache to see if a directory has children. Can you think of a case where this would lead to incorrect behavior?

Here's a sample session:

```console
$ ls -alF
total 16
drwxrwxrwx 3 o_o  o_o  4096 Nov 17 20:22 ./
drwxr-xr-x 3 root root 4096 Nov 17 20:23 ../
-rw-rw-rw- 1 o_o  o_o    13 Nov 17 20:22 hello.txt
drwxrwxrwx 2 o_o  o_o  4096 Nov 17 20:22 subdir/
$ mkdir bigtime
$ ls -alF
total 20
drwxrwxrwx 4 o_o  o_o  4096 Nov 17 20:23 ./
drwxr-xr-x 3 root root 4096 Nov 17 20:23 ../
drwxr-xr-x 2 o_o  o_o  4096 Nov 17 20:23 bigtime/
-rw-rw-rw- 1 o_o  o_o    13 Nov 17 20:22 hello.txt
drwxrwxrwx 2 o_o  o_o  4096 Nov 17 20:22 subdir/
$ cd bigtime
$ touch tommie
$ ls -alF
total 8
drwxr-xr-x 2 o_o o_o 4096 Nov 17 20:24 ./
drwxrwxrwx 4 o_o o_o 4096 Nov 17 20:23 ../
-rw-r--r-- 1 o_o o_o    0 Nov 17 20:24 tommie
$ cd ..
$ rmdir bigtime
rmdir: failed to remove 'bigtime': Directory not empty
$ ls -alF
total 20
drwxrwxrwx 4 o_o  o_o  4096 Nov 17 20:23 ./
drwxr-xr-x 3 root root 4096 Nov 17 20:23 ../
drwxr-xr-x 2 o_o  o_o  4096 Nov 17 20:24 bigtime/
-rw-rw-rw- 1 o_o  o_o    13 Nov 17 20:22 hello.txt
drwxrwxrwx 2 o_o  o_o  4096 Nov 17 20:22 subdir/
$ rm bigtime/tommie
$ rmdir bigtime
$ ls -alF
total 16
drwxrwxrwx 3 o_o  o_o  4096 Nov 17 20:25 ./
drwxr-xr-x 3 root root 4096 Nov 17 20:23 ../
-rw-rw-rw- 1 o_o  o_o    13 Nov 17 20:22 hello.txt
drwxrwxrwx 2 o_o  o_o  4096 Nov 17 20:22 subdir/
```

## Part 12: Compile and run executable files

Compiling and running executable files requires some additional functionality beyond what you have already implemented, specifically support for `mmap`. Given the approach you should have taken thus far, implementing `mmap` support should be trivial. Do it. At this point, you should now be able to compile and execute programs. This part will also double verify that you implemented the functionality of "read/write/fsync", "create/delete" correctly.

Here's a sample session:

```console
$ cd /mnt/ez
$ vim test.c
$ ls
hello.txt  subdir  test.c
$ cat test.c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
$ gcc test.c
$ ls
a.out  hello.txt  subdir  test.c
$ ./a.out
Hello, World!
```

At this point, you should make sure that whatever robustness tests you did earlier continue to pass with your completed filesystem, and your tests should include having multiple processes or threads perform various filesystem operations concurrently. In addition, you should try running various programs manipulating the files in your filesystem. You should also make sure you test by unmounting and remounting to make sure all your programs manipulating files work correctly with the file data actually written to disk and not just file data in the page cache. In your README, note which applications you have used, which ones worked, and which ones do not. What are some file operations supported on your default Linux filesystem that are not supported by EZFS? Which of these affect the functionality of the programs you ran?

## Submission Checklist

Include the following in your main branch. Only include source code (ie \*.c,\*.h) and text files, do **not** include compiled objects.

- `.armpls` file for teams developing on arm CPUs
- `README` file with additional course evaluation information
- `references.txt` file
- Implementation of `format_disk_as_ezfs.c`
- Implementation of `myez.c` and `ezfs_ops.h`
