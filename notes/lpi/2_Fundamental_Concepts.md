# Chapter 2. Fundamental Concepts

## 2.1 The Core Operating System: The Kernel
### Tasks performed by the Linux Kernel
- Linux Kernel: The central software that manages and allocates computer resources (i.e., the CPU, RAM, and devices). Tasks performed include:
    - Process Scheduling:
        - CPUs execute the instructions of programs.
        - Linux is a preemptive multitasking operating system, Multitasking means that multiple processes (i.e., running programs) can simultaneously reside in memory and each may receive use of the CPU(s). Preemptive means that the rules governing which processes receive use of the CPU and for how long are determined by the kernel process scheduler (rather than by the processes themselves).
    - Memory Management:
        - Processes are isolated from one another and from the kernel, so that one process can’t read or modify the memory of another process or the kernel.
        - Only part of a process needs to be kept in memory, thereby lowering the memory requirements of each process and allowing more processes to be held in RAM simultaneously. This leads to better CPU utilization, as the kernel can swap processes in and out of memory as needed. held in RAM simultaneously. This leads to better CPU utilization.
    - Provision of a file system: The kernel provides a file system on disk, allowing files to be created, retrieved, updated, deleted, and so on.
    - Creation and termination of processes: The kernel can load a new program into memory, providing it with the resources (e.g., CPU, memory, and access to files) that it needs in order to run. Such an instance of a running program is termed a process. Once a process has completed execution, the kernel ensures that the resources it uses are freed for subsequent reuse by later programs.
    - Access to devices: The devices (mice, monitors, keyboards, disk and tape drives, and so on) attached to a computer allow communication of information between the computer and the outside world, permitting input, output, or both. The kernel provides programs with an interface that standardizes and simplifies access to devices, while at the same time arbitrating access by multiple processes to each device.
    - Networking: The kernel transmits and receives network messages (packets) on behalf of user processes. This task includes routing of network packets to the target system.
    - Provision of a system call application programming interface (API): Processes can request the kernel to perform various tasks using kernel entry points known as system calls. 

### Process versus kernel views of the system.
- A process doesn't know where it is located in RAM or, in general, whether a particular part of its memory space is currently resident in memory or held in the swap area (a reserved area of disk space used to supplement the computer's RAM.)
- By contrast, a running system has one kernel that knows and controls everything. The kernel facilitates the running of all processes on the system. The kernel decides which process will next obtain access to the CPU, when it will do so, and for how long. The kernel maintains data structures containing information about all running processes and updates these structures as processes are created, change state, and terminate. The kernel maintains all of the low-level data structures that enable the filenames used by programs to be translated into physical locations on the disk. The kernel also maintains data structures that map the virtual memory of each process into the physical memory of the computer and the swap area(s) on disk. All communication between processes is done via mechanisms provided by the kernel. In response to requests from processes, the kernel creates new processes and terminates existing processes. Lastly, the kernel (in particular, device drivers) performs all direct communication with input and output devices, transferring information to and from user processes as required.

## 2.2 The Shell
- A shell is a special-purpose program designed to read commands typed by a user and execute appropriate programs in response to those commands. It's a user process.

## 2.4 Single Directory Hierarchy, Directories, Links, and Files
### File Types
- Within the file system, each file is marked with a type, indicating what kind of file it is.
    - One of these file types denotes ordinary data files, which are usually called regular or plain files to distinguish them from other file types.
    - These other file types include devices, pipes, sockets, directories, and symbolic links.

### Directories and Links
- A directory is a special file whose contents take the form of a table of filenames coupled with references to the corresponding files. This filename-plus-reference association is called a link.

## 2.5 File I/O Model
- One of the distinguishing features of the I/O model on UNIX systems is the concept of universality of I/O. This means that the same system calls (open(), read(), write(), close(), and so on) are used to perform I/O on all types of files, including devices. (The kernel translates the application’s I/O requests into appropriate filesystem or device-driver operations that perform I/O on the target file or device.)
- UNIX systems have no end-of-file character; the end of a file is detected by a read that returns no data.

## 2.6 Programs
- Program: Exists initially as source code, then compiled and linked into an executable file, and finally loaded into memory and run as a process.
- Script: A text file containing commands to be directly processed by a program such as a shell or other command interpreter.

## 2.7 Processes
- A process is an instance of an executing program. When a program is executed, the kernel loads the code of the program into virtual memory, allocates space for program variables, and sets up kernel bookkeeping data structures to record various information (such as process ID, termination status, user IDs, and group IDs) about the process.
- For resources that are limited, such as memory, the kernel initially allocates some amount of the resource to the process, and adjusts this allocation over the lifetime of the process in response to the demands of the process and the overall system demand for that resource. When the process terminates, all such resources are released for reuse by other processes.

### Process creation and program execution
- A process can create a new process using the fork() system call. The process that calls fork() is referred to as the parent process, and the new process is referred to as the child process. The kernel creates the child process by making a duplicate of the parent process. The child inherits copies of the parent’s data, stack, and heap segments, which it may then modify independently of the parent’s copies.
- The child process goes on either to execute a different set of functions in the same code as the parent, or, frequently, to use the execve() system call to load and execute an entirely new program. An execve() call destroys the existing text, data, stack, and heap segments, replacing them with new segments based on the code of the new program.

### The init process
- When booting the system, the kernel creates a special process called init, the “parent of all processes,” which is derived from the program file /sbin/init. All processes on the system are created (using fork()) either by init or by one of its descendants.
The init process always has the process ID 1 and runs with superuser privileges. The init process can’t be killed (not even by the superuser), and it terminates only when the system is shut down. The main task of init is to create and monitor a range of processes required by a running system.

### Environment List
- When a new process is created via fork(), it inherits a copy of its parent’s environment. Thus, the environment provides a mechanism for a parent process to communicate information to a child process. When a process replaces the program that it is running using exec(), the new program either inherits the environment used by the old program or receives a new environment specified as part of the exec() call.

``` bash
# This is why below works:
shad@linux:~/linux/notes$ export GREETING="hello world"
shad@linux:~/linux/notes$ python3 -c 'import os; print(os.getenv("GREETING"))'
hello world
```

## 2.8 Memory Mappings
- The memory in one process’s mapping may be shared with mappings in other processes. This can occur either because two processes map the same region of a file or because a child process created by fork() inherits a mapping from its parent.

## 2.10 Interprocess Communication and Synchronization
- signals, which are used to indicate that an event has occurred;
- pipes (familiar to shell users as the | operator) and FIFOs, which can be used to transfer data between processes;
- sockets, which can be used to transfer data from one process to another, either on the same host computer or on different hosts connected by a network;
- file locking, which allows a process to lock regions of a file in order to prevent other processes from reading or updating the file contents;
- message queues, which are used to exchange messages (packets of data) between processes;
- semaphores, which are used to synchronize the actions of processes; and
- shared memory, which allows two or more processes to share a piece of memory. When one process changes the contents of the shared memory, the changes are immediately visible to all other processes that have access to the same shared memory region.

## 2.11 Signals
- Signals are often described as “software interrupts.”  The arrival of a signal informs a process that some event or exceptional condition has occurred. Process can either:
    - Ignore the signal
    - Is killed upon receiving the signal
    - Suspended until later being resumed by receipt of a special-purpose signal.

## 2.12 Threads
- A thread is a lightweight set of processes that share the same virtual memory. Each thread is executing the same program code and shares the same data area and heap. However, each thread has its own stack containing local variables and function call linkage information.
- Threads can communicate with each other via the global variables that they share. The threading API provides condition variables and mutexes, which are primitives that enable the threads of a process to communicate and synchronize their actions, in particular, their use of shared variables.
