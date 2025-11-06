# Chapter 15. File Attributes

## 15.4 File Permissions

### 15.4.1 Permission on Regular Files

- The remaining 9 bits form the mask defining the permissions that are granted to various categories of users accessing the file. The file permissions mask divides the world into three categories:
  - Owner (also known as user): The permissions granted to the owner of the file.
  - Group: The permissions granted to users who are members of the file's group.
  - Other: The permissions granted to everyone else.
- Three permissions may be granted to each user category:
  - Read: The contents of the file may be read.
  - Write: The contents of the file may be changed.
  - Execute: The file may be executed (i.e., it is a program or a script). In order to execute a script file (e.g., a bash script), both read and execute permissions are required.

### 15.4.2 Permission on Directories
- The three permissions are interpreted differently:
    - Read: The contents (i.e., the list of filenames) of the directory may be listed (e.g., by ls).
    - Write: Files may be created in and removed from the directory. Note that it is not necessary to have any permission on a file itself in order to be able to delete it.
    - Execute: Files within the directory may be accessed. Execute permission on a directory is sometimes called search permission.
- When accessing a file, execute permission is required on all of the directories listed in the pathname. For example, reading the file /home/mtk/x would require execute permission on /, /home, and /home/mtk (as well as read permission on the file x itself). If the current working directory is /home/mtk/sub1 and we access the relative pathname ../sub2/x, then we need execute permission on /home/mtk and /home/mtk/sub2 (but not on / or /home).
- Read permission on a directory only lets us view the list of filenames in the directory. We must have execute permission on the directory in order to access the contents or the i-node information of files in the directory.
- Conversely, if we have execute permission on a directory, but not read permission, then we can access a file in the directory if we know its name, but we can't list the contents of (i.e., the other filenames in) the directory. This is a simple and frequently used technique to control access to the contents of a public directory.
- To add or remove files in a directory, we need both execute and write permissions on the directory.

## 15.6 Summary

- The stat() system call retrieves information about a file (metadata), most of which is drawn from the file i-node. This information includes file ownership, file permissions, and file timestamps.
- Each file has an associated user ID (owner) and group ID, as well as a set of permission bits. For permissions purposes, file users are divided into three categories:
owner (also known as user), group, and other. Three permissions may be granted to each category of user: read, write, and execute. The same scheme is used with directories, although the permission bits have slightly different meanings. The chown() and chmod() system calls change the ownership and permissions of a file.
- Each file has an associated user ID (owner) and group ID, as well as a set of permission bits. For permissions purposes, file users are divided into three categories: owner (also known as user), group, and other. Three permissions may be granted to each category of user: read, write, and execute. The same scheme is used with directories, although the permission bits have slightly different meanings. The chown() and chmod() system calls change the ownership and permissions of a file.
- Three additional permission bits are used for files and directories. The setuser-ID and set-group-ID permission bits can be applied to program files to create programs that cause the executing process to gain privilege by assuming a different effective user or group identity (that of the program file).
