# ps

[ref](https://www.youtube.com/watch?v=wYwGNgsfN3I&t=5s)

## w/ no options

``` bash
shad@linux:~$ ps
    PID TTY          TIME CMD
  10584 pts/0    00:00:00 bash
  21144 pts/0    00:00:00 ps
```

- TTY is the terminal associated with the process.
- TIME is how much time the process has used the CPU.
- ps with no args shows processes associated with the current terminal.

## w/ options 

``` bash
shad@linux:~$ ps -axjf
   PPID     PID    PGID     SID TTY        TPGID STAT   UID   TIME COMMAND
      1    6119    6119    6119 ?             -1 Ss       0   0:00 /usr/sbin/apache2 -k start
   6119    6122    6119    6119 ?             -1 Sl      33   0:00  \_ /usr/sbin/apache2 -k start
   6119    6123    6119    6119 ?             -1 Sl      33   0:00  \_ /usr/sbin/apache2 -k start
```

- TTY: The terminal associated with the process. `?` means no terminal and is typical for background processes.
- STAT:
    - s: Process leader.
    - S: Process is running on interruptible sleep (ex. waiting for user input and can't be disturbed until it receives that).
    - R: Running.
    - T: Stopped.
    - l: Multi-threaded process.
- UID: User ID of the process owner. 0 is root.

``` bash
shad@linux:~$ ps aux

USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root        6119  0.0  0.0   6880  5004 ?        Ss   18:38   0:00 /usr/sbin/apache2 -k start
www-data    6122  0.0  0.0 1213028 6328 ?        Sl   18:38   0:00 /usr/sbin/apache2 -k start
www-data    6123  0.0  0.0 1213028 6280 ?        Sl   18:38   0:00 /usr/sbin/apache2 -k start
root       10434  0.0  0.1  14740 10488 ?        Ss   19:20   0:00 sshd: shad [priv]
shad       10493  0.0  0.0  15384  7428 ?        S    19:20   0:05 sshd: shad@notty
shad       21840  0.0  0.0  11320  4452 pts/0    R+   21:11   0:00 ps aux
```

- We'll see the USER of these processes too
- Can also see CPU% and MEM% that's being used by that process.