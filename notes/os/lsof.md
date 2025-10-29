# lsof

[ref](https://www.youtube.com/watch?v=n9nZ1ellaV0)

How to find out which files are open on your system, and open by whom/what in particular.

lsof = list open files command.

Why would you use this?
- A common example is error message like "can't delete file because it's in use."
- Another is a security issue. You have a process running on your server, and you're not sure what that process is doing. You can investigate the open files for that process to understand what files it's working with to guage whether it's malicious or not.

``` bash
shad@linux:~/linux/notes/os$ lsof | head
COMMAND     PID   TID TASKCMD              USER   FD      TYPE             DEVICE  SIZE/OFF   NODE NAME
systemd       1                            root  cwd   unknown                                     /proc/1/cwd (readlink: Permission denied)
```

- The job of the init system is to schedule commands.
- COMMAND: The command associated with the open file.
- PID: The process that has the file open.
- USER: Which user created the process that has the file open.

``` bash
shad@linux:~/linux/notes/os$ lsof | wc -l
4264
shad@linux:~/linux/notes/os$ sudo lsof | wc -l
9853
```
- Above shows which files are open on the system. Note that running as root shows more files, because some files are restricted to root access.

## What a process has open in terms of files.

``` bash

# apache2 has ~138 files open
shad@linux:~/linux/notes/os$ sudo lsof -c apache2 | wc -l
138

# By the way, `grep -E "HEADER_PATTERN|SEARCH_PATTERN"` is a great way to find what you're looking for, and include the header line!!!
shad@linux:~/linux/notes/os$ sudo lsof -c apache2 | grep -E "COMMAND|log"
COMMAND   PID     USER   FD      TYPE DEVICE SIZE/OFF   NODE NAME
apache2  6119     root    2w      REG    8,1      279 266563 /var/log/apache2/error.log
apache2  6119     root    7w      REG    8,1        0 266304 /var/log/apache2/other_vhosts_access.log
apache2  6119     root    8w      REG    8,1        0 266302 /var/log/apache2/access.log
apache2 23437 www-data    2w      REG    8,1      279 266563 /var/log/apache2/error.log
apache2 23437 www-data    7w      REG    8,1        0 266304 /var/log/apache2/other_vhosts_access.log
apache2 23437 www-data    8w      REG    8,1        0 266302 /var/log/apache2/access.log
apache2 23438 www-data    2w      REG    8,1      279 266563 /var/log/apache2/error.log
apache2 23438 www-data    7w      REG    8,1        0 266304 /var/log/apache2/other_vhosts_access.log
apache2 23438 www-data    8w      REG    8,1        0 266302 /var/log/apache2/access.log

# To filter by PID.
shad@linux:~/linux/notes/os$ sudo lsof -p 6119

# To see which processes have a particular file or folder open.
shad@linux:~/linux/notes/os$ sudo lsof /var/log/apache2/*
COMMAND   PID     USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
apache2  6119     root    2w   REG    8,1      279 266563 /var/log/apache2/error.log
apache2  6119     root    7w   REG    8,1        0 266304 /var/log/apache2/other_vhosts_access.log
apache2  6119     root    8w   REG    8,1        0 266302 /var/log/apache2/access.log
apache2 23437 www-data    2w   REG    8,1      279 266563 /var/log/apache2/error.log
apache2 23437 www-data    7w   REG    8,1        0 266304 /var/log/apache2/other_vhosts_access.log
apache2 23437 www-data    8w   REG    8,1        0 266302 /var/log/apache2/access.log
apache2 23438 www-data    2w   REG    8,1      279 266563 /var/log/apache2/error.log
apache2 23438 www-data    7w   REG    8,1        0 266304 /var/log/apache2/other_vhosts_access.log
apache2 23438 www-data    8w   REG    8,1        0 266302 /var/log/apache2/access.log

# To see which processes are using files not owned by root.
shad@linux:~/linux/notes/os$ sudo lsof -u ^root

# 
shad@linux:~/linux/notes/os$ sudo lsof -i 4
COMMAND     PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd       1            root  189u  IPv4   8261      0t0  TCP *:ssh (LISTEN)
systemd-r   506 systemd-resolve   14u  IPv4   6291      0t0  UDP _localdnsstub:domain 
systemd-r   506 systemd-resolve   15u  IPv4   6292      0t0  TCP _localdnsstub:domain (LISTEN)
systemd-r   506 systemd-resolve   16u  IPv4   6293      0t0  UDP _localdnsproxy:domain 
systemd-r   506 systemd-resolve   17u  IPv4   6294      0t0  TCP _localdnsproxy:domain (LISTEN)
systemd-n   693 systemd-network   21u  IPv4   7741      0t0  UDP linux.internal.cloudapp.net:bootpc 
chronyd     927         _chrony    6u  IPv4   9431      0t0  UDP localhost:323 
sshd       1137            root    3u  IPv4   8261      0t0  TCP *:ssh (LISTEN)
sshd      10434            root    4u  IPv4  60114      0t0  TCP linux.internal.cloudapp.net:ssh->pool-108-28-[redacted].washdc.fios.verizon.net:60891 (ESTABLISHED)
sshd      10493            shad    4u  IPv4  60114      0t0  TCP linux.internal.cloudapp.net:ssh->pool-108-28-[redacted].washdc.fios.verizon.net:60891 (ESTABLISHED)
sshd      10493            shad    8u  IPv4  60257      0t0  TCP localhost:60962->localhost:37757 (ESTABLISHED)
code-7d84 10512            shad    9u  IPv4  59331      0t0  TCP localhost:37757 (LISTEN)
code-7d84 10512            shad   12u  IPv4  59350      0t0  TCP localhost:37757->localhost:60962 (ESTABLISHED)
node      10535            shad   44u  IPv4 172563      0t0  TCP linux.internal.cloudapp.net:34916->lb-140-82-112-22-iad.github.com:https (ESTABLISHED)
node      10535            shad   50u  IPv4 172850      0t0  TCP linux.internal.cloudapp.net:36440->20.85.[redacted]:https (ESTABLISHED)
node      10535            shad   53u  IPv4 173681      0t0  TCP linux.internal.cloudapp.net:36446->20.85.[redacted]:https (ESTABLISHED)
node      10535            shad   64u  IPv4 174139      0t0  TCP linux.internal.cloudapp.net:58740->lb-140-82-113-21-iad.github.com:https (ESTABLISHED)

# Show port numbers and don't resolve hostnames.
shad@linux:~/linux/notes/os$ sudo lsof -i 4 -P -n | grep sshd
sshd       1137            root    3u  IPv4   8261      0t0  TCP *:22 (LISTEN)
sshd      10434            root    4u  IPv4  60114      0t0  TCP 10.0.0.4:22->108.28.[redacted]:60891 (ESTABLISHED)
sshd      10493            shad    4u  IPv4  60114      0t0  TCP 10.0.0.4:22->108.28.[redacted]:60891 (ESTABLISHED)
sshd      10493            shad    8u  IPv4  60257      0t0  TCP 127.0.0.1:60962->127.0.0.1:37757 (ESTABLISHED)
```