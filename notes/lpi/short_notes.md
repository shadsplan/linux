# Chapter 7. Memory Allocation
- Using the malloc family of functions, a process can dynamically allocate and release memory on the heap.
- The alloca() function allocates memory on the stack. This memory is automatically deallocated when the function that calls alloca() returns.

# Chapter 8. Users and Groups
- Each user has a unique login name and an associated numeric user ID. Users can belong to one or more groups, each of which also has a unique name and an associated numeric identifier. The primary purpose of these identifiers is to establish ownership of various system resources (e.g., files) and permissions for accessing them.
- A user’s name and ID are defined in the /etc/passwd file, which also contains other information about the user. A user’s group memberships are defined by fields in the /etc/passwd and /etc/group files. A further file, /etc/shadow, which can be read only by privileged processes, is used to separate the sensitive password information from the publicly available user information in /etc/passwd.

