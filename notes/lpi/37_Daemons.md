# Chapter 37. Daemons

## 37.1 Overview

A daemon is a process with the following characteristics:
- It is long-lived. Often, a daemon is created at system startup and runs until the system is shut down.
- It runs in the background and has no controlling terminal. The lack of a controlling terminal ensures that the kernel never automatically generates any job-control or terminal-related signals (such as SIGINT, SIGTSTP, and SIGHUP) for a daemon.

Daemons are written to carry out specific tasks, as illustrated by the following examples:
- cron: a daemon that executes commands at a scheduled time.
- sshd: the secure shell daemon, which permits logins from remote hosts using a secure communications protocol.
- httpd: the HTTP server daemon (Apache), which serves web pages.
- inetd: the Internet superserver daemon (described in Section 60.5), which listens for incoming network connections on specified TCP/IP ports and launches appropriate server programs to handle these connections.

It is a convention (not universally observed) that daemons have names ending with the letter d.

## 37.2 Creating a Daemon

To become a daemon, a program performs the following steps:

1. Perform a fork(), after which the parent exits and the child continues. (As a consequence, the daemon becomes a child of the init process.) This step is done for two reasons:
– Assuming the daemon was started from the command line, the parent's termination is noticed by the shell, which then displays another shell prompt and leaves the child to continue in the background.
– The child process is guaranteed not to be a process group leader, since it inherited its process group ID from its parent and obtained its own unique process ID, which differs from the inherited process group ID. This is required in order to be able to successfully perform the next step.
2. The child process calls setsid() (Section 34.3) to start a new session and detach itself from the terminal.

37.3 Guidelines for Writing Daemons
- Since daemons are long-lived, we must be particularly wary of possible memory leaks (Section 7.1.3) and file descriptor leaks (where an application fails to close all of the file descriptors it opens). If such bugs affect a daemon, the only remedy is to kill it and restart it after (fixing the bug).
- Many daemons need to ensure that just one instance of the daemon is active at one time. For example, it makes no sense to have two copies of the cron daemon both trying to execute scheduled jobs.

## 37.5 Logging Messages and Errors using syslog
When writing a daemon, one problem we encounter is how to display error messages. Since a daemon runs in the background, we can't display messages on an associated terminal, as we would typically do with other programs. One possible alternative is to write messages to an application-specific log file, as is done in the program in Listing 37-3. The main problem with this approach is that it is difficult for a system administrator to manage multiple application log files and monitor them all for error messages. The syslog facility was devised to address this problem.

### 37.5.1 Overview
- The syslog facility provides a single, centralized logging facility that can be used to log messages by all applications on the system. An overview of this facility is provided in Figure 37-1.
- The syslog facility has two principal components: the syslogd daemon and the syslog(3) library function.

- The System Log daemon, syslogd, accepts log messages from two different sources: a UNIX domain socket, /dev/log, which holds locally produced messages, and (if enabled) an Internet domain socket (UDP port 514), which holds messages sent across a TCP/IP network.
- Each message processed by syslogd has a number of attributes, including a facility, which specifies the type of program generating the message, and a level, which specifies the severity (priority) of the message. The syslogd daemon examines the facility and level of each message, and then passes it along to any of several possible destinations according to the dictates of an associated configuration file, /etc/syslog.conf. Possible destinations include a terminal or virtual console, a disk file, a FIFO, one or more (or all) logged-in users, or a process (typically another syslogd
daemon) on another system connected via a TCP/IP network. (Sending the message to a process on another system is useful for reducing administrative overhead by consolidating messages from multiple systems to a single location.) A single message may be sent to multiple destinations (or none at all), and messages with different combinations of facility and level can be targeted to different destinations or to different instances of destinations (i.e., different consoles, different disk files, and so on).

- The syslog(3) library function can be used by any process to log a message. This function, which we describe in detail in a moment, uses its supplied arguments to construct a message in a standard format that is then placed on the /dev/log socket for reading by syslogd.

## 37.6 Summary
- A daemon is a long-lived process that has no controlling terminal (i.e., it runs in the background). Daemons perform specific tasks, such as providing a network login facility or serving web pages. To become a daemon, a program performs a standard sequence of steps, including calls to fork() and setsid().
- Where appropriate, daemons should correctly handle the arrival of the SIGTERM and SIGHUP signals. The SIGTERM signal should result in an orderly shutdown of the daemon, while the SIGHUP signal provides a way to trigger the daemon to reinitialize itself by rereading its configuration file and reopening any log files it may be using.
- The syslog facility provides a convenient way for daemons (and other applications) to log error and other messages to a central location. These messages are processed by the syslogd daemon, which redistributes the messages according to the dictates of the syslogd.conf configuration file. Messages may be redistributed to a number of targets, including terminals, disk files, logged-in users, and, via a TCP/IP network, to other processes on remote hosts (typically other syslogd daemons).