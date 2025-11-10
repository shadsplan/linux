# Performance Knowledge

## [Linux Performance Analysis in 60,000 Milliseconds](https://netflixtechblog.com/linux-performance-analysis-in-60-000-milliseconds-accc10403c55)

[Video](https://www.youtube.com/watch?v=ZdVpKx6Wmc8)

Look for errors and saturation metrics, as they are both easy to interpret, and then resource utilization. Saturation is where a resource has more load than it can handle, and can be exposed either as the length of a request queue, or time spent waiting.

 The metrics these commands expose will help you complete some of the [USE Method](http://www.brendangregg.com/usemethod.html): a methodology for locating performance bottlenecks. This involves checking utilization, saturation, and error metrics for all resources (CPUs, memory, disks, e.t.c.). Also pay attention to when you have checked and exonerated a resource, as by process of elimination this narrows the targets to study, and directs any follow on investigation.

 ### 1. `uptime`

 ``` bash
 # To check how the load averages have changed over time, to let me know if the system is getting busier or quieter.
shad@linux:~/linux/perf-labs/src$ uptime
02:56:35 up  8:40,  2 users,  load average: 0.06, 0.29, 0.71
```

- This is a quick way to view the load averages, which indicate the number of tasks (processes) wanting to run. On Linux systems, these numbers include processes wanting to run on CPU, as well as processes blocked in uninterruptible I/O (usually disk I/O). **This gives a high level idea of resource load (or demand), but can’t be properly understood without other tools. Worth a quick look only.**
- The three numbers are exponentially damped moving sum averages with a 1 minute, 5 minute, and 15 minute constant. The three numbers give us some idea of how load is changing over time. For example, if you’ve been asked to check a problem server, and the 1 minute value is much lower than the 15 minute value, then you might have logged in too late and missed the issue.

### 2. `sudo dmesg -H | tail`

``` bash
# To check if the system's letting me know why there's a performance issue.
shad@linux:~$ sudo dmesg -H | tail
[  +0.113671] mlx5_core 51b0:00:02.0 enP20912s1: Link up
[  +0.031463] hv_netvsc 6045bdef-f34d-6045-bdef-f34d6045bdef eth0: Data path switched to VF: enP20912s1
[  +0.040429] 8021q: 802.1Q VLAN Support v1.8
[  +0.266240] hv_netvsc 6045bdef-f34d-6045-bdef-f34d6045bdef eth0: Data path switched from VF: enP20912s1
[  +0.225353] mlx5_core 51b0:00:02.0 enP20912s1: Link up
[  +0.023502] hv_netvsc 6045bdef-f34d-6045-bdef-f34d6045bdef eth0: Data path switched to VF: enP20912s1
[  +0.662024] EXT4-fs (sdb1): mounted filesystem 42c07d73-7f18-4df3-b2c9-885e73f84b08 r/w with ordered data mode. Quota mode: none.
[  +0.998761] loop0: detected capacity change from 0 to 8
[  +1.779734] systemd-journald[141]: /var/log/journal/aa9e0761fad9437b96177cffcbf41daa/user-1000.journal: Journal file uses a different sequence number ID, rotating.
[ +37.372397] hv_balloon: Max. dynamic memory size: 8192 MB
```

- This views the last 10 system messages, if there are any. Look for errors that can cause performance issues. The example above includes the oom-killer, and TCP dropping a request.

### 3. `vmstat 1`

``` bash
# To get an overall sense of the system, including CPU utilization or if memory is on fire.
shad@linux:~/linux/perf-labs/src$ vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- -------cpu-------
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st gu
 0  0      0 5089336  92516 938244    0    0    26   618  658    4  5  2 88  5  0  0
 1  0      0 5088872  92516 938244    0    0     0     4  340  340  1  1 99  0  0  0
```

- Short for virtual memory stat, this command prints a summary of key server statistics on each line.
- Columns to check:
    - r: Number of processes running on CPU and waiting for a turn. This provides a better signal than load averages for determining CPU saturation, as it does not include I/O. To interpret: an “r” value greater than the CPU count is saturation.
    - free: Free memory in kilobytes. If there are too many digits to count, you have enough free memory. The “free -m” command, included as command 7, better explains the state of free memory.
    - si, so: Swap-ins and swap-outs. If these are non-zero, you’re out of memory.
    - us, sy, id, wa, st: These are breakdowns of CPU time, on average across all CPUs. They are user time, system time (kernel), idle, wait I/O, and stolen time.
- The CPU time breakdowns will confirm if the CPUs are busy, by adding user + system time. A constant degree of wait I/O points to a disk bottleneck; this is where the CPUs are idle, because tasks are blocked waiting for pending disk I/O. You can treat wait I/O as another form of CPU idle, one that gives a clue as to why they are idle.
- System time is necessary for I/O processing. A high system time average, over 20%, can be interesting to explore further: perhaps the kernel is processing the I/O inefficiently.

### 4. `mpstat -P ALL 1`

``` bash
# To check the CPU balance.
shad@linux:~/linux/perf-labs/src$ mpstat -P ALL 1
Linux 6.14.0-1014-azure (linux)         11/09/25        _x86_64_        (2 CPU)

03:01:03     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
03:01:04     all    1.01    0.00    1.01    0.00    0.00    0.00    0.00    0.00    0.00   97.99
03:01:04       0    0.00    0.00    1.01    0.00    0.00    0.00    0.00    0.00    0.00   98.99
03:01:04       1    2.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00   97.00

03:01:04     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
03:01:05     all    2.99    0.00    1.49    1.00    0.00    0.00    0.00    0.00    0.00   94.53
03:01:05       0    4.95    0.00    1.98    0.99    0.00    0.00    0.00    0.00    0.00   92.08
03:01:05       1    1.00    0.00    1.00    1.00    0.00    0.00    0.00    0.00    0.00   97.00

^C
Average:     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
Average:     all    2.17    0.00    0.83    0.33    0.00    0.00    0.00    0.00    0.00   96.66
Average:       0    3.01    0.00    1.00    0.33    0.00    0.00    0.00    0.00    0.00   95.65
Average:       1    1.33    0.00    0.67    0.33    0.00    0.00    0.00    0.00    0.00   97.67
```

This command prints CPU time breakdowns per CPU, which can be used to check for an imbalance. A single hot CPU can be evidence of a single-threaded application.

### 5. `pidstat 1`

``` bash
# To see who that is
shad@linux:~/linux/perf-labs/src$ pidstat 1
Linux 6.14.0-1014-azure (linux)         11/09/25        _x86_64_        (2 CPU)

03:01:49      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
03:01:50     1000      1352    0.99    0.00    0.00    0.00    0.99     1  node
03:01:50     1000      2819    4.95    0.00    0.00    0.00    4.95     0  node

03:01:50      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
03:01:51     1000      1352    1.00    0.00    0.00    0.00    1.00     0  node
03:01:51     1000      2819   20.00    2.00    0.00    0.00   22.00     0  node
03:01:51     1000    312470    0.00    1.00    0.00    0.00    1.00     1  pidstat
^C

Average:      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
Average:     1000      1352    0.40    0.00    0.00    0.00    0.40     -  node
Average:     1000      1434    0.20    0.00    0.00    0.00    0.20     -  node
Average:     1000      2771    0.20    0.00    0.00    0.20    0.20     -  sshd
Average:     1000      2790    0.20    0.20    0.00    0.00    0.40     -  code-7d842fb85a
Average:     1000      2819   37.33    0.80    0.00    5.99   38.12     -  node
Average:     1000    312470    0.20    0.40    0.00    0.00    0.60     -  pidstat
```

### 6. `iostat -xh 1`

``` bash
# To check Disk I/O, I'm looking at await and %util.
shad@linux:~$ iostat -xh 1
Linux 6.14.0-1014-azure (linux)         11/10/25        _x86_64_        (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.0%    0.0%    0.3%    0.1%    0.0%   97.6%

     r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     1.0k loop0
    1.53     67.3k     0.42  21.6%    3.86    43.9k sda
    0.04      1.2k     0.00   0.3%    0.22    31.8k sdb

     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     0.0k loop0
    1.16     15.9k     0.85  42.2%    0.83    13.7k sda
    0.05     42.0k     0.13  74.6%    2.50   922.6k sdb

     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     0.0k loop0
    0.09     23.3k     0.01   5.1%    3.35   247.8k sda
    0.00      2.1M     0.00  47.1%    0.22     1.8G sdb

     f/s f_await  aqu-sz  %util Device
    0.00    0.00    0.00   0.0% loop0
    0.06    3.16    0.01   0.2% sda
    0.00    0.00    0.00   0.0% sdb


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.5%    0.0%    0.5%    0.0%    0.0%   99.0%

     r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     0.0k loop0
    0.00      0.0k     0.00   0.0%    0.00     0.0k sda
    0.00      0.0k     0.00   0.0%    0.00     0.0k sdb

     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     0.0k loop0
    0.00      0.0k     0.00   0.0%    0.00     0.0k sda
    0.00      0.0k     0.00   0.0%    0.00     0.0k sdb

     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     0.0k loop0
    0.00      0.0k     0.00   0.0%    0.00     0.0k sda
    0.00      0.0k     0.00   0.0%    0.00     0.0k sdb

     f/s f_await  aqu-sz  %util Device
    0.00    0.00    0.00   0.0% loop0
    0.00    0.00    0.00   0.0% sda
    0.00    0.00    0.00   0.0% sdb


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.0%    0.0%    1.5%    0.0%    0.0%   97.5%

     r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     0.0k loop0
    0.00      0.0k     0.00   0.0%    0.00     0.0k sda
    0.00      0.0k     0.00   0.0%    0.00     0.0k sdb

     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     0.0k loop0
    0.00      0.0k     0.00   0.0%    0.00     0.0k sda
    0.00      0.0k     0.00   0.0%    0.00     0.0k sdb

     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz Device
    0.00      0.0k     0.00   0.0%    0.00     0.0k loop0
    0.00      0.0k     0.00   0.0%    0.00     0.0k sda
    0.00      0.0k     0.00   0.0%    0.00     0.0k sdb

     f/s f_await  aqu-sz  %util Device
    0.00    0.00    0.00   0.0% loop0
    0.00    0.00    0.00   0.0% sda
    0.00    0.00    0.00   0.0% sdb
```


This is a great tool for understanding block devices (disks), both the workload applied and the resulting performance.

- r/s, w/s, rkB/s, wkB/s: These are the delivered reads, writes, read Kbytes, and write Kbytes per second to the device. Use these for workload characterization. A performance problem may simply be due to an excessive load applied.
- await: The average time for the I/O in milliseconds. This is the time that the application suffers, as it includes both time queued and time being serviced. Larger than expected average times can be an indicator of device saturation, or device problems.
- avgqu-sz: The average number of requests issued to the device. Values greater than 1 can be evidence of saturation (although devices can typically operate on requests in parallel, especially virtual devices which front multiple back-end disks.)
- %util: Device utilization. This is really a busy percent, showing the time each second that the device was doing work. **Values greater than 60% typically lead to poor performance (which should be seen in await), although it depends on the device. Values close to 100% usually indicate saturation.**

If the storage device is a logical disk device fronting many back-end disks, then 100% utilization may just mean that some I/O is being processed 100% of the time, however, the back-end disks may be far from saturated, and may be able to handle much more work.

Bear in mind that poor performing disk I/O isn’t necessarily an application issue. Many techniques are typically used to perform I/O asynchronously, so that the application doesn’t block and suffer the latency directly (e.g., read-ahead for reads, and buffering for writes).

### 7. `free -wh`

``` bash
shad@linux:~/linux/perf-labs/src$ free -wh
               total        used        free      shared     buffers       cache   available
Mem:           7.7Gi       2.1Gi       4.9Gi       4.3Mi        90Mi       916Mi       5.7Gi
Swap:             0B          0B          0B
```

The right two columns show:

- buffers: For the buffer cache, used for block device I/O.
- cached: For the page cache, used by file systems.

We just want to check that these aren’t near-zero in size, which can lead to higher disk I/O (confirm using iostat), and worse performance.

### 8. `sar -n DEV 1 -h`

``` bash
# Network throughput
shad@linux:~$ sar -h -n DEV 1
Linux 6.14.0-1014-azure (linux)         11/10/25        _x86_64_        (2 CPU)

00:01:12      rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil IFACE
00:01:13        25.00     25.00      2.2k      2.2k      0.00      0.00      0.00      0.0% lo
00:01:13        11.00     10.00    984.0B      1.5k      0.00      0.00      0.00      0.0% eth0
00:01:13        11.00     10.00      1.1k      1.5k      0.00      0.00      0.00      0.0% enP20912s1

00:01:13      rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil IFACE
00:01:14        14.00     14.00      2.0k      2.0k      0.00      0.00      0.00      0.0% lo
00:01:14        35.00     27.00      8.5k    179.4k      0.00      0.00      0.00      0.0% eth0
00:01:14        34.00    142.00      8.7k    186.8k      0.00      0.00      0.00      0.0% enP20912s1

00:01:14      rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil IFACE
00:01:15        20.00     20.00      2.3k      2.3k      0.00      0.00      0.00      0.0% lo
00:01:15        23.00     22.00      8.5k      9.6k      0.00      0.00      0.00      0.0% eth0
00:01:15        23.00     26.00      8.6k      9.9k      0.00      0.00      0.00      0.0% enP20912s1

Average:      rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil IFACE
Average:        19.67     19.67      2.2k      2.2k      0.00      0.00      0.00      0.0% lo
Average:        23.00     19.67      6.0k     63.5k      0.00      0.00      0.00      0.0% eth0
Average:        22.67     59.33      6.1k     66.1k      0.00      0.00      0.00      0.0% enP20912s1
```

- Use this tool to check network interface throughput: rxkB/s (Total number of kilobytes received per second) and txkB/s (Total number of kilobytes transmitted per second), as a measure of workload, and also to check if any limit has been reached.
- This version also has %ifutil for device utilization (max of both directions for full duplex).

### 9. `sar -n TCP,ETCP 1 -h`

``` bash
# TCP statistics
# I'm doing some active outbound connections, also seeing some retransmissions.
shad@linux:~$ sar -n TCP,ETCP 1 -h
Linux 6.14.0-1014-azure (linux)         11/10/25        _x86_64_        (2 CPU)

00:17:35     active/s passive/s    iseg/s    oseg/s
00:17:36         1.00      0.00     92.00    177.00

00:17:35     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
00:17:36         0.00      0.00      0.00      0.00      0.00

00:17:36     active/s passive/s    iseg/s    oseg/s
00:17:37         4.00      0.00     45.00     49.00

00:17:36     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
00:17:37         0.00      0.00      0.00      0.00      0.00

Average:     active/s passive/s    iseg/s    oseg/s
Average:         1.75      0.00     68.00     90.50

Average:     atmptf/s  estres/s retrans/s isegerr/s   orsts/s
Average:         0.00      0.00      0.00      0.00      0.00
^C
```

This is a summarized view of some key TCP metrics. These include:

- active/s: Number of locally-initiated TCP connections per second (e.g., via connect()).
- passive/s: Number of remotely-initiated TCP connections per second (e.g., via accept()).
- retrans/s: Number of TCP retransmits per second.

- The active and passive counts are often useful as a rough measure of server load: number of new accepted connections (passive), and number of downstream connections (active). It might help to think of active as outbound, and passive as inbound, but this isn’t strictly true (e.g., consider a localhost to localhost connection).
- Retransmits are a sign of a network or server issue; it may be an unreliable network (e.g., the public Internet), or it may be due a server being overloaded and dropping packets.
- Example shows one new TCP connection per-second.

### 10. `top`

``` bash
# goat
shad@linux:~/linux/perf-labs/src$ top
```
- The top command includes many of the metrics we checked earlier. It can be handy to run it to see if anything looks wildly different from the earlier commands, which would indicate that load is variable.
- **A downside to top is that it is harder to see patterns over time, which may be more clear in tools like vmstat and pidstat, which provide rolling output. Evidence of intermittent issues can also be lost if you don’t pause the output quick enough (Ctrl-S to pause, Ctrl-Q to continue), and the screen clears.**

---

## [Linux Performance Troubleshooting Demos](https://www.youtube.com/watch?v=rwVLa9me7e4)

## Demo 1: The latency of my application has increased!

These notes are taken from [Brendan Gregg's Linux Performance Tools video](../..//perf-labs/README.md).

### Step 1: `top`

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
```

- 1.9% user time, 2.6% system time, 48.3% idle, 46.9% I/O wait.
- High amounts of idle CPU means that we're not getting saturation on CPU utilization, this softly says that the issue may not be CPU bound.

### Step 2: `vmstat 1`

``` bash
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

- The output for the first row is different, because it only shows a partial summary of the averages since boot, so we can ignore that.
- r column = number of processes that are either currently running on CPU or waiting for a turn.
    - If greater than number of CPUs, then the CPU is saturated.
    - Large r values would be a cause for concern.
    - In our example, they are in a small range, so CPU saturation is unlikely.
- memory: overview of memory:
    - In our example, each free/buff/cache values are comfortably above zero
- swap: swap memory, when some memory is moved to hard drive. Generally slow, used as last resort. If swap values are non-zero, we'd know that we've run out of memory.
    - in our example, swap values are zero. These outputs suggest we have enough memory in our system.
- cpu: us: user time, sy: system time, id: idle time, wa: I/O wait time, st: stolen time (time spent in virtualized environments)
    -  **In our example, we see the wait I/O value is consistent around 40-50%, which is high. This indicates a disk bottleneck: CPUs are idle because tasks are blocked while waiting for the disk.**

### Step 3: `mpstat -p ALL 1`

``` bash
# Purpose: Per-CPU (processor) statistics
# Layer: CPU / processor level
shad@linux:~/linux/perf-labs/src$ mpstat -P ALL 1
Linux 6.14.0-1014-azure (linux)         11/09/25        _x86_64_        (2 CPU)

01:39:58     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
01:39:59     all    1.54    0.00    2.56   47.18    0.00    0.00    0.00    0.00    0.00   48.72
01:39:59       0    1.02    0.00    2.04    8.16    0.00    0.00    0.00    0.00    0.00   88.78
01:39:59       1    2.06    0.00    3.09   86.60    0.00    0.00    0.00    0.00    0.00    8.25

01:39:59     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
01:40:00     all    1.01    0.00    2.53   46.97    0.00    0.51    0.00    0.00    0.00   48.99
01:40:00       0    0.00    0.00    2.02    7.07    0.00    0.00    0.00    0.00    0.00   90.91
01:40:00       1    2.02    0.00    3.03   86.87    0.00    1.01    0.00    0.00    0.00    7.07

01:40:00     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
01:40:01     all    2.05    0.00    1.54   48.21    0.00    0.00    0.00    0.00    0.00   48.21
01:40:01       0    1.02    0.00    2.04    7.14    0.00    0.00    0.00    0.00    0.00   89.80
01:40:01       1    3.09    0.00    1.03   89.69    0.00    0.00    0.00    0.00    0.00    6.19
^C
```

- Checks CPUs
- Prints time breakdowns per CPU, which can be used to check for imbalances.
- A single hot CPU could indicate a single-threaded application causing issues.
- In our example, CPU 1 has very high I/O wait (86-89%), while CPU 0 has low I/O wait (7-8%).
    - This indicates an imbalance, where some CPUs are waiting on I/O while others are idle.

### Step 4: `iostat -x 1`

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

- iostat is a great tool for understanding how the disks are running.
- most important statistic is the %util
    - In our example, sda has %util values of 95-96%, indicating that the disk is saturated.

> %util values greater than 60% typically lead to poor performance. Whereas systems can generally run well with max CPU. Why? Because Kernel understands priority and can rip threads off the CPU very quickly if it needs to run other threads. But this is not as true for disks, it is harder to send I/O with higher priority dispatch if it already doing something else. This is not relevant for SSDs though.

Great, this is what we're looking for. The high %util on the disk indicates that the disk are the most likely culprit for why our application has more latency. For the sake of completion, let's check the network I/O as well.

### Step 5: `sar -n DEV 1`

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

- Not much utilization here, so there is nothing to report.

**So in summary, we can point to disks as the most likely factor for causing performance issues for this demo.**

## Demo 2: The application is taking forever...

### Step 1: `vmstat 1`

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
```

- r column looks good. No processes are waiting for CPU.
- Memory seems fine as well, plenty of free buffer and cache to go around. Swap is zero too.
- Noticing a good amount of user / system time. And high idle time too.
- wa (I/O wait) is zero, so disks are not the issue here. Compared to the previous demo, where wa was high.
- Let's dive deeper on the CPU side of things.

### Step 2: `mpstat 1`

``` bash
shad@linux:~/linux/perf-labs/src$ mpstat 1
Linux 6.14.0-1014-azure (linux)         11/09/25        _x86_64_        (2 CPU)

02:23:45     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
02:23:46     all   35.32    0.00   16.42    0.00    0.00    0.00    0.00    0.00    0.00   48.26
02:23:47     all   36.00    0.00   17.00    0.00    0.00    0.00    0.00    0.00    0.00   47.00
02:23:48     all   36.68    0.00   16.58    0.00    0.00    0.00    0.00    0.00    0.00   46.73
^CAverage:     all   36.00    0.00   16.67    0.00    0.00    0.00    0.00    0.00    0.00   47.33
```

- We see the same thing, where lots of system and user time are being utilized.
- Let's find out which processes are consuming the CPU.

### Step 3: `pidstat 1`

``` bash
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

- We see that it is our application using up our CPU utilization.
- Let's drive down what is causing so much system time to be utilized, starting with disk I/O.

### Step 4: `iostat -x 1`

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

- No disk activity (await, %util), so disks are not the issue here. Let's check the network io next.

### Step 5: `sar -n DEV 1`

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

- Network I/O looks fine as well (rxKB/s and txKB/s are low, %ifutil is zero).
- We have checked swap, disk, and network causes.
- Given higher system time, can check strace next to see what syscalls are being made.

### Step 6: sudo strace -tp `pgrep lab003` 2>&1 | head -10

``` bash
shad@linux:~/linux/perf-labs/src$ sudo strace -tp `pgrep lab003` 2>&1 | head -10
strace: Process 10029 attached
02:34:13 read(3, "", 0)                 = 0
02:34:13 read(3, "", 0)                 = 0
02:34:13 read(3, "", 0)                 = 0
02:34:13 read(3, "", 0)                 = 0
02:34:13 read(3, "", 0)                 = 0
02:34:13 read(3, "", 0)                 = 0
02:34:13 read(3, "", 0)                 = 0
02:34:13 read(3, "", 0)                 = 0
02:34:13 read(3, "", 0)                 = 0
...
```
- Aha! The application is stuck in a loop, requesting 0 bytes over and over.
- Beware, strace could slow down the target process significantly.

