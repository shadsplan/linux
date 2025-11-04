# Short Notes

## Chapter 7. Memory Allocation
- Using the malloc family of functions, a process can dynamically allocate and release memory on the heap.
- The alloca() function allocates memory on the stack. This memory is automatically deallocated when the function that calls alloca() returns.

## Chapter 8. Users and Groups
- Each user has a unique login name and an associated numeric user ID. Users can belong to one or more groups, each of which also has a unique name and an associated numeric identifier. The primary purpose of these identifiers is to establish ownership of various system resources (e.g., files) and permissions for accessing them.
- A user’s name and ID are defined in the /etc/passwd file, which also contains other information about the user. A user’s group memberships are defined by fields in the /etc/passwd and /etc/group files. A further file, /etc/shadow, which can be read only by privileged processes, is used to separate the sensitive password information from the publicly available user information in /etc/passwd.

## Chapter 17. Access Control Lists
From version 2.6 onward, Linux supports ACLs. ACLs extend the traditional UNIX file permissions model, allowing file permissions to be controlled on a per-user and per-group basis.

## Chapter 19. Monitoring File Events

- Some applications need to be able to monitor files or directories in order to determine whether events have occurred for the monitored objects.
- Starting with kernel 2.6.13, Linux provides the inotify mechanism, which allows an application to monitor file events.

### 19.1 Overview
- The inotify mechanism can be used to monitor files or directories. When monitoring a directory, the application will be informed about events for the directory itself and for files inside the directory.
- The inotify monitoring mechanism is not recursive. If an application wants to monitor events within an entire directory subtree, it must issue inotify_add_watch() calls for each directory in the tree.

### 19.7 Summary
The Linux-specific inotify mechanism allows an application to obtain notifications when events (files are opened, closed, created, deleted, modified, renamed, and so on) occur for a set of monitored files and directories.

## Chapter 21. Signals: Signal Handlers
In general, it is preferable to write simple signal handlers. One important reason for this is to reduce the risk of creating race conditions. Two common designs for signal handlers are the following:
- The signal handler sets a global flag and exits. The main program periodically checks this flag and, if it is set, takes appropriate action. (If the main program cannot perform such periodic checks because it needs to monitor one or more file descriptors to see if I/O is possible, then the signal handler can also write a single byte to a dedicated pipe whose read end is included among the file descriptors monitored by the main program.)
- The signal handler performs some type of cleanup and then either terminates the process or uses a nonlocal goto (Section 21.2.1) to unwind the stack and return control to a predetermined location in the main program.

### 21.6 Summary
- Because signals are not queued, a signal handler must sometimes be coded to deal with the possibility that multiple events of a particular type have occurred, even though only one signal was delivered. The issue of reentrancy affects how we can update global variables and limits the set of functions that we can safely call from a signal handler.
- Instead of returning, a signal handler can terminate in a variety of other ways, including calling _exit(), terminating the process by sending a signal (kill(), raise(), or abort()), or performing a nonlocal goto.

## Chapter 22. Signals: Advanced Features
### 22.1 Core Dump Files
- Certain signals cause a process to create a core dump and terminate. A core dump is a file containing a memory image of the process at the
time it terminated. (The term core derives from an old memory technology.) This memory image includes the contents of the process’s address space, including the program code, data, heap, and stack segments.

### 22.12 Interprocess Communication with Signals
Not a great idea, because:
- The asynchronous nature of signals means that we face various problems, including reentrancy requirements, race conditions, and the correct handling of global variables from signal handlers.
-  Standard signals are not queued. Even for realtime signals, there are upper limits on the number of signals that may be queued. This means that in order to avoid loss of information, the process receiving the signals must have a method of informing the sender that it is ready to receive another signal. The most obvious method of doing this is for the receiver to send a signal to the sender.
- Signals carry only a limited amount of information: the
signal number, and in the case of realtime signals, a word (an integer or a pointer). This means that if more information needs to be conveyed from the sender to the receiver, some other method must be used to transfer the rest of additional data. This low bandwidth makes signals slow by comparison with other methods of IPC such as pipes.

## Chapter 25. Process Termination
### 25.1 Terminating a process: _exit() and exit()
- A process may terminate in two general ways.
    - One of these is abnormal termination, caused by the delivery of a signal whose default action is to terminate the process (with or without a core dump).
    - Alternatively, a process can terminate normally, using the _exit() system call.
- The status argument given to _exit() defines the termination status of the process, which is available to the parent of this process when it calls wait().

### 25.6 Summary
- A process can terminate either abnormally or normally. Abnormal termination occurs on delivery of certain signals, some of which also cause the process to produce a core dump file.
- By convention, a status of 0 is used to indicate successful termination, and a nonzero status indicates unsuccessful termination.
- As part of both normal and abnormal process termination, the kernel performs various cleanup steps. Terminating a process normally by calling exit() additionally causes exit handlers registered using atexit() and on_exit() to be called (in reverse order of registration), and causes stdio buffers to be flushed.

## Chapter 26. Program Execution

### 27.1 Executing a New Program: execve()
- The execve() system call loads a new program into a process’s memory. During this operation, the old program is discarded, and the process’s stack, data, and heap are replaced by those of the new program. After executing various C library run-time startup code and program initialization code the new program commences execution at its main() function.

### 27.3 Interpreter Scripts
- An interpreter is a program that reads commands in text form and executes them. (This contrasts with a compiler, which translates input source code into a machine language that can then be executed on a real or virtual machine.) Examples of interpreters include the various UNIX shells and programs such as awk, sed, perl, python, and ruby. In addition to being able to read and execute commands interactively, interpreters usually provide a facility to read and execute commands from a text file, referred to as a script.
- UNIX kernels allow interpreter scripts to be run in the same way as a binary program file, as long as two requirements are met. First, execute permission must be enabled for the script file. Second, the file must contain an initial line that specifies the pathname of the interpreter to be used to run the script. This line has the following form: `#! interpreter-path [ optional-arg ]`
    - As an example, UNIX shell scripts usually begin with the following line, which specifies that the shell is to be used to execute the script: `#!/bin/sh`

### 27.8 Summary
- Using execve(), a process can replace the program that it is currently running by a new program. Arguments to the execve() call allow the specification of the argument list (argv) and environment list for the new program. Various similarly named library functions are layered on top of execve() and provide different interfaces to the same functionality.
- All of the exec() functions can be used to load a binary executable file or to execute an interpreter script. When a process execs a script, the script’s interpreter program replaces the program currently being executed by the process. The script’s interpreter is normally identified by an initial line (starting with the characters #!) in the script that specifies the pathname of the interpreter. If no such line is present, then the script is executable only via execlp() or execvp(), and these functions exec the shell as the script interpreter.