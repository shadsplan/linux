373

# Chapter 12. System and Process

## 12.1 The /proc File System
- Want to understand the attributes of the kernel to answer the following:
    - How many processes are running on the system and who owns them?
    - What files does a process have open?
    - What files are currently locked, and which processes hold the locks?
    - What sockets are being used on the system?
- In order to provide easier access to kernel information, many modern UNIX implementations provide a /proc virtual file system. This file system resides under the /proc directory and contains various files that expose kernel information, allowing processes to conveniently read that information, and change it in some cases, using normal file I/O system calls.
- **The /proc file system is said to be virtual because the files and subdirectories that it contains don't reside on a disk. Instead, the kernel creates them “on the fly” as processes access them.**

``` bash
shad@linux:~/linux/notes/lpi$ ps aux | grep "apache2"
root       10606  0.0  0.0   6880  4880 ?        Ss   02:43   0:00 /usr/sbin/apache2 -k start

shad@linux:~/linux/notes/lpi$ sudo ls -l /proc/10606/fd/
total 0
lr-x------ 1 root root 64 Nov  1 03:22 0 -> /dev/null
l-wx------ 1 root root 64 Nov  1 03:22 1 -> /dev/null
l-wx------ 1 root root 64 Nov  1 03:22 2 -> /var/log/apache2/error.log
lrwx------ 1 root root 64 Nov  1 03:22 3 -> 'socket:[134798]'
lrwx------ 1 root root 64 Nov  1 03:22 4 -> 'socket:[134799]'
lr-x------ 1 root root 64 Nov  1 03:22 5 -> 'pipe:[134822]'
l-wx------ 1 root root 64 Nov  1 03:22 6 -> 'pipe:[134822]'
l-wx------ 1 root root 64 Nov  1 03:22 7 -> /var/log/apache2/other_vhosts_access.log
l-wx------ 1 root root 64 Nov  1 03:22 8 -> /var/log/apache2/access.log

shad@linux:~/linux/notes/lpi$ cat /proc/10606/status
Name:   apache2
Umask:  0022
State:  S (sleeping)
Tgid:   10606
Ngid:   0
Pid:    10606
PPid:   1
TracerPid:      0
Uid:    0       0       0       0
Gid:    0       0       0       0
FDSize: 64
Groups:  
NStgid: 10606
NSpid:  10606
NSpgid: 10606
NSsid:  10606
Kthread:        0
VmPeak:     6880 kB
VmSize:     6880 kB
VmLck:         0 kB
VmPin:         0 kB
VmHWM:      4880 kB
VmRSS:      4880 kB
RssAnon:            1300 kB
RssFile:            3324 kB
RssShmem:            256 kB
VmData:     1340 kB
VmStk:       132 kB
VmExe:       360 kB
VmLib:      2940 kB
VmPTE:        52 kB
VmSwap:        0 kB
HugetlbPages:          0 kB
CoreDumping:    0
THP_enabled:    1
untag_mask:     0xffffffffffffffff
Threads:        1
SigQ:   1/31481
SigPnd: 0000000000000000
ShdPnd: 0000000000000000
SigBlk: 0000000000000000
SigIgn: 0000000001001000
SigCgt: 00000000080046eb
CapInh: 0000000000000000
CapPrm: 000001ffffffffff
CapEff: 000001ffffffffff
CapBnd: 000001ffffffffff
CapAmb: 0000000000000000
NoNewPrivs:     0
Seccomp:        0
Seccomp_filters:        0
Speculation_Store_Bypass:       vulnerable
SpeculationIndirectBranch:      always enabled
Cpus_allowed:   3
Cpus_allowed_list:      0-1
Mems_allowed:   00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
Mems_allowed_list:      0
voluntary_ctxt_switches:        2926
nonvoluntary_ctxt_switches:     4
x86_Thread_features:
x86_Thread_features_locked:
```

## 12.3 Summary
- The /proc file system exposes a range of kernel information to application programs. Each /proc/PID subdirectory contains files and subdirectories that provide information about the process whose ID matches PID. Various other files and directories under /proc expose system-wide information that programs can read and, in some cases, modify.