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