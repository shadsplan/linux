# Chapter 4. File I/O: The Universal I/O Model

## 4.1 Overview
- All system calls for performing I/O refer to open files using a file descriptor, a (usually small) nonnegative integers. File descriptors are used to refer to all types of open files, including pipes, FIFOs, sockets, terminals, devices, and regular files. Each process has its own set of file descriptors.
- By convention, most programs expect to be able to use the three standard file
descriptors:
    - 0 = standard input (stdin)
    - 1 = standard output (stdout)
    - 2 = standard error (stderr)

## 4.2 Universality of I/O
- This means that the same four system calls—open(), read(), write(), and close()—are used to perform I/O on all types of files, including devices such as terminals.
- Universality of I/O is achieved by ensuring that each file system and device driver implements the same set of I/O system calls.

## 4.9 Summary
- In order to perform I/O on a regular file, we must first obtain a file descriptor using open(). I/O is then performed using read() and write(). After performing all I/O, we should free the file descriptor and its associated resources using close(). These system calls can be used to perform I/O on all types of files.
- The fact that all file types and device drivers implement the same I/O interface allows for universality of I/O, meaning that a program can typically be used with any type of file without requiring code that is specific to the file type.
- For each open file, the kernel maintains a file offset, which determines the location at which the next read or write will occur. The file offset is implicitly updated by reads and writes. Using lseek(), we can explicitly reposition the file offset to any location within the file or past the end of the file.
