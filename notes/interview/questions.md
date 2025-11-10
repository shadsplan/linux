# Interview Questions

[Top 10 Linux Job Interview Questions](https://www.youtube.com/watch?v=l0QGLMwR-lY)

1. **How can you see which kernel version a system is currently running?**

``` bash
shad@linux:~/linux/notes/interview$ uname -a

# Hostname, OS, Kernel, Release, Kernel Version, Architecture
Linux linux 6.14.0-1014-azure #14~24.04.1-Ubuntu SMP Fri Oct  3 20:52:11 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

2. **How to see the current IP address on Linux?**

``` bash
shad@linux:~/linux/notes/interview$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 60:45:bd:ef:f3:4d brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6245:bdff:feef:f34d/64 scope link 
       valid_lft forever preferred_lft forever
3: enP43014s1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc mq master eth0 state UP group default qlen 1000
    link/ether 60:45:bd:ef:f3:4d brd ff:ff:ff:ff:ff:ff
    altname enP43014p0s2
    inet6 fe80::6245:bdff:feef:f34d/64 scope link 
       valid_lft forever preferred_lft forever

shad@linux:~/linux/notes/interview$ curl ifconfig.me
20.121.121.163
```

3. **How to check for free disk space in Linux?**

``` bash

# all filesystems, human readable
shad@linux:~/linux/notes/interview$ df -ah

Filesystem      Size  Used Avail Use% Mounted on
/dev/root        29G  2.9G   26G  11% /
devtmpfs        3.9G     0  3.9G   0% /dev
proc               0     0     0    - /proc
sysfs              0     0     0    - /sys
securityfs         0     0     0    - /sys/kernel/security
tmpfs           3.9G     0  3.9G   0% /dev/shm
devpts             0     0     0    - /dev/pts
tmpfs           1.6G 1008K  1.6G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
cgroup2            0     0     0    - /sys/fs/cgroup
pstore             0     0     0    - /sys/fs/pstore
efivarfs        128K   37K   87K  30% /sys/firmware/efi/efivars
bpf                0     0     0    - /sys/fs/bpf
systemd-1          -     -     -    - /proc/sys/fs/binfmt_misc
hugetlbfs          0     0     0    - /dev/hugepages
debugfs            0     0     0    - /sys/kernel/debug
mqueue             0     0     0    - /dev/mqueue
tracefs            0     0     0    - /sys/kernel/tracing
fusectl            0     0     0    - /sys/fs/fuse/connections
configfs           0     0     0    - /sys/kernel/config
/dev/sda16      881M  110M  710M  14% /boot
/dev/sda15      105M  6.2M   99M   6% /boot/efi
binfmt_misc        0     0     0    - /proc/sys/fs/binfmt_misc
/dev/sdb1        16G   28K   15G   1% /mnt
tmpfs           790M   12K  790M   1% /run/user/1000
```

5. **How to see if a Linux service is running?**

``` bash
shad@linux:~/linux/notes/interview$ systemctl status apache2
○ apache2.service - Apache is chill
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; disabled; preset: enabled)
    Drop-In: /etc/systemd/system/apache2.service.d
             └─override.conf
     Active: inactive (dead)
       Docs: https://httpd.apache.org/docs/2.4/
```

6. **How to check the size of a directory in Linux?**

``` bash
shad@linux:~/linux$ du -h notes/
20K     notes/systemd
8.0K    notes/interview
32K     notes/os
16K     notes/networking
180K    notes/lpi
260K    notes/
shad@linux:~/linux$ du -hs notes/
260K    notes/
```

7. **How to check for open ports in Linux?**

``` bash
# n = numeric, p = process, a = all, t = tcp, u = udp
shad@linux:~/linux$ sudo ss -tupna
Netid  State   Recv-Q  Send-Q   Local Address:Port     Peer Address:Port   Process                                                                                 
udp    UNCONN  0       0           127.0.0.54:53            0.0.0.0:*       users:(("systemd-resolve",pid=532,fd=16))                                              
udp    UNCONN  0       0        127.0.0.53%lo:53            0.0.0.0:*       users:(("systemd-resolve",pid=532,fd=14))                                              
udp    UNCONN  0       0        10.0.0.4%eth0:68            0.0.0.0:*       users:(("systemd-network",pid=694,fd=21))                                              
udp    UNCONN  0       0            127.0.0.1:323           0.0.0.0:*       users:(("chronyd",pid=921,fd=6))                                                       
udp    UNCONN  0       0                [::1]:323              [::]:*       users:(("chronyd",pid=921,fd=7))                                                       
tcp    LISTEN  0       4096        127.0.0.54:53            0.0.0.0:*       users:(("systemd-resolve",pid=532,fd=17))                                              
tcp    LISTEN  0       4096     127.0.0.53%lo:53            0.0.0.0:*       users:(("systemd-resolve",pid=532,fd=15))                                              
tcp    LISTEN  0       1024         127.0.0.1:33999         0.0.0.0:*       users:(("code-7d842fb85a",pid=1276,fd=9))                                              
tcp    LISTEN  0       4096           0.0.0.0:22            0.0.0.0:*       users:(("sshd",pid=1120,fd=3),("systemd",pid=1,fd=55))                                 
tcp    ESTAB   0       0            127.0.0.1:55432       127.0.0.1:33999   users:(("sshd",pid=1255,fd=8))                                                         
tcp    ESTAB   0       0            127.0.0.1:33999       127.0.0.1:55432   users:(("code-7d842fb85a",pid=1276,fd=12))                                             
tcp    ESTAB   0       0             10.0.0.4:54142   20.85.130.105:443     users:(("node",pid=1420,fd=40))                                                        
tcp    ESTAB   0       0             10.0.0.4:54154   20.85.130.105:443     users:(("node",pid=1420,fd=63))                                                        
tcp    ESTAB   0       316           10.0.0.4:22       108.28.[REDACTED]:51424   users:(("sshd",pid=1255,fd=4),("sshd",pid=1121,fd=4))                                  
tcp    ESTAB   0       0             10.0.0.4:51976   140.82.112.22:443     users:(("node",pid=1420,fd=38))                                                        
tcp    LISTEN  0       511                  *:80                  *:*       users:(("apache2",pid=3129,fd=4),("apache2",pid=3128,fd=4),("apache2",pid=3125,fd=4))  
tcp    LISTEN  0       4096              [::]:22               [::]:*       users:(("sshd",pid=1120,fd=4),("systemd",pid=1,fd=56))                                 
```

8. **How to check Linux process information (CPU usage, memory, user information, etc.)?**

``` bash
shad@linux:~/linux$ ps aux | grep apache2
root        3125  0.0  0.0   6880  4920 ?        Ss   15:15   0:00 /usr/sbin/apache2 -k start
www-data    3128  0.0  0.0 1212964 5524 ?        Sl   15:15   0:00 /usr/sbin/apache2 -k start
www-data    3129  0.0  0.0 1212964 5544 ?        Sl   15:15   0:00 /usr/sbin/apache2 -k start
shad        3513  0.0  0.0   7076  2200 pts/0    S+   15:20   0:00 grep --color=auto apache2

# Can sort by memory or CPU usage.
shad@linux:~/linux$ top
shad@linux:~/linux$ htop
```

9. **How to deal with mounts in Linux?**

- Note: Not super relevant, not a ton of devices that get mounted once they get booted.

``` bash

# /mnt/ directory is where we mount things
shad@linux:~/linux$ ls /mnt/
DATALOSS_WARNING_README.txt  lost+found

# Absolute path to volume (ex. 2nd partition on sda hard drive), mount point
shad@linux:~/linux$ mount /dev/sda2 /mnt/

# Checking for existing mounts. See what's mounted, and what type of filesystem.
shad@linux:~/linux$ mount
/dev/sda1 on / type ext4 (rw,relatime,discard,errors=remount-ro,commit=30)
devtmpfs on /dev type devtmpfs (rw,nosuid,noexec,relatime,size=4036848k,nr_inodes=1009212,mode=755,inode64)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,size=1616532k,nr_inodes=819200,mode=755,inode64)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k,inode64)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
efivarfs on /sys/firmware/efi/efivars type efivarfs (rw,nosuid,nodev,noexec,relatime)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=843)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,nosuid,nodev,relatime,pagesize=2M)
debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime)
fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
/dev/sda16 on /boot type ext4 (rw,relatime,discard)
/dev/sda15 on /boot/efi type vfat (rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,nosuid,nodev,noexec,relatime)
/dev/sdb1 on /mnt type ext4 (rw,relatime,x-systemd.after=cloud-init.service,_netdev)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=808264k,nr_inodes=202066,mode=700,uid=1000,gid=1000,inode64)

# If you need to mount a volume at boot, which file would you look in?

shad@linux:~/linux$ cat /etc/fstab
# CLOUD_IMG: This file was created/modified by the Cloud Image build process
UUID=fad0ebd7-fc6c-42fe-a34d-4e8d2f07946f       /        ext4   discard,commit=30,errors=remount-ro     0 1
LABEL=BOOT      /boot   ext4    defaults,discard        0 2
UUID=B0C0-7511  /boot/efi       vfat    umask=0077      0 1
/dev/disk/cloud/azure_resource-part1    /mnt    auto    defaults,nofail,x-systemd.after=cloud-init.service,_netdev,comment=cloudconfig  0       2
```

[Popular Linux Interview Questions for DevOps Interviews](https://www.youtube.com/watch?v=GdrjTVDelI0&t=2s)

(basics)
1. **How would you connect to a machine securely through a shell?**

`ssh -i C:\Users\shad\.ssh\shad_key.pem shad@20.121.121.163`

2. **How would you copy files from a remote server to your local machine securely?**

``` bash
PS C:\Users\shad\Downloads> scp -i C:\Users\shad\.ssh\shad_key.pem shad@20.121.121.163:/home/shad/linux/README.md C:\Users\shad\Downloads
README.md                                                                                                                               100%  313     6.2KB/s   00:00
```

3. **How would you manage services on this box?**

``` bash
shad@linux:~/linux/notes/interview$ sudo systemctl start apache2
had@linux:~/linux$ curl localhost
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <!--
    Modified from the Debian original for Ubuntu
    Last updated: 2022-03-22
    See: https://launchpad.net/bugs/1966004
  -->
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Apache2 Ubuntu Default Page: It works</title>
    <style type="text/css" media="screen">
  * {
    margin: 0px 0px 0px 0px;
    padding: 0px 0px 0px 0px;
  }
```

4. **Difference between enable and start for systemctl?**
- `start` will start the service immediately but won't enable it to start on boot.
- `enable` will configure the service to start automatically on boot but won't start it immediately.

5. **How would you how much disk space on the file system is taken up?**

``` bash
had@linux:~/linux$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        29G  2.9G   26G  11% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           1.6G 1012K  1.6G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
efivarfs        128K   37K   87K  30% /sys/firmware/efi/efivars
/dev/sda16      881M  110M  710M  14% /boot
/dev/sda15      105M  6.2M   99M   6% /boot/efi
/dev/sdb1        16G   28K   15G   1% /mnt
tmpfs           790M   12K  790M   1% /run/user/1000
```

6. **How would you see what all the filesare taking up (in terms of disk space)?**

``` bash
shad@linux:~/linux$ du -sh /home/shad/*
24K     /home/shad/folder1
4.0K    /home/shad/hello.txt
45M     /home/shad/linux
16K     /home/shad/root
```

(advanced)
7. **What is an init system (systemd)?**

- init is the first process that the linux kernel starts.
- responsible for starting all services/units you expect to have running.
- reparenting orphaned processes.

8. **What is a linux user made of?**

- an entry in a few files: /etc/passwd, /etc/shadow, /etc/group
- users made of little in a file system.
- linux: abstracts everything as files.

9. **/proc filesystem**

- kernel state reflected as a filesystem tree
- what can you do with /proc?
    - see information about processes
    - the arguments that something was run with
    - the environment that it was started with

(troubleshooting)

10. **I found a weird process running on my linux box, how would I investigate?**

- One of the `top`/`htop` to see resource usage.
- Look for the binary or config files.

``` bash
shad@linux:~/linux$ which apache2
/usr/sbin/apache2

shad@linux:~/linux$ ps aux | grep "apache2"
root        3125  0.0  0.0   6880  4920 ?        Ss   15:15   0:00 /usr/sbin/apache2 -k start
www-data    3128  0.0  0.0 1212972 5524 ?        Sl   15:15   0:00 /usr/sbin/apache2 -k start
www-data    3129  0.0  0.0 1213028 6400 ?        Sl   15:15   0:00 /usr/sbin/apache2 -k start
shad        6270  0.0  0.0   7076  2164 pts/0    S+   16:34   0:00 grep --color=auto apache2

shad@linux:~/linux$ cat /proc/3125/cmdline 
/usr/sbin/apache2-kstart

shad@linux:~/linux$ sudo lsof -p 3125
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
apache2 3125 root  cwd    DIR    8,1     4096      2 /
apache2 3125 root  rtd    DIR    8,1     4096      2 /
apache2 3125 root  txt    REG    8,1   754232  34621 /usr/sbin/apache2
apache2 3125 root  DEL    REG    0,1            1082 /dev/zero
apache2 3125 root  mem    REG    8,1    31088 265731 /usr/lib/apache2/modules/mod_status.so
apache2 3125 root  mem    REG    8,1    76144 265692 /usr/lib/apache2/modules/mod_mpm_event.so
apache2 3125 root  mem    REG    8,1  2125328   6534 /usr/lib/x86_64-linux-gnu/libc.so.6
apache2 3125 root  mem    REG    8,1    18800 265722 /usr/lib/apache2/modules/mod_setenvif.so
apache2 3125 root  mem    REG    8,1    39280 265695 /usr/lib/apache2/modules/mod_negotiation.so
apache2 3125 root  mem    REG    8,1    26992 265690 /usr/lib/apache2/modules/mod_mime.so
apache2 3125 root  mem    REG    8,1    22896 265671 /usr/lib/apache2/modules/mod_filter.so
apache2 3125 root  mem    REG    8,1    18800 265714 /usr/lib/apache2/modules/mod_reqtimeout.so
apache2 3125 root  mem    REG    8,1    14704 265667 /usr/lib/apache2/modules/mod_env.so
apache2 3125 root  mem    REG    8,1   113000   5153 /usr/lib/x86_64-linux-gnu/libz.so.1.3
apache2 3125 root  mem    REG    8,1    14704 265664 /usr/lib/apache2/modules/mod_dir.so
apache2 3125 root  mem    REG    8,1    39280 265662 /usr/lib/apache2/modules/mod_deflate.so
apache2 3125 root  mem    REG    8,1    43376 265644 /usr/lib/apache2/modules/mod_autoindex.so
apache2 3125 root  mem    REG    8,1    14704 265643 /usr/lib/apache2/modules/mod_authz_user.so
apache2 3125 root  mem    REG    8,1    14704 265641 /usr/lib/apache2/modules/mod_authz_host.so
apache2 3125 root  mem    REG    8,1    31088 265637 /usr/lib/apache2/modules/mod_authz_core.so
apache2 3125 root  mem    REG    8,1    14704 265633 /usr/lib/apache2/modules/mod_authn_file.so
apache2 3125 root  mem    REG    8,1    14704 265630 /usr/lib/apache2/modules/mod_authn_core.so
apache2 3125 root  mem    REG    8,1    18800 265626 /usr/lib/apache2/modules/mod_auth_basic.so
apache2 3125 root  mem    REG    8,1    22896 265623 /usr/lib/apache2/modules/mod_alias.so
apache2 3125 root  mem    REG    8,1    14704 265621 /usr/lib/apache2/modules/mod_access_compat.so
apache2 3125 root  mem    REG    8,1    35032   5441 /usr/lib/x86_64-linux-gnu/libuuid.so.1.3.0
apache2 3125 root  mem    REG    8,1   198664   5139 /usr/lib/x86_64-linux-gnu/libcrypt.so.1.1.0
apache2 3125 root  mem    REG    8,1   174336   6256 /usr/lib/x86_64-linux-gnu/libexpat.so.1.9.1
apache2 3125 root  mem    REG    8,1   245648  34477 /usr/lib/x86_64-linux-gnu/libapr-1.so.0.7.2
apache2 3125 root  mem    REG    8,1   184184  34581 /usr/lib/x86_64-linux-gnu/libaprutil-1.so.0.6.3
apache2 3125 root  mem    REG    8,1   625344   5502 /usr/lib/x86_64-linux-gnu/libpcre2-8.so.0.11.2
apache2 3125 root  mem    REG    8,1   236616   6531 /usr/lib/x86_64-linux-gnu/ld-linux-x86-64.so.2
apache2 3125 root    0r   CHR    1,3      0t0      5 /dev/null
apache2 3125 root    1w   CHR    1,3      0t0      5 /dev/null
apache2 3125 root    2w   REG    8,1      693 266433 /var/log/apache2/error.log
apache2 3125 root    3u  sock    0,9      0t0  22523 protocol: TCP
apache2 3125 root    4u  IPv6  22524      0t0    TCP *:http (LISTEN)
apache2 3125 root    5r  FIFO   0,15      0t0  23045 pipe
apache2 3125 root    6w  FIFO   0,15      0t0  23045 pipe
apache2 3125 root    7w   REG    8,1        0 266304 /var/log/apache2/other_vhosts_access.log
apache2 3125 root    8w   REG    8,1      145 266302 /var/log/apache2/access.log

shad@linux:~/linux$ sudo ss -tupnl | grep -E "Netid|3125"
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess                                                                               
tcp   LISTEN 0      511                *:80               *:*    users:(("apache2",pid=3129,fd=4),("apache2",pid=3128,fd=4),("apache2",pid=3125,fd=4))
```

11. **Aside from disk space, what are other things that could prevent you from creating a file?**

- incorrect ownership => leveraging `chown shad:shad lab_daemon.py`
- incorrect permissions => leveraging `chmod 777 lab_daemon.py`
- i-node exhaustion / what are i-nodes?
    - linux data structure that contains information about a file.
    - each file is represented by an i-node, which stores metadata such as file size, ownership, and permissions.
    - leveraging `df -i` to check i-node usage.

[17 Linux Interview Questions and Answers](https://www.youtube.com/watch?v=ppmUsyafyoo)

1. **How do I see my default route in Linux?**

``` bash
shad@linux:~/linux$ ip route
default via 10.0.0.1 dev eth0 proto dhcp src 10.0.0.4 metric 100 
10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.4 metric 100 
10.0.0.1 dev eth0 proto dhcp scope link src 10.0.0.4 metric 100 
168.63.129.16 via 10.0.0.1 dev eth0 proto dhcp src 10.0.0.4 metric 100 
169.254.169.254 via 10.0.0.1 dev eth0 proto dhcp src 10.0.0.4 metric 100 
```

2. **How do I bring up/down a network interface in Linux?**

``` bash
# shows all network interfaces
shad@linux:~/linux$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 60:45:bd:ef:f3:4d brd ff:ff:ff:ff:ff:ff
3: enP43014s1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 1500 qdisc mq master eth0 state UP mode DEFAULT group default qlen 1000
    link/ether 60:45:bd:ef:f3:4d brd ff:ff:ff:ff:ff:ff
    altname enP43014p0s2

shad@linux:~/linux$ sudo ip link set down enP43014s1
```

3. **What is the file /etc/fstab used for?**

- It's where you describe how your disks are mounted by default when the system boots.

``` bash
shad@linux:~/linux$ cat /etc/fstab
# CLOUD_IMG: This file was created/modified by the Cloud Image build process
UUID=fad0ebd7-fc6c-42fe-a34d-4e8d2f07946f       /        ext4   discard,commit=30,errors=remount-ro     0 1
LABEL=BOOT      /boot   ext4    defaults,discard        0 2
UUID=B0C0-7511  /boot/efi       vfat    umask=0077      0 1
/dev/disk/cloud/azure_resource-part1    /mnt    auto    defaults,nofail,x-systemd.after=cloud-init.service,_netdev,comment=cloudconfig  0       2

shad@linux:~/linux$ sudo blkid
/dev/sdb1: UUID="007be729-46f0-4cc9-833b-64fe19340e4d" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="76ec78d5-01"
/dev/sda16: LABEL="BOOT" UUID="4e0b0f7d-f018-493d-b384-d56a4d4661bd" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="bc888243-c7df-4555-b820-0fa1621cf3e0"
/dev/sda15: LABEL_FATBOOT="UEFI" LABEL="UEFI" UUID="B0C0-7511" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="d403963f-6b02-4be2-96ef-428181705f25"
/dev/sda1: LABEL="cloudimg-rootfs" UUID="fad0ebd7-fc6c-42fe-a34d-4e8d2f07946f" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="625cad27-7b63-4e5e-8bbd-6efc33b38415"
/dev/sda14: PARTUUID="a9d448c0-a37e-4b4c-8190-cf5a5200263f"
```

4. **How do I check for free disk space?**

``` bash
shad@linux:~/linux$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        29G  2.9G   26G  11% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           1.6G 1012K  1.6G   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
efivarfs        128K   37K   87K  30% /sys/firmware/efi/efivars
/dev/sda16      881M  110M  710M  14% /boot
/dev/sda15      105M  6.2M   99M   6% /boot/efi
/dev/sdb1        16G   28K   15G   1% /mnt
tmpfs           790M   12K  790M   1% /run/user/1000

# list block devices (extra credit)
# helps to see disk partitions and mount points
shad@linux:~/linux$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda       8:0    0   30G  0 disk 
├─sda1    8:1    0   29G  0 part /
├─sda14   8:14   0    4M  0 part 
├─sda15   8:15   0  106M  0 part /boot/efi
└─sda16 259:0    0  913M  0 part /boot
sdb       8:16   0   16G  0 disk 
└─sdb1    8:17   0   16G  0 part /mnt
```

5. **How do I check for open ports on my Linux server?**

``` bash
# n = numeric, p = process, a = all, t = tcp, u = udp
shad@linux:~/linux$ sudo ss -tupna
Netid  State   Recv-Q  Send-Q   Local Address:Port     Peer Address:Port   Process                                                                                 
udp    UNCONN  0       0           127.0.0.54:53            0.0.0.0:*       users:(("systemd-resolve",pid=532,fd=16))                                              
udp    UNCONN  0       0        127.0.0.53%lo:53            0.0.0.0:*       users:(("systemd-resolve",pid=532,fd=14))                                              
udp    UNCONN  0       0        10.0.0.4%eth0:68            0.0.0.0:*       users:(("systemd-network",pid=694,fd=21))                                              
udp    UNCONN  0       0            127.0.0.1:323           0.0.0.0:*       users:(("chronyd",pid=921,fd=6))                                                       
udp    UNCONN  0       0                [::1]:323              [::]:*       users:(("chronyd",pid=921,fd=7))                                                       
tcp    LISTEN  0       4096        127.0.0.54:53            0.0.0.0:*       users:(("systemd-resolve",pid=532,fd=17))                                              
tcp    LISTEN  0       4096     127.0.0.53%lo:53            0.0.0.0:*       users:(("systemd-resolve",pid=532,fd=15))                                              
tcp    LISTEN  0       1024         127.0.0.1:33999         0.0.0.0:*       users:(("code-7d842fb85a",pid=1276,fd=9))                                              
tcp    LISTEN  0       4096           0.0.0.0:22            0.0.0.0:*       users:(("sshd",pid=1120,fd=3),("systemd",pid=1,fd=55))                                 
tcp    ESTAB   0       0            127.0.0.1:55432       127.0.0.1:33999   users:(("sshd",pid=1255,fd=8))                                                         
tcp    ESTAB   0       0            127.0.0.1:33999       127.0.0.1:55432   users:(("code-7d842fb85a",pid=1276,fd=12))                                             
tcp    ESTAB   0       0             10.0.0.4:54142   20.85.130.105:443     users:(("node",pid=1420,fd=40))                                                        
tcp    ESTAB   0       0             10.0.0.4:54154   20.85.130.105:443     users:(("node",pid=1420,fd=63))                                                        
tcp    ESTAB   0       316           10.0.0.4:22       108.28.[REDACTED]:51424   users:(("sshd",pid=1255,fd=4),("sshd",pid=1121,fd=4))                                  
tcp    ESTAB   0       0             10.0.0.4:51976   140.82.112.22:443     users:(("node",pid=1420,fd=38))                                                        
tcp    LISTEN  0       511                  *:80                  *:*       users:(("apache2",pid=3129,fd=4),("apache2",pid=3128,fd=4),("apache2",pid=3125,fd=4))  
tcp    LISTEN  0       4096              [::]:22               [::]:*       users:(("sshd",pid=1120,fd=4),("systemd",pid=1,fd=56))                                 
```

6. **How do I found out which processes are consuming most of my CPU?**
``` bash
shad@linux:~/linux$ ps aux --sort -pcpu | head
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
shad        1420  5.8 13.0 55923340 1056700 ?    Sl   14:38   8:41 /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/node --dns-result-order=ipv4first /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/out/bootstrap-fork --type=extensionHost --transformURIs --useHostProxy=false
shad        1276  0.1  0.2  35752 20948 ?        Sl   14:38   0:17 /home/shad/.vscode-server/code-7d842fb85a0275a4a8e4d7e040d2625abbf7f084 command-shell --cli-data-dir /home/shad/.vscode-server/cli --parent-process-id 1258 --on-host=127.0.0.1 --on-port
shad        1363  0.1  1.5 11843092 123772 ?     Sl   14:38   0:14 /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/node /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/out/server-main.js --connection-token=remotessh --accept-server-license-terms --start-server --enable-remote-auto-shutdown --socket-path=/tmp/code-0e8d7d6d-2625-4188-a665-0b11b3c50357
shad        1444  0.1  1.0 1163856 87484 ?       Sl   14:38   0:13 /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/node /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/out/bootstrap-fork --type=ptyHost --logsPath /home/shad/.vscode-server/data/logs/20251106T143841
shad        1255  0.1  0.0  15316  7596 ?        S    14:38   0:12 sshd: shad@notty
shad        1494  0.1  1.0 1030840 85540 ?       Sl   14:38   0:11 /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/node /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/extensions/markdown-language-features/dist/serverWorkerMain --node-ipc --clientProcessId=1420
shad        1431  0.0  0.8 1263312 68748 ?       Sl   14:38   0:08 /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/node /home/shad/.vscode-server/cli/servers/Stable-7d842fb85a0275a4a8e4d7e040d2625abbf7f084/server/out/bootstrap-fork --type=fileWatcher
root        1023  0.0  0.4 412584 35832 ?        Sl   14:37   0:07 /usr/bin/python3 -u bin/WALinuxAgent-2.15.0.1-py3.12.egg -run-exthandlers
_chrony      921  0.0  0.0  11204  3668 ?        S    14:37   0:03 /usr/sbin/chronyd -F 1

# also top/htop
```

7. **Name two popular open source web servers?**
- ngnix
- apache

8. **How do I check how many packages are installed on my Linux server?**

``` bash
shad@linux:~/linux$ apt list --installed | wc -l

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

677
```

9. **Ports**
- ssh: 22
- dns: 53
- https: 443
- smtp: 25

``` bash
shad@linux:~/linux$ cat /etc/services | grep "22"
ssh             22/tcp                          # SSH Remote Login Protocol
```

9. **How do I keep a user account on the server, but not allow them to login?**

- Looking for user mod and setting their login shell.
- `shad@linux:~/linux$ usermod -s /usr/sbin/nologin <user>`

10. **What hypervisor is built as a module into the linux kernel?**
- KVM (Kernel-based Virtual Machine)

11. **Name some popular OSS config management utilities?**
- Ansible
- Puppet
- SaltStack

11. **Using systemd, how do I tell a service to start on boot?**
- `sudo systemctl enable apache2`
- `enable --now` to start immediately and enable on boot.

12. **What is your favorite piece of OSS?**

13. **What Linux Distro are you running and why?**

``` bash
shad@linux:~/linux$ uname -a
Linux linux 6.14.0-1014-azure #14~24.04.1-Ubuntu SMP Fri Oct  3 20:52:11 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

14. **You log into a linux server, what is the first command you run?**

``` bash
shad@linux:~/linux$ w
 17:21:43 up  2:44,  1 user,  load average: 0.24, 0.26, 0.23
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU  WHAT
shad              108.28.[REDACTED]     14:38    2:37m  0.00s  0.01s sshd: shad [priv]

# This merges into checking the OS version
shad@linux:~/linux$ cat /etc/*release*
```

- This shows you who is logged in, uptime of the server, and load averages.
- combo of `who` and `uptime`

15. *What software do you have to install on a server?**

- htop
