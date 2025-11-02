# Chapter 18. Directories and Links

## 18.1 Directories and (Hard) Links
- A directory is stored in the file system in a similar way to a regular file. Two things distinguish a directory from a regular file:
  - A directory is marked with a different file type in its i-node entry.
  - A directory is a file with a special organization. Essentially, it is a table consisting of filenames and i-node numbers.
- The relationship between directories and i-nodes is illustrated in Figure 18-1, which shows the partial contents of the file system i-node table and relevant directory files that are maintained for an example file (/etc/passwd).
- If we review the list of information stored in a file i-node (Section 14.4), we see that the i-node doesn’t contain a filename; it is only the mapping within a directory list that defines the name of a file. This has a useful consequence: we can create multiple names—in the same or in different directories—each of which refers to the same i-node. These multiple names are known as links, or sometimes as hard links to distinguish them from symbolic links, which we discuss shortly.
- From the shell, we can create new hard links to an existing file using the ln command, as shown in the following shell session log:

``` bash
$ echo -n 'It is good to collect things,' > abc
$ ls -li abc
 122232 -rw-r--r-- 1 mtk users 29 Jun 15 17:07 abc
$ ln abc xyz
$ echo ' but it is better to go on walks.' >> xyz
$ cat abc
It is good to collect things, but it is better to go on walks.

# abc and xyz refer to the same i-node, the link count of the i-node referred to by both files has risen to 2.
$ ls -li abc xyz
 122232 -rw-r--r-- 2 mtk users 63 Jun 15 17:07 abc
 122232 -rw-r--r-- 2 mtk users 63 Jun 15 17:07 xyz
 
$ rm abc
$ ls -li xyz
 122232 -rw-r--r-- 1 mtk users 63 Jun 15 17:07 xyz
 ```

- The i-node entry and data blocks for the file are removed (deallocated) only when the i-node’s link count falls to 0—that is, when all of the names for the file have been removed. To summarize: the rm command removes a filename from a directory list, decrements the link count of the corresponding i-node by 1, and, if the link count thereby falls to 0, deallocates the i-node and the data blocks to which it refers.
- A question often asked in online forums is “How can I find the filename associated with the file descriptor X in my program?” The short answer is that we can’t—at least not portably and unambiguously—since a file descriptor refers to an i-node, and multiple filenames (or even, as described in Section 18.3, none at all) may refer to this i-node.

Hard links have two limitations, both of which can be circumvented by the use of symbolic links:
- Because directory entries (hard links) refer to files using just an i-node number, and i-node numbers are unique only within a file system, a hard link must reside on the same file system as the file to which it refers.
- A hard link can’t be made to a directory. This prevents the creation of circular links, which would confuse many system programs.

## 18.2 Symbolic (Soft) Links
- A symbolic link, also sometimes called a soft link, is a special file type whose data is the name of another file.
- From the shell, symbolic links are created using the ln –s command. The ls –F command displays a trailing @ character at the end of symbolic links.
- The pathname to which a symbolic link refers may be either absolute or relative. A relative symbolic link is interpreted relative to the location of the link itself.
- Symbolic links don’t have the same status as hard links. In particular, a symbolic link is not included in the link count of the file to which it refers. Therefore, if the filename to which the symbolic link refers is removed, the symbolic link itself continues to exist, even though it can no longer be dereferenced (followed). We say that it has become a dangling link. It is even possible to create a symbolic link to a filename that doesn’t exist at the time the link is created.
- Since a symbolic link refers to a filename, rather than an i-node number, it can be used to link to a file in a different file system. Symbolic links also do not suffer the other limitation of hard links: we can create symbolic links to directories. Tools such as find and tar can tell the difference between hard and symbolic links, and either don’t follow symbolic links by default, or avoid getting trapped in circular references created using symbolic links.
- Starting with kernel 2.6.18, Linux implements the SUSv3-specified minimum of 8 dereferences. Linux also imposes a total of 40 dereferences for an entire pathname. These limits are required to prevent extremely long symbolic link chains, as well as symbolic link loops, from causing stack overflows in the kernel code that resolves symbolic links.

### Interpretation of symbolic links by system calls
- Many system calls dereference (follow) symbolic links and thus work on the file to which the link refers. Some system calls don’t dereference symbolic links, but instead operate directly on the link file itself.
- One point generally applies: symbolic links in the directory part of a pathname (i.e., all of the components preceding the final slash) are always dereferenced. Thus, in the pathname /somedir/somesubdir/file, somedir and somesubdir will always be dereferenced if they are symbolic links, and file may be dereferenced, depending on the system call to which the pathname is passed.

### File permissions and ownerships for symbolic links
- The ownership and permissions of a symbolic link are ignored for most operations (symbolic links are always created with all permissions enabled). Instead, the ownership and permissions of the file to which the link refers are used in determining whether an operation is permitted.

## 18.15 Summary
- An i-node doesn’t contain a file’s name. Instead, files are assigned names via entries in directories, which are tables listing filename and i-node number correspondences. These directory entries are called (hard) links. A file may have multiple hard links, all of which enjoy equal status. Links are created and removed using link() and unlink(). A file can be renamed using the rename() system call.
- A symbolic (or soft) link is created using symlink(). Symbolic links are similar to hard links in some respects, with the differences that symbolic links can cross filesystem boundaries and can refer to directories. A symbolic link is just a file containing the name of another file; this name may be retrieved using readlink(). A symbolic link is not included in the (target) i-node’s link count, and it may be left dangling if the filename to which it refers is removed. Some system calls automatically dereference (follow) symbolic links; others do not. In some cases, two versions of a system call are provided: one that dereferences symbolic links and another that does not. Examples are stat() and lstat().
- Each process has a root directory, which determines the point from which absolute pathnames are interpreted, and a current working directory, which determines the point from which relative pathnames are interpreted. The chroot() and chdir() system calls are used to change these attributes. The getcwd() function returns a process’s current working directory.