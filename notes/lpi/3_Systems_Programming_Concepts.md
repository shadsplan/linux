# Chapter 3. Systems Programming Concepts

## 3.1 System Calls
A system call is a controlled entry point into the kernel, allowing a process to request that the kernel perform some action on the process’s behalf. The kernel makes a range of services accessible to programs via the system call application programming interface (API). These services include, for example, creating a new process, performing I/O, and creating a pipe for interprocess communication.
- A system call changes the processor state from user mode to kernel mode, so that the CPU can access protected kernel memory.
- The set of system calls is fixed. Each system call is identified by a unique number. (This numbering scheme is not normally visible to programs, which identify system calls by name.)
- Each system call may have a set of arguments that specify information to be transferred from user space (i.e., the process’s virtual address space) to kernel space and vice versa.

Almost every system call and library function returns some type of status value indicating whether the call succeeded or failed. This status value should always be
checked to see whether the call succeeded. 

## Summary
- System calls allow processes to request services from the kernel. Even the simplest system calls have a significant overhead by comparison with a user-space function call, since the system must temporarily switch to kernel mode to execute the system call, and the kernel must verify system call arguments and transfer data between user memory and kernel memory.
- The standard C library provides a multitude of library functions that perform a wide range of tasks. Some library functions employ system calls to do their work; others perform tasks entirely within user space.
- Most system calls and library functions return a status indicating whether a call has succeeded or failed. Such status returns should always be checked.