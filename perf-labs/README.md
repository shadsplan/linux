# perf-labs

```bash
sudo apt install make
sudo apt install gcc
```

## [Linux Performance Tools, Brendan Gregg, part 1 of 2](https://www.youtube.com/watch?v=FJW8nGV4jxY)

Useful Links:
- [Brendan Gregg's Linux Performance](https://www.brendangregg.com/linuxperf.html): Has a great image.
- [Brendan Gregg's USE Method](https://www.brendangregg.com/usemethod.html)
- [Brendan Gregg's USE Method - Linux](https://www.brendangregg.com/USEmethod/use-linux.html)

Goals: For any application running on a server, we'll learn how to:
- Begin analyzing its performance
- Step through a sequence of meaningful analysis
- Getting to an ending point.

> Where do I start? There's so many tools, there's so many metrics!

`shad@linux:~/linux/perf-labs/src$ ./lab002`

## My system is slow... DEMO & DISCUSSION

``` bash
shad@linux:~/linux/perf-labs/src$ top
top - 21:02:00 up  3:20,  2 users,  load average: 0.00, 0.05, 0.01
Tasks: 150 total,   1 running, 149 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.0 us,  0.8 sy,  0.0 ni, 98.0 id,  0.0 wa,  0.0 hi,  0.2 si,  0.0 st 
MiB Mem :   7893.2 total,   5429.4 free,   1609.1 used,   1120.5 buff/cache     
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   6284.1 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                   
   1416 shad      20   0   11.3g 125228  51752 S   0.7   1.5   0:15.94 node                      
   1474 shad      20   0   53.2g 902600  60200 S   0.7  11.2   2:00.01 node                      
   1498 shad      20   0 1161552  83400  48060 S   0.3   1.0   0:16.91 node                      
   4556 shad      20   0   15280   7372   5096 S   0.3   0.1   0:01.28 sshd                      
```

- I don't see it in `top`, so it's probably not eating CPU.

``` bash
shad@linux:~/linux/perf-labs/src$ iostat 1
Linux 6.14.0-1014-azure (linux)         11/07/25        _x86_64_        (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.89    0.02    0.30    0.12    0.00   98.67

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
loop0             0.00         0.00         0.00         0.00         14          0          0
sda               3.48        62.05        72.72       151.63     766306     898086    1872552
sdb               0.05         0.79        26.89      1358.33       9773     332132   16775172


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.50    0.00    0.50    0.00    0.00   99.00

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
loop0             0.00         0.00         0.00         0.00          0          0          0
sda               0.00         0.00         0.00         0.00          0          0          0
sdb               0.00         0.00         0.00         0.00          0          0          0
```

- Shows that our disks are idle too.

``` bash
shad@linux:~/linux/perf-labs/src$ sudo strace -p `pgrep lab002`
strace: Process 6986 attached
accept(3, 
```

- strace shows that it is waiting on an accept() syscall.
- `-p` means attach to a process.
- `pgrep` gets the process ID of our program.

> One of the things about performance methodologies is that you can learn how to exonerate a system, and do a methodology in 5 minutes. "I've checked system resources, they're good, have you checked your own code?"

Otherwise, we could be chasing tools for hours.

## Methodologies

### Anti-Methodologies

- Running tools that are familiar to us, but not appropriate for the problem at hand.
- Changing things at random until the problem goes away.
- Find a system or environment component you are not responsible for, and blame them.
    - Network team: Dropped packets.

### Actual Methodologies

#### Problem Statement Method
1. What makes you **think** there is a performance problem?
2. Has this system **ever** performed well?
3. What has **changed** recently? (Software? Hardware? Load?)
4. Can the performance degradation be expressed in terms of **latency** or run time?
5. Does the problem affect **other** people or applications (or is it just you)?
6. What is the **environment**? Software, hardware, instance types? Versions? Configuration?

#### Workload Characterization Method
1. **Who** is causing the load? PID, UID, IP addr,...
2. **Why** is the load called? code path, stack trace
3. **What** is the load? IOPS, tput, type, r/w
4. **How** is the load changing over time?

#### USE Method
Take a functional diagram of the system, and for each resource, check:
1. Utilization: Busy Time
2. Saturation: Queue Length or Queued Time
3. Errors: Easy to interpret (objective)

https://www.brendangregg.com/USEmethod/use-linux.html
 
- Off-CPU Analysis
- CPU Profile Method
- RTFM Method
- Active Benchmarking (covered later)
- Static Performance Tuning (covered later)

## Tools

Tool Types:
- Observability: Watch activity. Safe, usually , depending on resource overhead.
- Benchmarking: Load test. Caution: Production tests can cause issues due to contention.
- Tuning: Change. Danger: Changes could hurt performance, now or later with load.
- Static: Check configuration. Should be safe.

## Observability Tools: Basic

### uptime

``` bash
shad@linux:~/linux/perf-labs/src$ uptime
22:01:37 up  4:19,  2 users,  load average: 0.31, 0.10, 0.05

shad@linux:~/linux/perf-labs/src$ nproc
2
 ```

- One way to print load averages.
- A measure of resource demand: CPU + disks.
- Time constants of 1, 5, and 15 minutes.
- Load > # of CPUs `nproc`, may mean CPU saturation.
    - Don't spend more that 5 seconds studying these.
- Shad: I also like `w` because it shows who is logged in.

### top/htop
``` bash
shad@linux:~/linux/perf-labs/src$ top
top - 22:14:22 up  4:32,  2 users,  load average: 0.05, 0.07, 0.08
Tasks: 149 total,   1 running, 148 sleeping,   0 stopped,   0 zombie
%Cpu(s):  4.8 us,  0.0 sy,  0.0 ni, 95.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st 
MiB Mem :   7893.2 total,   5131.2 free,   1891.6 used,   1136.4 buff/cache     
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   6001.6 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                   
   1474 shad      20   0   53.5g   1.1g  60104 S   9.1  14.7   7:29.62 node                      
      1 root      20   0   22632  13820   9596 S   0.0   0.2   0:03.54 systemd                   
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthreadd                  
      3 root      20   0       0      0      0 S   0.0   0.0   0:00.00 pool_workqueue_release    
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 kworker/R-rcu_gp          
```

- System and per-processes interval summary.
- %CPU is summed across all CPUs.
- Can miss short-lived processes (`atop` won't)
- Can consume noticable CPU to read `/proc`.

### ps
``` bash
shad@linux:~/linux/perf-labs/src$ ps -ef f
UID          PID    PPID  C STIME TTY      STAT   TIME CMD
root        2463       1  0 18:18 ?        Ss     0:00 /usr/sbin/apache2 -k start
www-data    2465    2463  0 18:18 ?        Sl     0:00  \_ /usr/sbin/apache2 -k start
www-data    2467    2463  0 18:18 ?        Sl     0:00  \_ /usr/sbin/apache2 -k start
```

- Process status listing.

### vmstat

``` bash
shad@linux:~/linux/perf-labs/src$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 0  0      0 5319312  45448 1118612    0    0    47    77  238    1  2  0 98  0  0  0
 1  0      0 5319584  45448 1118652    0    0     0     0  329  336  1  1 98  0  0  0
 0  0      0 5318788  45448 1118652    0    0     0     4  236  251  1  1 98  0  0  0
```

- Virtual memory statistics and more.
- High level CPU summary.
    - "r" is runnable tasks.
- Interesting columns:
    - r = run queue length. Indicates CPUs have tasks running and tasks that are runnable but waiting.
    - memory = free, buff, cache.
    - cpu = us (user time = what my applications are doing in ex. jvm runtime), sy (system time = what the kernel is doing).

### iostat

``` bash
shad@linux:~/linux/perf-labs/src$ iostat -xmdz 1
Linux 6.14.0-1014-azure (linux)         11/07/25        _x86_64_        (2 CPU)

Device            r/s     rMB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wMB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dMB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
loop0            0.00      0.00     0.00   0.00    0.09     1.27    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
sda              1.05      0.04     0.21  16.71    1.55    43.36    1.82      0.06     1.26  40.97    1.83    31.02    0.09      0.11     0.00   5.14    3.44  1242.04    0.30    1.34    0.01   0.22
sdb              0.02      0.00     0.00   0.32    0.35    31.83    0.02      0.02     0.06  74.63    2.35   922.59    0.00      0.97     0.00  47.06    0.22 1863908.00    0.00    0.00    0.00   0.00


Device            r/s     rMB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wMB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dMB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util


Device            r/s     rMB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wMB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dMB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
sda              0.00      0.00     0.00   0.00    0.00     0.00    1.00      0.00     0.00   0.00    0.00     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00


shad@linux:~/linux/perf-labs/src$ iostat 1
Linux 6.14.0-1014-azure (linux)         11/07/25        _x86_64_        (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.94    0.02    0.34    0.09    0.00   97.60

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
loop0             0.00         0.00         0.00         0.00         14          0          0
sda               2.95        45.41        56.13       110.97     769554     951122    1880484
sdb               0.04         0.58        19.60       989.95       9773     332132   16775172


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.51    0.00    0.00    0.00    0.00   99.49

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
loop0             0.00         0.00         0.00         0.00          0          0          0
sda               0.00         0.00         0.00         0.00          0          0          0
sdb               0.00         0.00         0.00         0.00          0          0          0


```

- Block I/O (disk) stats. 1st output is since boot.
- High IOPS = a lot of load.
- Useful Stats:
    - r/s, w/s, rMB/s, wBM/s
    - avgqu-sz
    - await: average wait time (ms) - you're waiting for the disk i/o.
    - svctm: service time
    - %util: during an interval, how busy the device was. 100% means saturated.

### mpstat

``` bash
shad@linux:~/linux/perf-labs/src$ mpstat -P ALL 1
Linux 6.14.0-1014-azure (linux)         11/07/25        _x86_64_        (2 CPU)

22:29:54     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
22:29:55     all    0.50    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   99.50
22:29:55       0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
22:29:55       1    1.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00   99.00

22:29:55     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
22:29:56     all    0.50    0.00    1.49    0.00    0.00    0.00    0.00    0.00    0.00   98.01
22:29:56       0    0.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00   99.00
22:29:56       1    0.99    0.00    1.98    0.00    0.00    0.00    0.00    0.00    0.00   97.03
```

- idle column is important.
- Look for unbalanced workloads, hot CPUs.

### free

``` bash
shad@linux:~/linux/perf-labs/src$ free -m 
               total        used        free      shared  buff/cache   available
Mem:            7893        1950        5068           4        1140        5942
Swap:              0           0           0
```

- Main memory usage.
- buff/cache: block device I/O cache + virtual page cache

## Latency is now much higher... DEMO & DISCUSSION

``` bash

shad@linux:~/linux/perf-labs/src$ ./lab005

# Purpose: Per-process CPU and memory usage
# Layer: Process scheduler / runtime level.
shad@linux:~/linux/perf-labs/src$ top
top - 22:49:14 up  5:07,  2 users,  load average: 1.60, 0.55, 0.26
Tasks: 151 total,   1 running, 150 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.9 us,  2.6 sy,  0.0 ni, 48.3 id, 46.9 wa,  0.0 hi,  0.3 si,  0.0 st 
MiB Mem :   7893.2 total,   5111.8 free,   1846.1 used,   1202.1 buff/cache     
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   6047.1 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                           
   1485 shad      20   0 1263540  68632  48016 S   3.0   0.8   0:19.92 node                                              
     98 root      20   0       0      0      0 D   1.7   0.0   0:01.93 jbd2/sda1-8                                       
  10364 shad      20   0    2548   1056   1056 D   1.7   0.0   0:01.56 lab005

# Purpose: Virtual memory and process scheduling stats
# Layer: Kernel VM + scheduler
shad@linux:~/linux/perf-labs/src$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 2  1      0 5138512  95300 1136284    0    0    41   125  280    1  2  0 97  1  0  0
 1  1      0 5138260  95300 1136288    0    0     0  4704 2667 4743  1  4 48 47  0  0
 0  1      0 5139072  95300 1136288    0    0     0  4604 2801 4907 14  5 41 41  0  0
 0  1      0 5139600  95300 1136288    0    0     0  4524 2741 4738 11  4 43 42  0  0
 0  1      0 5139600  95300 1136288    0    0     0  4584 2400 4410  1  3 49 47  0  0
 0  1      0 5139600  95300 1136288    0    0     0  4464 2486 4436  3  3 48 47  0  0                                          
```

- We don't see high CPU utilization, no evidence of CPU saturation.
- Plenty of idle CPU.
- Also plenty of free memory.

``` bash
# Purpose: Per-device disk utilization
# Block I/O subsystem
shad@linux:~/linux/perf-labs/src$ iostat -x 1
Linux 6.14.0-1014-azure (linux)         11/07/25        _x86_64_        (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.14    0.02    0.40    1.05    0.00   96.40

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
loop0            0.00      0.00     0.00   0.00    0.09     1.27    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
sda              0.95     41.14     0.19  16.63    1.56    43.24   16.42    150.42     6.33  27.83    1.43     9.16    0.09    101.15     0.00   4.75    3.43  1114.10    5.14    2.16    0.04   2.23
sdb              0.02      0.52     0.00   0.32    0.35    31.83    0.02     17.71     0.06  74.63    2.35   922.59    0.00    894.36     0.00  47.06    0.22 1863908.00    0.00    0.00    0.00   0.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.06    0.00    2.06   47.42    0.00   48.45

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
loop0            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
sda              0.00      0.00     0.00   0.00    0.00     0.00  704.00   4696.00   235.00  25.03    1.37     6.67    0.00      0.00     0.00   0.00    0.00     0.00  235.00    2.12    1.47  96.40
sdb              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.53    0.00    3.57   46.94    0.00   47.96

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
loop0            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
sda              0.00      0.00     0.00   0.00    0.00     0.00  659.00   4400.00   221.00  25.11    1.46     6.68    0.00      0.00     0.00   0.00    0.00     0.00  219.00    2.23    1.45  95.00
sdb              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
```

- utilization is high, at 95%
- 659 write IOPS, 4 MB/s
- When you max out CPUs, Unix-like systems can be graceful about it, because Kernel understands priorities and handles multithreading well.
- Not so true with disks. When you max out disks, it's hard to send I/O with higher priority if they're busy.
- With disks, you see perf issues beyond 60-80% utilization.

> "Disks look busy, you should check out why is your application doing more writes to the disk?"

Let's complete the USE method, and check network I/O.

``` bash
# Purpose: Per-interface network throughput
# Layer: Network device (NIC) layer.
shad@linux:~/linux/perf-labs/src$ sar -n DEV 1
Linux 6.14.0-1014-azure (linux)         11/07/25        _x86_64_        (2 CPU)

23:09:12        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
23:09:13           lo     26.00     26.00      2.64      2.64      0.00      0.00      0.00      0.00
23:09:13         eth0     10.00     10.00      1.09      1.75      0.00      0.00      0.00      0.00
23:09:13    enP44461s1     10.00     10.00      1.22      1.75      0.00      0.00      0.00      0.00

23:09:13        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
23:09:14           lo     18.00     18.00      1.90      1.90      0.00      0.00      0.00      0.00
23:09:14         eth0     23.00     27.00      4.56      6.28      0.00      0.00      0.00      0.00
23:09:14    enP44461s1      8.00     28.00      1.02      6.36      0.00      0.00      0.00      0.00

23:09:14        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
23:09:15           lo     19.00     19.00      1.78      1.78      0.00      0.00      0.00      0.00
23:09:15         eth0      8.00      7.00      0.80      1.12      0.00      0.00      0.00      0.00
23:09:15    enP44461s1      8.00      7.00      0.91      1.12      0.00      0.00      0.00      0.00
```

- It looks idle.

Takeaways:
- Disks look busy.
- CPU looks moderately busy, but could be related to above.

Potential Reason: "Changed config in database, so it's flushing the writes all the time. Let's revert."

## Observability Tools: Intermediate

### strace

``` bash
shad@linux:~/linux/perf-labs/src$ sudo strace -tttT -p 10364
strace: Process 10364 attached
1762557934.822483 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.003915>
1762557934.826665 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.004045>
1762557934.830864 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.005562>
1762557934.836637 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.004218>
1762557934.841067 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.004261>
1762557934.845494 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.005494>
1762557934.851323 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.003985>
1762557934.855451 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.005496>
1762557934.861163 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.004511>
1762557934.865853 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.004689>
1762557934.870677 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.004299>
1762557934.875151 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.003961>
1762557934.879207 write(3, "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0"..., 8192) = 8192 <0.003966>

# Wow... that's a lot of writes!
# 10364 is our lab005 process from above.
```

- Translates syscall args: Very helpful for solving system usage issues.
- Currently has massive overhead (ptrace based): Can slow the target by > 100x. Use extreme caution in production.
- `-ttt`: time (us) since epoch, `-T`: time spent in syscall (s)

### tcpdump

``` bash
shad@linux:~/linux/perf-labs/src$ sudo tcpdump -i eth0
libibverbs: Warning: couldn't open config directory '/etc/libibverbs.d'.
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
23:27:59.699631 IP pool-108-28-[REDACTED].washdc.fios.verizon.net.57846 > linux.internal.cloudapp.net.ssh: Flags [.], ack 1410067280, win 250, length 0
23:27:59.699631 IP pool-108-28-[REDACTED].washdc.fios.verizon.net.57846 > linux.internal.cloudapp.net.ssh: Flags [P.], seq 0:124, ack 1, win 250, length 124
23:27:59.699696 IP linux.internal.cloudapp.net.ssh > pool-108-28-[REDACTED].washdc.fios.verizon.net.57846: Flags [.], ack 4294967188, win 9096, options [nop,nop,sack 1 {0:124}], length 0
23:27:59.701436 IP linux.internal.cloudapp.net.ssh > pool-108-28-[REDACTED].washdc.fios.verizon.net.57846: Flags [P.], seq 1:237, ack 4294967188, win 9096, options [nop,nop,sack 1 {0:124}], length 236
23:27:59.712964 IP pool-108-28-[REDACTED].washdc.fios.verizon.net.57846 > linux.internal.cloudapp.net.ssh: Flags [P.], seq 4294967188:124, ack 1, win 250, length 232
```

- Instead of system call traces, we're doing packet traces.
- Sniff network packets for post analysis.
- Study packet sequences with timestamps
- CPU overhead optimized (socket ring buffers), but can still be significant. Use caution.

### pidstat

``` bash
shad@linux:~/linux/perf-labs/src$ pidstat -t 1
Linux 6.14.0-1014-azure (linux)         11/07/25        _x86_64_        (2 CPU)

23:55:50      UID      TGID       TID    %usr %system  %guest   %wait    %CPU   CPU  Command
23:55:51        0        56         -    0.00    0.96    0.00    0.00    0.96     0  kworker/0:1H-kblockd
23:55:51        0        73         -    0.00    0.96    0.00    0.00    0.96     1  kworker/1:1H-kblockd
23:55:51        0         -        73    0.00    0.96    0.00    0.00    0.96     1  |__kworker/1:1H-kblockd
23:55:51        0        98         -    0.00    1.92    0.00    0.00    1.92     1  jbd2/sda1-8
23:55:51        0         -        98    0.00    1.92    0.00    0.00    1.92     1  |__jbd2/sda1-8
23:55:51        0       201         -    0.00    0.96    0.00    0.00    0.96     1  multipathd
23:55:51     1000      1416      1416    0.96    0.00    0.00    0.00    0.96     0  (node)__node
23:55:51     1000      1474         -   19.23    0.96    0.00    0.00   20.19     1  node
23:55:51     1000         -      1474    8.65    0.00    0.00    0.00    8.65     1  |__node
23:55:51     1000         -      1481    1.92    0.00    0.00    0.00    1.92     0  |__libuv-worker
23:55:51     1000         -      1482    2.88    0.00    0.00    0.00    2.88     0  |__libuv-worker
23:55:51     1000         -      1483    2.88    0.96    0.00    0.00    3.85     1  |__libuv-worker
23:55:51     1000         -      1484    1.92    0.00    0.00    0.00    1.92     0  |__libuv-worker
23:55:51     1000      1485         -    1.92    0.96    0.00    0.00    2.88     1  node
23:55:51     1000         -      1485    0.96    0.00    0.00    0.00    0.96     1  |__node
23:55:51     1000         -      1497    0.96    0.96    0.00    0.96    1.92     1  |__node
23:55:51     1000      1498      1498    0.00    0.96    0.00    0.00    0.96     0  (node)__node
23:55:51     1000     10364         -    0.00    0.96    0.00    0.00    0.96     1  lab005
23:55:51     1000         -     10364    0.00    0.96    0.00    0.00    0.96     1  |__lab005
23:55:51     1000     12875         -    0.96    0.96    0.00    0.00    1.92     0  pidstat
23:55:51     1000         -     12875    0.96    0.96    0.00    0.00    1.92     0  |__pidstat

shad@linux:~/linux/perf-labs/src$ pidstat -d 1
Linux 6.14.0-1014-azure (linux)         11/07/25        _x86_64_        (2 CPU)

23:57:06      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
23:57:07     1000     10364      0.00   1893.07      0.00       0  lab005

23:57:07      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
23:57:08     1000     10364      0.00   1848.00      0.00       0  lab005
```

- Very useful process stats. eg. by-thread, disk I/O.
- `-t`: per-thread stats. usr/system time important, for each process. Helpful for analysis:
    - mostly usr time: application code.
    - mostly sys time: system calls, device I/O.
- `-d`: per-process block I/O stats.

### swapon
- Shows swap device usage, if you're exhausted memory.
- However, most systems these days don't have swap enabled...

### lsof
- More of a debug tool. Investigate file descriptor usage.
- Helps you understand an environment, by seeing who's connected.

### sar
- System Activity Reporter.

``` bash
had@linux:~/linux/perf-labs/src$ sar -n TCP,ETCP,DEV 1
Linux 6.14.0-1014-azure (linux)         11/08/25        _x86_64_        (2 CPU)

00:01:51        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
00:01:52           lo     29.00     29.00      2.78      2.78      0.00      0.00      0.00      0.00
00:01:52         eth0     22.00     17.00      2.31      2.32      0.00      0.00      0.00      0.00
00:01:52    enP44461s1     22.00     17.00      2.61      2.32      0.00      0.00      0.00      0.00

00:01:51     active/s passive/s    iseg/s    oseg/s
00:01:52         0.00      0.00     51.00     46.00

00:01:51     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
00:01:52         0.00      0.00      0.00      0.00      1.00
```

- Active/s - Inbound TCP connection attempts
- Passive/s - Outbound TCP connection attempts

## App is taking forever... DEMO & DISCUSSION

``` bash
shad@linux:~/linux/perf-labs/src$ ./lab003

# A lot of user time and system time
shad@linux:~/linux/perf-labs/src$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 1  0      0 5165680  95432 1172004    0    0    33   931  696    5  3  1 87  9  0  0
 2  0      0 5165448  95432 1172004    0    0     0     0 1179  262 32 20 49  0  0  0
 1  0      0 5164608  95432 1172004    0    0     0     0 1170  222 31 22 48  0  0  0
 1  0      0 5169808  95432 1172004    0    0     0 10244 1303  349 32 22 46  0  0  0

# same thing observed in mpstat
shad@linux:~/linux/perf-labs/src$ mpstat 1
Linux 6.14.0-1014-azure (linux)         11/08/25        _x86_64_        (2 CPU)

00:19:09     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
00:19:10     all   31.50    0.00   19.50    0.00    0.00    0.00    0.00    0.00    0.00   49.00
00:19:11     all   32.16    0.00   21.11    0.00    0.00    0.00    0.00    0.00    0.00   46.73
00:19:12     all   30.85    0.00   20.90    0.00    0.00    0.00    0.00    0.00    0.00   48.26
00:19:13     all   29.50    0.00   21.00    0.00    0.00    0.50    0.00    0.00    0.00   49.00
00:19:14     all   29.00    0.00   21.50    0.00    0.00    0.00    0.00    0.00    0.00   49.50
00:19:15     all   31.16    0.00   21.61    0.00    0.00    0.00    0.00    0.00    0.00   47.24
^CAverage:     all   30.69    0.00   20.93    0.00    0.00    0.08    0.00    0.00    0.00   48.29

# Oh no! One process is taking up all the CPU!
shad@linux:~/linux/perf-labs/src$ pidstat 1
Linux 6.14.0-1014-azure (linux)         11/08/25        _x86_64_        (2 CPU)

00:24:54      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
00:24:55        0      1041    0.98    0.00    0.00    0.00    0.98     0  python3
00:24:55     1000      1498    0.98    0.98    0.00    0.00    1.96     0  node
00:24:55     1000     13741   60.78   40.20    0.00    0.00  100.98     1  lab003
00:24:55     1000     14596    0.98    0.98    0.00    0.98    1.96     0  pidstat

00:24:55      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
00:24:56     1000      1416    1.00    1.00    0.00    1.00    2.00     1  node
00:24:56     1000      1474    1.00    0.00    0.00    0.00    1.00     0  node
00:24:56     1000     13741   61.00   37.00    0.00    0.00   98.00     0  lab003
00:24:56     1000     14596    0.00    1.00    0.00    0.00    1.00     1  pidstat

00:24:56      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
00:24:57     1000     13741   61.00   39.00    0.00    0.00  100.00     1  lab003
00:24:57     1000     14596    1.00    0.00    0.00    0.00    1.00     0  pidstat
^C

Average:      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
Average:        0      1041    0.33    0.00    0.00    0.00    0.33     -  python3
Average:     1000      1416    0.33    0.33    0.00    0.33    0.66     -  node
Average:     1000      1474    0.33    0.00    0.00    0.00    0.33     -  node
Average:     1000      1498    0.33    0.33    0.00    0.33    0.66     -  node
Average:     1000     13741   60.93   38.74    0.00    0.00   99.67     -  lab003
Average:     1000     14596    0.66    0.66    0.00    0.33    1.32     -  pidstat
```

- Main takeaway: lab003 is taking up all the CPU. It's also using lots of user and system time.
- Why is it taking a lot of system time? Let's check disk I/O.

``` bash
shad@linux:~/linux/perf-labs/src$ iostat -x 1
Linux 6.14.0-1014-azure (linux)         11/08/25        _x86_64_        (2 CPU)
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          30.35    0.00   21.39    0.00    0.00   48.26

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
loop0            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
sda              0.00      0.00     0.00   0.00    0.00     0.00    1.00      4.00     0.00   0.00    1.00     4.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
sdb              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
```

- Disk I/O looks fine. Very low utilization. 

``` bash

shad@linux:~/linux/perf-labs/src$ sar -n DEV 1
Linux 6.14.0-1014-azure (linux)         11/08/25        _x86_64_        (2 CPU)

00:28:11        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
00:28:12           lo     24.00     24.00      2.61      2.61      0.00      0.00      0.00      0.00
00:28:12         eth0    133.00     63.00     14.56    862.66      0.00      0.00      0.00      0.00
00:28:12    enP44461s1    123.00    646.00     14.11    900.25      0.00      0.00      0.00      0.00

00:28:12        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
00:28:13           lo     15.00     15.00      1.68      1.68      0.00      0.00      0.00      0.00
00:28:13         eth0     25.00     28.00      5.10      6.34      0.00      0.00      0.00      0.00
00:28:13    enP44461s1     10.00     29.00      1.59      6.42      0.00      0.00      0.00      0.00

00:28:13        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
00:28:14           lo     14.00     14.00      1.41      1.41      0.00      0.00      0.00      0.00
00:28:14         eth0      7.00      6.00      0.71      0.92      0.00      0.00      0.00      0.00
00:28:14    enP44461s1      7.00      6.00      0.81      0.93      0.00      0.00      0.00      0.00
^C

Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
Average:           lo     17.67     17.67      1.90      1.90      0.00      0.00      0.00      0.00
Average:         eth0     55.00     32.33      6.79    289.97      0.00      0.00      0.00      0.00
Average:    enP44461s1     46.67    227.00      5.50    302.53      0.00      0.00      0.00      0.00
```

- Network I/O looks fine as well.

``` bash
shad@linux:~/linux/perf-labs/src$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 2  0      0 5060416  95432 1173140    0    0    31   878  727    5  5  2 85  8  0  0
 3  0      0 5059680  95432 1173140    0    0     0     4 1205  316 31 22 48  0  0  0
```

- I don't see swapping either.

``` bash
shad@linux:~/linux/perf-labs/src$ ps aux | grep "lab003"
shad       13741 99.3  0.0   2548  1052 pts/1    R+   00:16  28:54 ./lab003

shad@linux:~/linux/perf-labs/src$ sudo strace -p "13741"
strace: Process 13741 attached
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0

# Because strace fills up the terminal, let's limit the output
# 2>&1 redirects stderr to stdout, so we can pipe both to head
shad@linux:~/linux/perf-labs/src$ sudo strace -p `pgrep lab003` 2>&1 | head -10
strace: Process 13741 attached
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
read(3, "", 0)                          = 0
....

```

- Aha! The application is stuck in a loop, requesting 0 bytes over and over.
- Beware, strace could slow down the target process significantly.

## [Linux Performance Tools, Brendan Gregg, part 2 of 2 (a bit more in-depth)](https://www.youtube.com/watch?v=zrr2nUln9Kk)

## Observability Tools: Intermediate

- ss = socket statistics
- iptraf = histogram of network packet sizes (not installed by default)
- iotop = Block device I/O usage by process (needs kernel CONFIG_TASK_IO_ACCOUNTING enabled), not installed by default
- slabtop = kernel slab allocator memory usage. (not installed by default)
- pcstat = show page cache residency by file. (not installed by default.) Useful for database performance analysis.
- perf_events = mutli-tool for cpu/pmc profiling, static & dynamic tracing.

## Benchmarking Tools

- Multi: UnixBench, Imbench, sysbench, perf bench
- FS/disk: dd, hdparm, fio
- App/lib: ab, wrk, jmeter, openssl
- Networking: ping, hping3, iperf, ttcp, traceroute, mtr, pchar

### Benchmarking issues

- ~100% of benchmarks are wrong. Results are usually misleading: You benchmark A, but actually measure B, and conclude you measured C.
- Common mistakes:
    - Testing the wrong target: ex. FS cache instead of disk.
    - Choosing the wrong target: ex. disk instead of FS cache => doesn't resemble real world usage.
    - Invalid results: ex. bugs
- The energy needed to refute benchmarks is multiple orders of magnitude bigger than to run them.

### Active Benchmarking (Method)

1. Run the benchmark for hours
2. While running, analyze and confirm the performance limiter using observability tools:
    - Disk benchmark: run iostat, ...
    - CPU benchmark: run pidstat, perf, flame graphs

And answer the question: Why isn't the result 10x?

- lmbench: CPU, memory, and kernel micro-benchmarks. Eg, memory latency by stride size.
- fio: FS or disk I/O micro-benchmarks. Results include basic latency distribution.
- pchar: Traceroute with bandwidth per hop!

## Tuning Tools

- Generic Interfaces: sysctl, /sys
- Many areas have custom tuning tools:
    - Applications: their own config
    - CPU/scheduler: nice, renice, taskset, ulimit, chcpu
    - Storage I/O: tune2fs, ionice, hdparm, blockdev, ...
    - Network: ethtool, tc, ip, route

### Tuning Methods

Scientific Method:
1. Question
2. Hypothesis
3. Prediction
4. Test
5. Analysis

- Any observational or benchmarking tests you can try before tuning?
- Consider risks, and see previous tools.
- He has a nice tuning tools image on the page above.

## Static Tools

- Static Performance Tuning: Check the static state and configuration of the system.
    - CPU types & flags
    - CPU frequency scaling config
    - Storage devices
    - File system capacity
    - File system and volume configuration
    - Route table
    - State of hardware.
- What can be checked on a system without load.

### Storage Devices

``` bash
shad@linux:~$ cat /proc/scsi/scsi 
Attached devices:
Host: scsi0 Channel: 00 Id: 00 Lun: 00
  Vendor: Msft     Model: Virtual Disk     Rev: 1.0 
  Type:   Direct-Access                    ANSI  SCSI revision: 05
Host: scsi0 Channel: 00 Id: 00 Lun: 01
  Vendor: Msft     Model: Virtual Disk     Rev: 1.0 
  Type:   Direct-Access                    ANSI  SCSI revision: 05
```

- Shows SCSI devices attached to the system.

### Routing Table

``` bash
# IP is google.com
shad@linux:~$ ip route get 64.233.180.102
64.233.180.102 via 10.0.0.1 dev eth0 src 10.0.0.4 uid 1000 
    cache 
```

- Use "ip groute get" to test a given IP.

## Profiling

- Objectives:
    - Profile CPU usage by stack sampling
    - Generate CPU flame graphs
    - Understand gotchas with stacks & symbols.

### CPU Profiling
- Record stacks at a timed interval: simple and effective
    - Pros: Low (deterministic) overhead.
    - Cons: Coarse accuracy, but usually sufficient.

## Tracing
- Objectives:
    - Understand frameworks: tracepoints, kprobes, uprobes
    - Use mainline tracers: ftrace, perf_events, eBPF
    - Awareness of other tracers: SystemTap, LTTng, ktap, sysdig
    - Awareness of what tracing can accomplish (eg, perf-tools)

### Tracing Frameworks: Tracepoints

- Statically placed at logical places in the kernel.
- Provides key event details as a "format" string.

### Tracing Frameworks: +probes
- kprobes: dynamic kernel tracing - function calls, returns, line numbers
- uprobes: dynamic user-level tracing

### eBPF

- Extended BPF: programs on tracepoints
    - High performance filtering: JIT
    - In-kernel summaries: maps
