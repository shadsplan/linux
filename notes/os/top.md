# top
[ref](https://www.youtube.com/watch?v=WsR11EGF9PA)

## Basic usage of the top command.
- `top` shows us a snapshot of the resource usage on our system.
- Great for investigating resource contention.

``` bash
shad@linux:~$ top
top - 19:27:56 up  4:01,  2 users,  load average: 0.00, 0.03, 0.02
Tasks: 145 total,   1 running, 144 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.5 us,  0.5 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st 
MiB Mem :   7892.2 total,   6056.8 free,   1534.5 used,    552.6 buff/cache     
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   6357.7 avail Mem                                                                                                                                              
```

- Load average is the utilization of CPU cores over time.
- us: user space (where programs run), sy: kernel space (where the OS runs), ni: nice (priority processes), id: idle, wa: wait (I/O wait) - we generally want this to be low, hi: hardware interrupts, si: software interrupts
- Memory is in MiB (Mebibytes, 1 MiB = 1024 * 1024 bytes) btw, so we have about 8 GiB of RAM, with 6 GiB free and 1.5 GiB used. 552 MiB the cache is using.
- Swap is memory typically located on your disk. Too much of it could imply your system is low on RAM.

``` bash
shad@linux:~$ sudo w
19:02:55 up  3:36,  2 users,  load average: 0.01, 0.02, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU  WHAT
shad              108.redacted     17:49    2:55m  0.00s  0.01s sshd: shad [priv]

shad@linux:~$ sudo who
shad     pts/1        2025-10-28 19:03
```

- `up 4:01` means the system has been up for 4 hours and 1 minute. That's when I started the Azure Linux VM.
- 2 users: Confused as to why that is, because of below. But it could be stale shells due to ssh.

``` bash
PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                               
2485 shad      20   0 1296032  86728  47808 S   0.7   1.1   0:09.47 node                                                                                  
2444 shad      20   0   11.3g 128768  52588 S   0.3   1.6   0:12.40 node                                                                                  
10512 shad      20   0   35184  20152  13452 S   0.3   0.2   0:00.97 code-7d842fb85a                                                                       
10832 shad      20   0   12364   5900   3724 R   0.3   0.1   0:00.01 top                                                                                   
1 root      20   0   22736  13864   9512 S   0.0   0.2   0:02.90 systemd       
```

- Now let's talk about the lower section of the `top` output.
- VIRT - Virtual memory used by the process.
- RES - Physical memory being used by the process.
- SHR - Shared memory used by the process.
- We can sort the processes as well:
    - `M` to sort by memory usage.
    - `P` to sort by CPU usage.
- To kill a process, press `k` and enter the PID.

## What should you pay attention to?
- The uptime `up`, especially if it's unexpectedly low.
- The idle field `id`, sort of a double-edged sword.
    - If it's too high, your system is underutilized.
        - Waste of money?
        - Could also mean the process that should be running is not running. Then go troubleshoot that.
    - If it's too low, your system might be overloaded.
- Swap usage, you don't want it to be high. It implies your system is low on RAM.