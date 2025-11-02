# Chapter 5. File I/O: Further Details

## 5.4 Relationship between File Descriptors and Open Files

- There is not a one-to-one correspondence between file descriptors and open files. It is possible and useful to have multiple descriptors referring to the same open file. These file descriptors may be open in the same process or in different processes.

3 data structures mantained by the kernel to keep track of open files:
1. For each process, the kernel maintains a table of open file descriptors. Each entry in this table records information about a single file descriptor, including:
    - a set of flags controlling the operation of the file descriptor
    - a reference to the open file description.
2. The kernel maintains a system-wide table of all open file descriptions. (This table is sometimes referred to as the open file table, and its entries are sometimes called open file handles.) An open file description stores all information relating to an open file, including:
    - the current file offset (as updated by read() and write(), or explicitly modified using lseek());
    - status flags specified when opening the file (i.e., the flags argument to open());
    - the file access mode (read-only, write-only, or read-write, as specified in open());
    - settings relating to signal-driven I/O (Section 63.3); and
    - a reference to the i-node object for this file.
3. Each file system has a table of i-nodes for all files residing in the file system. An i-node for each file file includes the following information:
    - file type (e.g., regular file, socket, or FIFO) and permissions;
    - a pointer to a list of locks held on this file; and
    - various properties of the file, including its size and timestamps relating to different types of file operations.

Page 95 shows a relationship between file descriptors, open file descriptions, and i-nodes. Takeaways:
- Two different file descriptors that refer to the same open file description share a file offset value. Therefore, if the file offset is changed via one file descriptor (as a consequence of calls to read(), write(), or lseek()), this change is visible through the other file descriptor. This applies both when the two file descriptors belong to the same process and when they belong to different processes.
- Similar scope rules apply when retrieving and changing the open file status flags (e.g., O_APPEND, O_NONBLOCK, and O_ASYNC) using the fcntl() F_GETFL and F_SETFL operations.
- By contrast, the file descriptor flags (i.e., the close-on-exec flag) are private to the process and file descriptor. Modifying these flags does not affect other file descriptors in the same process or a different process.