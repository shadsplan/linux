# Chapter 14. File Systems

## 14.1 Device Special Files (Devices)

-  A device special file corresponds to a device on the system. Within the kernel, each device type has a corresponding device driver, which handles all I/O requests for the device. A device driver is a unit of kernel code that implements a set of operations that (normally) correspond to input and output actions on an associated piece of hardware. The API provided by device drivers is fixed, and includes operations corresponding to the system calls open(), close(), read(), write(), mmap(), and ioctl(). The fact that each device driver provides a consistent interface, hiding the differences in operation of individual devices, allows for universality of I/O operations.
-  Device files appear within the file system, just like other files, usually under the /dev directory.

## 14.2 Disks and Partitions

Regular files and directories typically reside on hard disk devices.

### Disk Drives

- A hard disk drive is a mechanical device consisting of one or more platters that rotate at high speed (of the order of thousands of revolutions per minute). Magnetically encoded information on the disk surface is retrieved or modified by read/write heads that move radially across the disk. Physically, information on the disk surface is located on a set of concentric circles called tracks. Tracks themselves are divided into a number of sectors, each of which consists of a series of physical blocks. Physical blocks are typically 512 bytes (or some multiple thereof) in size, and represent the smallest unit of information that the drive can read or write.
- Although modern disks are fast, reading and writing information on the disk still takes significant time. The disk head must first move to the appropriate track (seek time), then the drive must wait until the appropriate sector rotates under the head (rotational latency), and finally the required blocks must be transferred (transfer time). The total time required to carry out such an operation is typically of the order of milliseconds. By comparison, modern CPUs are capable of executing millions of instructions in this time.

### Disk Partitions
- Each disk is divided into one or more (nonoverlapping) partitions. Each partition is treated by the kernel as a separate device residing under the /dev directory.
- A disk partition may hold any type of information, but usually contains one of the following:
  - a file system holding regular files and directories, as described in Section 14.3;
  - a data area accessed as a raw-mode device, as described in Section 13.6 (some database management systems use this technique); or
  - a swap area used by the kernel for memory management

## 14.3 File Systems
- A file system is an organized collection of regular files and directories. A file system is created using the mkfs command.

### File-system structure
- The basic unit for allocating space in a file system is a logical block, which is some multiple of contiguous physical blocks on the disk device on which the file system resides.
- A file system contains the following parts:
  - Boot block: This is always the first block in a file system. The boot block is not used by the file system; rather, it contains information used to boot the operating system. Although only one boot block is needed by the operating system, all file systems have a boot block (most of which are unused).
  - Superblock: This is a single block, immediately following the boot block, which contains parameter information about the file system, including:
    – the size of the i-node table;
    – the size of logical blocks in this file system; and
    – the size of the file system in logical blocks.
    - Different file systems residing on the same physical device can be of different types and sizes, and have different parameter settings (e.g., block size). This is one of the reasons for splitting a disk into multiple partitions.
  - I-node table: Each file or directory in the file system has a unique entry in the i-node table. This entry records various information about the file. I-nodes are discussed in greater detail in the next section. The i-node table is sometimes also called the i-list.
  - Data blocks: The great majority of space in a file system is used for the blocks of data that form the files and directories residing in the file system.

## 14.4 I-nodes
A file system’s i-node table contains one i-node (short for index node) for each file residing in the file system. I-nodes are identified numerically by their sequential location in the i-node table. The i-node number (or simply i-number) of a file is the first field displayed by the ls –li command. The information maintained in an i-node includes the following:
- File type (e.g., regular file, directory, symbolic link, character device).
- Owner (also referred to as the user ID or UID) for the file.
- Group (also referred to as the group ID or GID) for the file.
- Access permissions for three categories of user: owner (sometimes referred to as user), group, and other (the rest of the world)
- Three timestamps: time of last access to the file (shown by ls –lu), time of last modification of the file (the default time shown by ls –l), and time of last status change (last change to i-node information, shown by ls –lc). As on other UNIX implementations, it is notable that most Linux file systems don’t record the creation time of a file.
- Number of hard links to the file.
- Size of the file in bytes.
- Number of blocks actually allocated to the file, measured in units of 512-byte blocks. There may not be a simple correspondence between this number and the size of the file in bytes, since a file can contain holes (Section 4.7), and thus require fewer allocated blocks than would be expected according to its nominal size in bytes.
- Pointers to the data blocks of the file.

### I-node and data block pointers in ext2

- Like most UNIX file systems, the ext2 file system doesn’t store the data blocks of a file contiguously or even in sequential order (though it does attempt to store them close to one another). To locate the file data blocks, the kernel maintains a set of pointers in the i-node.
- Under ext2, each i-node contains 15 pointers.
    - The first 12 of these pointers (numbered 0 to 11 in Figure 14-2) point to the location in the file system of the first 12 blocks of the file.
    - The next pointer is a pointer to a block of pointers that give the locations of the thirteenth and subsequent data blocks of the file. The number of pointers in this block depends on the block size of the file system. Each pointer requires 4 bytes, so there may be from 256 pointers (for a 1024-byte block size) to 1024 pointers (for a 4096-byte block size). This allows for quite large files.
    - For even larger files, the fourteenth pointer (numbered 13 in the diagram) is a double indirect pointer—it points to blocks of pointers that in turn point to blocks of pointers that in turn point to data blocks of the file.
    - And should the need for a truly enormous file arise, there is a further level of indirection: the last pointer in the i-node is a triple-indirect pointer.
- This seemingly complex system is designed to satisfy a number of requirements.
    - To begin with, it allows the i-node structure to be a fixed size, while at the same time allowing for files of an arbitrary size.
    - Additionally, it allows the file system to store the blocks of a file noncontiguously, while also allowing the data to be accessed randomly via lseek(); the kernel just needs to calculate which pointer(s) to follow.
    - Finally, for small files, which form the overwhelming majority of files on most systems, this scheme allows the file data blocks to be accessed rapidly via the direct pointers of the i-node.

## 14.5 The Virtual File System (VFS)
- Each of the file systems available on Linux differs in the details of its implementation. Such differences include, for example, the way in which the blocks of a file are allocated and the manner in which directories are organized. If every program that worked with files needed to understand the specific details of each file system, the task of writing programs that worked with all of the different file systems would be nearly impossible.
- The virtual file system (VFS, sometimes also referred to as the
virtual file switch) is a kernel feature that resolves this problem by creating an abstraction layer for file-system operations. VFS goals:
    - The VFS defines a generic interface for file-system operations.
    - All programs that work with files specify their operations in terms of this generic interface.

| Under this scheme, programs need to understand only the VFS interface and can ignore details of individual file-system implementations.

- The VFS interface includes operations corresponding to all of the usual system calls for working with file systems and directories, such as open(), read(), write(), lseek(), close(), truncate(), stat(), mount(), umount(), mmap(), mkdir(), link(), unlink(), symlink(), and rename().

## 14.6 Journaling File Systems

- The ext2 file system is a good example of a traditional UNIX file system, and suffers from a classic limitation of such file systems: after a system crash, a file-system consistency check ( fsck) must be performed on reboot in order to ensure the integrity of the file system. This is necessary because, at the time of the system crash, a file update may have been only partially completed, and the file-system metadata (directory entries, i-node information, and file data block pointers) may be in an inconsistent state, so that the file system might be further damaged if these inconsistencies are not repaired. A file-system consistency check ensures the consistency of the file-system metadata. Where possible, repairs are performed; otherwise, information that is not retrievable (possibly including file data) is discarded.
- The problem is that a consistency check requires examining the entire file system. On a small file system, this may take anything from several seconds to a few minutes. On a large file system, this may require several hours, which is a serious problem for systems that must maintain high availability (e.g., network servers).
- Journaling file systems eliminate the need for lengthy file-system consistency checks after a system crash. A journaling file system logs (journals) all metadata updates to a special on-disk journal file before they are actually carried out. The updates are logged in groups of related metadata updates (transactions). In the event of a system crash in the middle of a transaction, on system reboot, the log can be used to rapidly redo any incomplete updates and bring the file system back to a consistent state. (To borrow database parlance, we can say that a journaling file system ensures that file metadata transactions are always committed as a complete unit.) Even very large journaling file systems can typically be available within seconds after a system crash, making them very attractive for systems with highavailability requirements.
- The most notable disadvantage of journaling is that it adds time to file updates, though good design can make this overhead low.
    - The ext3 file system was a result of a project to add journaling to ext2 with minimal impact. The migration path from ext2 to ext3 is very easy (no backup and restore are required), and it is possible to migrate in the reverse direction as well.
    - The ext4 file system (http://ext4.wiki.kernel.org/) is the successor to ext3. The first pieces of the implementation were added in kernel 2.6.19, and various features were added in later kernel versions. Among the planned (or already implemented) features for ext4 are extents (reservation of contiguous blocks of storage) and other allocation features that aim to reduce file fragmentation, online file-system defragmentation, faster file-system checking, and support for nanosecond timestamps.

## 14.7 Single Directory Hierarchy and Mount Points
- On Linux, as on other UNIX systems, all files from all file systems reside under a single directory tree. At the base of this tree is the root directory, / (slash). Other file systems are mounted under the root directory and appear as subtrees within the overall hierarchy.
- `mount device directory`: This command attaches the file system on the named device into the directory hierarchy at the specified directory—the file system’s mount point.

``` bash
$ mount
/dev/sda6 on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,mode=0620,gid=5)
/dev/sda8 on /home type ext3 (rw,acl,user_xattr)
/dev/sda1 on /windows/C type vfat (rw,noexec,nosuid,nodev)
/dev/sda9 on /home/mtk/test type reiserfs (rw)
```

- Figure 14-4 shows a partial directory and file structure for the system on which the above mount command was performed. This diagram shows how the mount points map onto the directory hierarchy.
    - Shows that the `sda6` file system in the `/dev/sda6` partition is mounted at the root directory (/).
    - `/home`, `/windows`, and `/home/mtk/test` are mount points for the file systems residing on the `sda8`, `sda1`, and `sda9` partitions, respectively.
    - This shows that within a file system, you can have mount points for other file systems.

## 14.8 Mounting and Unmounting File Systems
- A list of the currently mounted file systems can be read from the Linux-specific `/proc/mounts` virtual file. `//proc/mounts` is an interface to kernel data structures, so it always contains accurate information about mounted file systems.

``` bash
shad@linux:~/linux/notes/lpi$ cat /proc/mounts 
/dev/root / ext4 rw,relatime,discard,errors=remount-ro,commit=30 0 0
```
1. The name of the mounted device.
2. The mount point for the device.
3. The file-system type.
4. Mount flags. In the above example, rw indicates that the file system was mounted read-write.
5. A number used to control the operation of file-system backups by dump(8). This field and the next are used only in the /etc/fstab file; for /proc/mounts and /etc/mtab, these fields are always 0.
6. A number used to control the order in which fsck(8) checks file systems at system boot time.

## 14.10 A Virtual Memory File System: tmpfs

- All of the file systems we have described so far in this chapter reside on disks. However, Linux also supports the notion of virtual file systems that reside in memory. To applications and users, these file systems look just like any other file system—the same operations (open(), read(), write(), link(), mkdir(), and so on) can be applied to files and directories in such file systems. There is, however, one important difference: file operations are much faster, since no disk access is involved.
- The tmpfs file system differs from other memory-based file systems in that it is a virtual memory file system. This means that tmpfs uses not only RAM, but also the swap space, if RAM is exhausted.
- By default, a tmpfs file system is permitted to grow to half the size of RAM, but the size=nbytes mount option can be used to set a different ceiling for the file-system size, either when the file system is created or during a later remount. (A tmpfs file system consumes only as much memory and swap space as is currently required for the files it holds.)
- If we unmount a tmpfs file system, or the system crashes, then all data in the file system is lost; hence the name tmpfs.
- A tmpfs file system mounted at /dev/shm is used for the glibc implementation of POSIX shared memory and POSIX semaphores.

## 14.12 Summary

- Devices are represented by entries in the /dev directory. Each device has a corresponding device driver, which implements a standard set of operations, including those corresponding to the open(), read(), write(), and close() system calls. A device may be real, meaning that there is a corresponding hardware device, or virtual, meaning that no hardware device exists, but the kernel nevertheless provides a device driver that implements an API that is the same as a real device.
- A hard disk is divided into one or more partitions, each of which may contain a file system. A file system is an organized collection of regular files and directories. Linux implements a wide variety of file systems, including the traditional ext2 file system. The ext2 file system is conceptually similar to early UNIX file systems, consisting of a boot block, a superblock, an i-node table, and a data area containing file data blocks. Each file has an entry in the file system’s i-node table. This entry contains various information about the file, including its type, size, link count, ownership,
permissions, timestamps, and pointers to the file’s data blocks.
- Linux provides a range of journaling file systems, including Reiserfs, ext3, ext4, XFS, JFS, and Btrfs. A journaling file system records metadata updates (and optionally on some file systems, data updates) to a log file before the actual file updates are performed. This means that in the event of a system crash, the log file can be
replayed to quickly restore the file system to a consistent state. The key benefit of journaling file systems is that they avoid the lengthy file-system consistency checks required by conventional UNIX file systems after a system crash.
- All file systems on a Linux system are mounted under a single directory tree, with the directory / at its root. The location at which a file system is mounted in the
directory tree is called its mount point.
A privileged process can mount and unmount a file system using the mount() and umount() system calls. Information about a mounted file system can be retrieved using statvfs() system call.