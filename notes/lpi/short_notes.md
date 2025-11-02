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

