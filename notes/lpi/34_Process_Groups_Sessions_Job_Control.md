# Chapter 34. Process Groups, Sessions, and Job Control

- Process groups and sessions form a two-level hierarchical relationship between processes: a process group is a collection of related processes, and a session is a collection of related process groups. 

## Chapter 34.1 Overview

- A process group is a set of one or more processes sharing the same process group identifier (PGID). A process group ID is a number of the same type (pid_t) as a process ID. A process group has a process group leader, which is the process that creates the group and whose process ID becomes the process group ID of the group. A new process inherits its parent’s process group ID.
- A process group has a lifetime, which is the period of time beginning when the leader creates the group and ending when the last member process leaves the group. A process may leave a process group either by terminating or by joining another process group.
- All of the processes in a session share a single controlling terminal. The controlling terminal is established when the session leader first opens a terminal device. A terminal may be the controlling terminal of at most one session.
- At any point in time, one of the process groups in a session is the foreground process group for the terminal, and the others are background process groups. Only processes in the foreground process group can read input from the controlling terminal. When the user types one of the signal-generating terminal characters on the controlling terminal, a signal is sent to all members of the foreground process group. These characters are the interrupt character (usually Control-C), which generates SIGINT; the quit character (usually Control-\), which generates SIGQUIT; and the suspend character (usually Control-Z), which generates SIGTSTP.
- As a consequence of establishing the connection to (i.e., opening) the controlling terminal, the session leader becomes the controlling process for the terminal. The principal significance of being the controlling process is that the kernel sends this process a SIGHUP signal if a terminal disconnect occurs.
- The main use of sessions and process groups is for shell job control. Looking at a
specific example from this domain helps clarify these concepts. For an interactive
login, the controlling terminal is the one on which the user logs in. The login shell
becomes the session leader and the controlling process for the terminal, and is also
made the sole member of its own process group. Each command or pipeline of commands started from the shell results in the creation of one or more processes, and the shell places all of these processes in a new process group. (These processes are initially the only members of that process group, although any child processes that they create will also be members of the group.) A command or pipeline is created as a background process group if it is terminated with an ampersand (&). Otherwise, it becomes the foreground process group. All processes created during the login session are part of the same session.
- Figure 34-1 shows the process group and session relationships between the various processes resulting from the execution of the following commands:

``` bash
shad@linux:~/linux/notes/lpi$ sleep 10 &
[1] 2143
shad@linux:~/linux/notes/lpi$ ps aux | grep 2143
shad        2143  0.0  0.0   6112  1856 pts/4    S    23:06   0:00 sleep 10
shad        2165  0.0  0.0   7076  2136 pts/4    S+   23:06   0:00 grep --color=auto 2143
shad@linux:~/linux/notes/lpi$ ps aux | grep 2143
[1]+  Done                    sleep 10
shad        2192  0.0  0.0   7076  2136 pts/4    S+   23:06   0:00 grep --color=auto 2143
```

## 34.1 Process Groups
``` bash
shad@linux:~/linux/notes$ ps -axjf | grep -E "PPID|apache2"
   PPID     PID    PGID     SID TTY        TPGID STAT   UID   TIME COMMAND
   1461    3055    3054    1461 pts/4       3054 S+    1000   0:00              |                   \_ grep --color=auto -E PPID|apache2
      1    2435    2435    2435 ?             -1 Ss       0   0:00 /usr/sbin/apache2 -k start
   2435    2437    2435    2435 ?             -1 Sl      33   0:00  \_ /usr/sbin/apache2 -k start
   2435    2439    2435    2435 ?             -1 Sl      33   0:00  \_ /usr/sbin/apache2 -k start
``` 

- Each process has a numeric process group ID that defines the process group to which it belongs. A new process inherits its parent’s process group ID.

## 34.3 Sessions
- A session is a collection of process groups. The session membership of a process is defined by its numeric session ID. A new process inherits its parent’s session ID.

## 34.4 Controlling Terminals and Controlling Processes
- All of the processes in a session may have a (single) controlling terminal. Upon creation, a session has no controlling terminal; the controlling terminal is established when the session leader first opens a terminal that is not already the controlling terminal for a session.
- The controlling terminal is inherited by the child of a fork() and preserved across an exec().
- When a session leader opens a controlling terminal, it simultaneously becomes the controlling process for the terminal.

## 34.5 Foreground and Background Process Groups
- The controlling terminal maintains the notion of a foreground process group. Within a session, only one process can be in the foreground at a particular moment; all of the other process groups in the session are background process groups. The foreground process group is the only process group that can freely read and write on the controlling terminal. When one of the signal-generating terminal characters is typed on the controlling terminal, the terminal driver delivers the corresponding signal to the members of the foreground process group.

## 34.8 Summary
- Sessions and process groups (also known as jobs) form a two-level hierarchy of processes: a session is a collection of process groups, and a process group is a collection of processes. A session leader is the process that created the session using setsid(). Similarly, a process group leader is the process that created the group using setpgid(). All of the members of a process group share the same process group ID (which is the same as the process group ID of the process group leader), and all processes in the process groups that constitute a session have the same session ID (which is the same as the ID of the session leader). Each session may have a controlling terminal (/dev/tty), which is established when the session leader opens a terminal device. Opening the controlling terminal also causes the session leader to become the controlling process for the terminal.
- The notion of the terminal’s foreground job is also used to arbitrate terminal I/O requests. Only processes in the foreground job may read from the controlling terminal.
- When a terminal disconnect occurs, the kernel delivers a SIGHUP signal to the controlling process to inform it of the fact. Such an event may result in a chain reaction. whereby a SIGHUP signal is delivered to many other processes. First, if the controlling process is a shell (as is typically the case), then, before terminating, the shell sends SIGHUP to each of the process groups it has created. Second, if delivery of SIGHUP results in termination of a controlling process, then the kernel also sends SIGHUP to all of the members of the foreground process group of the controlling terminal.