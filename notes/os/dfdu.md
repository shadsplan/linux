# df and du
[ref](https://www.youtube.com/watch?v=ZRs5zVv_1UU)

## df - disk free

- Information pertaining to how much disk space you have free.

``` bash
shad@linux:~/linux/notes/os$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        29G  2.6G   26G  10% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           1.6G 1016K  1.6G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
efivarfs        128K   37K   87K  30% /sys/firmware/efi/efivars
/dev/sda16      881M  110M  710M  14% /boot
/dev/sda15      105M  6.2M   99M   6% /boot/efi
/dev/sdb1        16G   28K   15G   1% /mnt
tmpfs           790M   12K  790M   1% /run/user/1000
```

- `df -h` for human readable format (shows in MB/GB/TB instead of bytes). Always use this flag.
- `/dev/root        29G  2.6G   26G  10% /`
    - `/` is the root file system, the beginning of the linux installation.
- Whatever storage devices I have attached to my server, they'll show up here.

``` bash
# Shows the file system type
shad@linux:~/linux/notes/os$ df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/root      ext4       29G  2.6G   26G  10% /

# tmpfs is a temporary file system that lives in RAM. Useful for storing temporary files that don't need to be persisted.
# we're not really interested in tmpfs for right now, so we can exclude it with -x flag
shad@linux:~/linux/notes/os$ df -hT -x tmpfs
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/root      ext4       29G  2.6G   26G  10% /
efivarfs       efivarfs  128K   37K   87K  30% /sys/firmware/efi/efivars
/dev/sda16     ext4      881M  110M  710M  14% /boot
/dev/sda15     vfat      105M  6.2M   99M   6% /boot/efi
/dev/sdb1      ext4       16G   28K   15G   1% /mnt
```

| This does not show us how much space is used on a folder or directory level. If the root fs is full, we'd like to know which folders are taking up the most space. This is why we use `du`.

## du - disk usage

``` bash

# Our home directory is using 466MB of space. 
shad@linux:~/linux/notes/os$ du -h /home/shad/ | tail -n 5
44M     /home/shad/linux
8.0K    /home/shad/.config/htop
4.0K    /home/shad/.config/procps
16K     /home/shad/.config
466M    /home/shad/

# Controls how many directories deep the du command goes.
shad@linux:~/linux/notes/os$ du -h --max-depth 1 /home/shad
12K     /home/shad/.local
20K     /home/shad/folder1
422M    /home/shad/.vscode-server
16K     /home/shad/.cache
24K     /home/shad/.ssh
16K     /home/shad/root
224K    /home/shad/.dotnet
44M     /home/shad/linux
16K     /home/shad/.config
466M    /home/shad

# To get a summary of multiple folders. Helps to compare sizes between folders.
shad@linux:~/linux/notes/os$ sudo du -hs /home/shad /etc
466M    /home/shad
6.9M    /etc

# To get a total size as well.
shad@linux:~/linux/notes/os$ sudo du -hsc /home/shad /etc
466M    /home/shad
6.9M    /etc
473M    total

# To see sizes of all items in a folder. Note: this doesn't contain hidden files/folders (those that start with a dot).
shad@linux:~/linux/notes/os$ sudo du -hsc /home/shad/*
20K     /home/shad/folder1
4.0K    /home/shad/hello.txt
44M     /home/shad/linux
16K     /home/shad/root
44M     total

shad@linux:~/linux/notes/os$ sudo du -hsc /home/shad/* /home/shad/.*
20K     /home/shad/folder1
4.0K    /home/shad/hello.txt
44M     /home/shad/linux
16K     /home/shad/root
16K     /home/shad/.bash_history
4.0K    /home/shad/.bash_logout
4.0K    /home/shad/.bashrc
16K     /home/shad/.cache
16K     /home/shad/.config
224K    /home/shad/.dotnet
4.0K    /home/shad/.gitconfig
4.0K    /home/shad/.lesshst
12K     /home/shad/.local
4.0K    /home/shad/.profile
4.0K    /home/shad/.python_history
24K     /home/shad/.ssh
0       /home/shad/.sudo_as_admin_successful
# culprit 👀👀👀
423M    /home/shad/.vscode-server
4.0K    /home/shad/.wget-hsts
466M    total
```

- `ncdu` allows you to navigate through folders and see sizes in a more interactive way. Not installed by default.

``` bash
# How to know which file system you're using
shad@linux:~/linux/notes/lpi$ mount | grep 'on / '
/dev/sda1 on / type ext4 (rw,relatime,discard,errors=remount-ro,commit=30)

# More in depth:

# Note the PARTUUID in the output below
shad@linux:~/linux/notes/lpi$ cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-6.14.0-1014-azure root=PARTUUID=625cad27-7b63-4e5e-8bbd-6efc33b38415 ro console=tty1 console=ttyS0 earlyprintk=ttyS0 nvme_core.io_timeout=240 panic=-1

# We see that 625cad27-7b63-4e5e-8bbd-6efc33b38415 is the PARTUUID of our root file system.
# It is located on /dev/sda1 as we saw in the mount command above and has a type of ext4.
# These are all the 
shad@linux:~/linux/notes/lpi$ sudo blkid
/dev/sdb1: UUID="aa9c62c5-36b5-4be9-acdd-b4267069b28e" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="76ec78d5-01"
/dev/sda16: LABEL="BOOT" UUID="4e0b0f7d-f018-493d-b384-d56a4d4661bd" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="bc888243-c7df-4555-b820-0fa1621cf3e0"
/dev/sda15: LABEL_FATBOOT="UEFI" LABEL="UEFI" UUID="B0C0-7511" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="d403963f-6b02-4be2-96ef-428181705f25"
/dev/sda1: LABEL="cloudimg-rootfs" UUID="fad0ebd7-fc6c-42fe-a34d-4e8d2f07946f" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="625cad27-7b63-4e5e-8bbd-6efc33b38415"
/dev/sda14: PARTUUID="a9d448c0-a37e-4b4c-8190-cf5a5200263f"

# Nice play here:
# Tells us which disks (storage devices) and partitions we have, their sizes, and where they are mounted.
shad@linux:~/linux/notes/lpi$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda       8:0    0   30G  0 disk 
├─sda1    8:1    0   29G  0 part /
├─sda14   8:14   0    4M  0 part 
├─sda15   8:15   0  106M  0 part /boot/efi
└─sda16 259:0    0  913M  0 part /boot
sdb       8:16   0   16G  0 disk 
└─sdb1    8:17   0   16G  0 part /mnt

# Which file systems are mounted where, their types, and mount options.
shad@linux:~/linux/notes/lpi$ findmnt
TARGET                         SOURCE      FSTYPE      OPTIONS
/                              /dev/sda1   ext4        rw,relatime,discard,errors=remount-ro,commit=30
├─/dev                         devtmpfs    devtmpfs    rw,nosuid,noexec,relatime,size=4036848k,nr_inodes=1009212,mode=755,inode64
│ ├─/dev/shm                   tmpfs       tmpfs       rw,nosuid,nodev,inode64
│ ├─/dev/pts                   devpts      devpts      rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000
│ ├─/dev/hugepages             hugetlbfs   hugetlbfs   rw,nosuid,nodev,relatime,pagesize=2M
│ └─/dev/mqueue                mqueue      mqueue      rw,nosuid,nodev,noexec,relatime
├─/proc                        proc        proc        rw,nosuid,nodev,noexec,relatime
│ └─/proc/sys/fs/binfmt_misc   systemd-1   autofs      rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=714
│   └─/proc/sys/fs/binfmt_misc binfmt_misc binfmt_misc rw,nosuid,nodev,noexec,relatime
├─/sys                         sysfs       sysfs       rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/security       securityfs  securityfs  rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup             cgroup2     cgroup2     rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot
│ ├─/sys/fs/pstore             pstore      pstore      rw,nosuid,nodev,noexec,relatime
│ ├─/sys/firmware/efi/efivars  efivarfs    efivarfs    rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/bpf                bpf         bpf         rw,nosuid,nodev,noexec,relatime,mode=700
│ ├─/sys/kernel/debug          debugfs     debugfs     rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/tracing        tracefs     tracefs     rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/fuse/connections   fusectl     fusectl     rw,nosuid,nodev,noexec,relatime
│ └─/sys/kernel/config         configfs    configfs    rw,nosuid,nodev,noexec,relatime
├─/run                         tmpfs       tmpfs       rw,nosuid,nodev,size=1616532k,nr_inodes=819200,mode=755,inode64
│ ├─/run/lock                  tmpfs       tmpfs       rw,nosuid,nodev,noexec,relatime,size=5120k,inode64
│ └─/run/user/1000             tmpfs       tmpfs       rw,nosuid,nodev,relatime,size=808264k,nr_inodes=202066,mode=700,uid=1000,gid=1000,inode64
├─/boot                        /dev/sda16  ext4        rw,relatime,discard
│ └─/boot/efi                  /dev/sda15  vfat        rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=rem
└─/mnt                         /dev/sdb1   ext4        rw,relatime



```