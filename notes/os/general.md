# General OS Notes

## [How does Linux Boot Process Work?](https://www.youtube.com/watch?v=XpFsMB6FoOs)

1. When you press the power button, a program called the BIOS/UEFI boots up. It is pieces of software that gets all the main parts of your computer ready for action (keyboard, screen, hard drive, etc).
    1. UEFI is the newer version, offering faster boot times and better security features (secure boot).
    2. BIOS is tied to Master Boot Record (MBR) partitioning scheme, which limits disk size to 2TB and supports up to 4 primary partitions.
    3. UEFI is tied to GUID Partition Table (GPT) scheme, which supports disks larger than 2TB and up to 128 partitions.
2. The BIOS/UEFI runs a check called a POST (Power-On Self Test) to make sure all the hardware is working properly before fully turning on.
   1. If POST finds a problem, it will show an error message on the screen.
   2. If everything checks out with POST, the BIOS/UEFI needs to find and load up the bootloader software.
3. The bootloader is usually set to check the hard drive first, then USB drives, then CD/DVD drives for bootable software.
    1. On a BIOS system, the boot loader code lives in the first little chunk of the hard drive called the Master Boot Record (MBR). For UEFI, there is a seperate partition on the hard drive that stores files like the .efi boot loader file.
    2. The key jobs for the boot loader are:
        1. Locate the operating system kernel on the disk.
        2. Load the kernel into the computer's memory.
        3. Start running the kernel code.
    3. Common boot loaders are GRUB2 (GRand Unified Bootloader) and LILO (LInux LOader). LiLo is outdated and rarely used in modern distributions. GRUB2 is the most full-featured and widely used today.
    4. Once GRUB2 is loaded, it inserts the Linux kernel into memory and hands control over to the kernel to finish the startup process.
4. After the bootloader starts the kernel, the kernel takes over the computer's resources and starts initiating all the background processes and services.
    1. It decompresses itself into memory, checks the hardware, and loads device drivers and other kernel modules.
    2. An initial process called "init" kicks off, which in modern Linux systems is typically systemd.
5. systemd is the parent of all processes in linux. It has a ton of responsibilities to get the system booted and ready to use:
    1. It's checking for any remaining hardware that needs drivers loaded.
    2. It mounts up all your different file systems and disks so that they're accessible.
    3. It starts launching all background services you need like networking, sound, and power management.
    4. It handles user logins once you get to the graphical prompt. And it loads up your desktop environment with the panels and menus.
    5. In short: systemd handles initializing everything that needs to launch behind the scenes when starting up Linux.


## [Linux File System/Structure Basics](https://www.youtube.com/watch?v=HbgzrKJvDRw)

- `/bin`: binaries
    - Stores basic binaries (programs/applications).
    - ls, cat, other basic functions.
- `/sbin`: system binaries
    - Standard user wouldn't have permission to run these.
- `/boot`: boot files
    - Contains everything your OS needs to boot. Your bootloader lives here.
``` bash
shad@linux:~/linux/notes/os$ ls /boot
System.map-6.14.0-1012-azure  config-6.14.0-1014-azure  initrd.img                    initrd.img.old  vmlinuz-6.14.0-1012-azure
System.map-6.14.0-1014-azure  efi                       initrd.img-6.14.0-1012-azure  lost+found      vmlinuz-6.14.0-1014-azure
config-6.14.0-1012-azure      grub                      initrd.img-6.14.0-1014-azure  vmlinuz         vmlinuz.old
```

- `/dev`: device files
    - Where your devices live.
    - Linux, following UNIX, has a standard where it was decided everything is a file.
    - A disk for example would be `/dev/sda`. A partition on that disk would be `/dev/sda1`.
    - Can find all device files here, from webcam to keyboard.

``` bash
shad@linux:~/linux/notes/os$ ls /dev/
autofs           fuse          loop6         pts       stderr  tty19  tty34  tty5   tty8    ttyS22  ttyprintk    vcsa5
block            hpet          loop7         random    stdin   tty2   tty35  tty50  tty9    ttyS23  udmabuf      vcsa6
bsg              hugepages     mapper        rfkill    stdout  tty20  tty36  tty51  ttyS0   ttyS24  uinput       vcsu
btrfs-control    hwrng         mcelog        root      tpm0    tty21  tty37  tty52  ttyS1   ttyS25  urandom      vcsu1
char             infiniband    mem           rtc       tpmrm0  tty22  tty38  tty53  ttyS10  ttyS26  userfaultfd  vcsu2
console          initctl       mqueue        rtc0      tty     tty23  tty39  tty54  ttyS11  ttyS27  vcs          vcsu3
core             input         net           sda       tty0    tty24  tty4   tty55  ttyS12  ttyS28  vcs1         vcsu4
cpu              kmsg          null          sda1      tty1    tty25  tty40  tty56  ttyS13  ttyS29  vcs2         vcsu5
cpu_dma_latency  kvm           nvme-fabrics  sda14     tty10   tty26  tty41  tty57  ttyS14  ttyS3   vcs3         vcsu6
cuse             log           nvram         sda15     tty11   tty27  tty42  tty58  ttyS15  ttyS30  vcs4         vfio
disk             loop-control  port          sda16     tty12   tty28  tty43  tty59  ttyS16  ttyS31  vcs5         vga_arbiter
dma_heap         loop0         ppp           sdb       tty13   tty29  tty44  tty6   ttyS17  ttyS4   vcs6         vhost-net
dri              loop1         psaux         sdb1      tty14   tty3   tty45  tty60  ttyS18  ttyS5   vcsa         vhost-vsock
ecryptfs         loop2         ptmx          sg0       tty15   tty30  tty46  tty61  ttyS19  ttyS6   vcsa1        vmbus
fb0              loop3         ptp0          sg1       tty16   tty31  tty47  tty62  ttyS2   ttyS7   vcsa2        zero
fd               loop4         ptp1          shm       tty17   tty32  tty48  tty63  ttyS20  ttyS8   vcsa3        zfs
full             loop5         ptp_hyperv    snapshot  tty18   tty33  tty49  tty7   ttyS21  ttyS9   vcsa4
```

- `/etc`: configuration files
    - Specifically for system-wide configuration files (ex. apt)
    - Not user-specific config files (ex. Libre Office would have settings in each user's folder and it wouldn't be system-wide because each user could have different settings).
- `/lib`, `/lib64`: libraries
    - libraries are files that applications can use to perform various functions.
    - required by the binaries in `/bin` and `/sbin`.
    - ex: gcc, python3
- `/mnt`: where you would find other mounted drives.
    - floppy disk, usb stick, external hard drive, network drive, or a second hard drive.
- `/media`: most distros automatically mount devices for you in the media directory so your USB stick that you inserted would be in `/media/username/devicename`.
   - Why are there two directories? If you're mounting things manually, use the `/mnt` directory and leave the `/media` directory for the OS to manage.
- `/opt`: optional software
    - Manually installed software from vendors reside.
    - Although some software packages found in the repo can also find their way here (ex. Virtual Box Guest Extensions).
    - Install software you've created yourself.
- `/proc`: pseudo-files that contain information about system processes and resources.
    - Every process will have a directory here named after its PID, containing all kinds of information about that process.
    - Like `/dev` where they're not actually files on disk, this is the kernel translating other information to appear as files. 

``` bash
shad@linux:~/linux/notes/os$ ls /proc/
1     1492  2     3    41   458  528   64    748   8313  992         driver         kmsg           pagetypeinfo   timer_list
10    15    20    30   42   46   53    65    749   8403  999         dynamic_debug  kpagecgroup    partitions     tty
100   1521  202   31   43   460  54    66    75    8414  acpi        execdomains    kpagecount     pressure       uptime
1004  1532  206   32   435  461  55    67    7774  8433  bootconfig  fb             kpageflags     schedstat      version
1006  1546  21    34   437  462  56    68    79    85    buddyinfo   filesystems    latency_stats  scsi           version_signature
1189  1577  217   36   439  463  57    69    7996  883   bus         fs             loadavg        self           vmallocinfo
13    16    22    366  443  47   58    693   8     914   cgroups     interrupts     locks          slabinfo       vmstat
1379  1613  23    37   444  474  59    7     801   926   cmdline     iomem          mdstat         softirqs       zoneinfo
1383  162   24    38   445  48   6     7050  806   927   consoles    ioports        meminfo        stat
14    163   25    383  446  49   60    7077  809   928   cpuinfo     irq            misc           swaps
141   17    26    39   447  5    61    73    816   937   crypto      kallsyms       modules        sys
1418  18    28    4    448  50   62    7313  829   954   devices     kcore          mounts         sysrq-trigger
1473  19    2873  40   45   51   6270  7383  83    972   diskstats   key-users      mtrr           sysvipc
1474  1925  29    401  457  52   63    74    830   99    dma         keys           net            thread-self

shad@linux:~/linux/notes/os$ cat /proc/cpuinfo | head
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 85
model name      : Intel(R) Xeon(R) Platinum 8272CL CPU @ 2.60GHz
stepping        : 7
microcode       : 0xffffffff
cpu MHz         : 2992.372
cache size      : 36608 KB
physical id     : 0

shad@linux:~/linux/notes/os$ cat /proc/uptime
19711.21 38123.96
```

- `/root`: root user's home directory. Need root permissions to access.
- `/run`: Used for processes that start early in the boot procedure to store runtime information that they use to function.
    - fairly new. It's a tempfs filesystem (runs in RAM), and gone if the system reboots.
- `/snap`: Where snap packages are stored.
    - Mainly used by Ubuntu.
    - Snap packages are self-contained applications that run differently than regular packages and applications.
- `/srv`: service directory where service data is stored
    - if you run a web server, you would store the files that will be accessed by external users here.
    - allows for better security since it's at the root of the drive, and it also allows you to easily moun thtis folder from another hard drive.
- `/sys`: It's a way to interact with the kernel.
    - Similar to `/proc` and `/dev` where the files here aren't actually stored on disk, but are representations of kernel data structures.
    - You can read and write to these files to get information about the system and change kernel parameters on the fly.

``` bash
shad@linux:~/linux/notes/os$ ls /sys/
block  bus  class  dev  devices  firmware  fs  kernel  module  power
```

- `/tmp`: temporary directory
    - where files are temporarily stored by applications that could be used during a session.
    - Ex. Writing a document in Libre Office, it will regularly save a temporary copy of what you're writing here, so that if the application crashes, it can look here to see if there's a recent saved copy that you can recover.
    - Usually emptied on reboot.
    - Occasionally, you might find some files or directory that remain and could be stuck there because the system can't delete them, normally not a big deal unless there's hundreds of files or the files are taking lots of disk space. In that case, you might want to log into root and delete them manually.
- `/usr`: user application space where applications will be installed that are used by the user.
    - as opposed by the `/bin` used by the system / system administrator to perform maintenance.
    - any applications installed here are considered non-essential for basic system operation.
- `/var`: variable directory.
    - It contains files and directories that are expected to grow in size.

``` bash
shad@linux:~/linux/notes/os$ ls /var/log/
README              auth.log       btmp                   dist-upgrade  dmesg.4.gz  kern.log.2.gz  syslog       unattended-upgrades
alternatives.log    auth.log.1     btmp.1                 dmesg         dpkg.log    kern.log.3.gz  syslog.1     waagent.log
alternatives.log.1  auth.log.2.gz  chrony                 dmesg.0       dpkg.log.1  kern.log.4.gz  syslog.2.gz  wtmp
apache2             auth.log.3.gz  cloud-init-output.log  dmesg.1.gz    journal     landscape      syslog.3.gz
apport.log          auth.log.4.gz  cloud-init.log.1       dmesg.2.gz    kern.log    lastlog        syslog.4.gz
apt                 azure          cloud-init.log.2.gz    dmesg.3.gz    kern.log.1  private        sysstat
```

- `/home`: Where you store your personal files and documents. 
    - Each user has their own directory here.
    - Each user can only access their own home directory, unless they use admin permissions.
    - Some users mount the `/home` folder on a different drive or partition.
    - Also contains many different directories which store your application settings.
    - A hidden directory starts with a `.`, Linux hides these by default. `ls -a` to see them. 
        - These hidden directories store things like `.cache` that some applications like a browser used to store temporary files. Other applications might store thumbnails or information that will be used over and over again.
        - `.config` and `.local`, which store individual application settings.

## Notes about Processes

### [Program](https://en.wikipedia.org/wiki/Computer_program)
- A computer program is a sequence or set of instructions in a programming language for a computer to execute.
- A computer program in its human-readable form is called source code. Source code needs another computer program to execute because computers can only execute their native machine instructions. Therefore, source code may be translated to machine instructions using a compiler written for the language. The resulting file is called an executable. The central processing unit will soon switch to this process so it can fetch, decode, and then execute each machine instruction.
- If the executable is requested for execution, then the operating system loads it into memory and starts a process.
- If the source code is requested for execution, then the operating system loads the corresponding interpreter into memory and starts a process. The interpreter then loads the source code into memory to translate and execute each statement. Running the source code is slower than running an executable. Moreover, the interpreter must be installed on the computer.

### [Process](https://en.wikipedia.org/wiki/Process_(computing))
- In computing, a process is the instance of a computer program that is being executed by one or many threads.
- While a computer program is a passive collection of instructions typically stored in a file on disk, a process is the execution of those instructions after being loaded from the disk into memory.
- Multitasking is a method to allow multiple processes to share processors (CPUs) and other system resources. Each CPU (core) executes a single process at a time. However, multitasking allows each processor to switch between tasks that are being executed without having to wait for each task to finish (preemption).
- A multitasking operating system may just switch between processes to give the appearance of many processes executing simultaneously (that is, in parallel), though in fact only one process can be executing at any one time on a single CPU (unless the CPU has multiple cores, then multithreading or other similar technologies can be used).
- The OS holds the below information about active processes in data structures called process control blocks: 
    - An image of the executable machine code associated with a program.
    - Memory (typically some region of virtual memory); which includes the executable code, process-specific data (input and output), a call stack (to keep track of active subroutines and/or other events), and a heap to hold intermediate computation data generated during run time.
    - Operating system descriptors of resources that are allocated to the process, such as file descriptors (Unix terminology) or handles (Windows), and data sources and sinks.
    - Security attributes, such as the process owner and the process' set of permissions (allowable operations).
    - Processor state (context), such as the content of registers and physical memory addressing. The state is typically stored in computer registers when the process is executing, and in memory otherwise.
- Process States: 
    - First, the process is "created" by being loaded from a secondary storage device (hard disk drive, CD-ROM, etc.) into main memory. After that the process scheduler assigns it the "waiting" state.
    - While the process is "waiting", it waits for the scheduler to do a so-called context switch. The context switch loads the process into the processor and changes the state to "running" while the previously "running" process is stored in a "waiting" state.
    - If a process in the "running" state needs to wait for a resource (wait for user input or file to open, for example), it is assigned the "blocked" state. The process state is changed back to "waiting" when the process no longer needs to wait (in a blocked state).
    - Once the process finishes execution, or is terminated by the operating system, it is no longer needed. The process is removed instantly or is moved to the "terminated" state. When removed, it just waits to be removed from main memory.

### [Thread](https://en.wikipedia.org/wiki/Thread_(computing))
- In computer science, a thread of execution is the smallest sequence of programmed instructions that can be managed independently by a scheduler, which is typically a part of the operating system.
- They differ from processes in the following ways:
    - processes are typically independent, while threads exist as subsets of a process
    - processes carry considerably more state information than threads, whereas multiple threads within a process share process state as well as memory and other resources
    - processes have separate address spaces, whereas threads share their address space
    - processes interact only through system-provided inter-process communication mechanisms
    - context switching between threads in the same process typically occurs faster than context switching between processes
- Process Control Blocks (PCBs) and Thread Control Blocks (TCBs) are kernel-resident data structures that store all the metadata the OS needs to manage processes and threads — including register values, scheduling state, memory mappings, and resource handles. When the scheduler preempts a running thread on a CPU, it saves that thread’s context — its registers, stack pointer, and instruction pointer — into its TCB (and related PCB) and then selects another runnable thread. The scheduler restores the new thread’s saved context by loading its registers, including its last instruction pointer, so it resumes execution from exactly where it was paused. The previously preempted process’s memory — its code, heap, stack, and data — remains intact in RAM; only its CPU context is swapped out until it’s scheduled again.

### [`fork()`](https://en.wikipedia.org/wiki/Exec_(system_call))
- In computing, particularly in the context of the Unix operating system and its workalikes, fork is an operation whereby a process creates a copy of itself.
- In multitasking operating systems, processes (running programs) need a way to create new processes, e.g. to run other programs. Fork and its variants are typically the only way of doing so in Unix-like systems. For a process to start the execution of a different program, it 
    - first forks to create a copy of itself.
    - Then, the copy, called the "child process", makes any environment changes the child will need and then calls the exec system call to overlay itself with the new program: it ceases execution of its former program in favor of the new.
- The fork operation creates a separate address space for the child. The child process has an exact copy of all the memory segments of the parent process. In modern UNIX variants that follow the virtual memory model from SunOS-4.0, copy-on-write semantics are implemented and the physical memory need not be actually copied. Instead, virtual memory pages in both processes may refer to the same pages of physical memory until one of them writes to such a page: then it is copied. This optimization is important in the common case where fork is used in conjunction with exec to execute a new program: typically, the child process performs only a small set of actions before it ceases execution of its program in favour of the program to be started, and it requires very few, if any, of its parent's data structures.

### [`exec()`](https://en.wikipedia.org/wiki/Exec_(system_call))
- In computing, exec is a functionality of an operating system that runs an executable file in the context of an already existing process, replacing the previous executable. As no new process is created, the process identifier (PID) does not change, but the machine code, data, heap, and stack of the process are replaced by those of the new program.

### [Orphan Process](https://en.wikipedia.org/wiki/Orphan_process)
- An orphan process is a computer process whose parent process has finished or terminated, though it remains running itself.
- In a Unix-like operating system any orphaned process will be immediately adopted by an implementation-defined system process: the kernel sets the parent to this process. This operation is called re-parenting and occurs automatically. Even though technically the process has a system process as its parent, it is still called an orphan process since the process that originally created it no longer exists.
- Most Unix systems have historically used init as the system process to which orphans are reparented.
- A process can be orphaned unintentionally, such as when the parent process terminates or crashes. The process group mechanism in most Unix-like operating systems can be used to help protect against accidental orphaning, where in coordination with the user's shell will try to terminate all the child processes with the "hangup" signal (SIGHUP), rather than letting them continue to run as orphans.
- It is sometimes desirable to intentionally orphan a process, usually to allow a long-running job to complete without further user attention, or to start an indefinitely running service or agent; such processes (without an associated session) are known as daemons, particularly if they are indefinitely running. A low-level approach is to fork twice, running the desired process in the grandchild, and immediately terminating the child. The grandchild process is now orphaned, and is not adopted by its grandparent, but rather by init.

### [Zombie Process](https://en.wikipedia.org/wiki/Zombie_process)
- On Unix and Unix-like computer operating systems, a zombie process or defunct process is a process that has completed execution (via the exit system call) but still has an entry in the process table: it is a process in the "terminated state". This occurs for the child processes, where the entry is still needed to allow the parent process to read its child's exit status: once the exit status is read via the wait system call, the defunct process' entry is removed from the process table and it is said to be "reaped". A child process initially becomes defunct, only then being removed from the resource table. Under normal system operation, defunct processes are immediately waited on by their parent and then reaped by the system.
- On Unix and Unix-like computer operating systems, a zombie process or defunct process is a process that has completed execution (via the exit system call) but still has an entry in the process table: it is a process in the "terminated state". This occurs for the child processes, where the entry is still needed to allow the parent process to read its child's exit status: once the exit status is read via the wait system call, the defunct process' entry is removed from the process table and it is said to be "reaped". A child process initially becomes defunct, only then being removed from the resource table. Under normal system operation, defunct processes are immediately waited on by their parent and then reaped by the system.
- Processes that stay defunct for a long time are usually an error and can cause a resource leak. Generally, the only kernel resource they occupy is the process table entry, their process ID. However, defuncts can also hold buffers open, consuming memory. Defunct processes can hold handles to file descriptors, which prevents the space for those files from being available to the filesystem.
- After the zombie is removed, its process identifier (PID) and entry in the process table can then be reused. However, if a parent fails to call wait, the zombie will be left in the process table, causing a resource leak.
- Zombies can be identified in the output from the Unix ps command by the presence of a "Z" in the "STAT" column.[2] Zombies that exist for more than a short period of time typically indicate a bug in the parent program, or just an uncommon decision to not reap children (see example). 
- As with other resource leaks, the presence of a few zombies is not worrisome in itself, but may indicate a problem that would grow serious under heavier loads. Since there is no memory allocated to zombie processes – the only system memory usage is for the process table entry itself – the primary concern with many zombies is not running out of memory, but rather running out of process table entries, concretely process ID numbers. However, zombies can hold open buffers that are associated with file descriptors, and thereby cause memory to be consumed by the zombie. Zombies can also hold a file descriptor to a file that has been deleted. This prevents the file system from recovering the i-nodes for the deleted file.
- When a process loses its parent, init becomes its new parent. init periodically executes the wait system call to reap any zombies with init as parent.

### [Daemon Process](https://en.wikipedia.org/wiki/Daemon_(computing))
- In computing, a daemon is a program that runs as a background process, rather than being under the direct control of an interactive user.
- In a Unix-like system, the parent process of a daemon is often, but not always, the init process. A daemon is usually created either by the init process directly launching the daemon, by the daemon being run by an initialization script run by init, by the daemon being launched by a super-server launched by init.
- The init process in Research Unix and BSD starts daemons from an initialization script. A daemon started as a command in an initialization script must either fork a child process and then immediately exit, or must be run as a background process using &, so that the shell running the initialization script can continue after starting the daemon. In the former case, the daemon process run from the shell exits, thus causing init to adopt the child process that runs as the daemon; in the latter case, when the shell running the initialization script exits, the child daemon process is adopted by init.
- Operations such a daemon must do include:
    - Optionally removing unnecessary variables from environment.
    - Executing as a background task by forking and exiting (in the parent "half" of the fork). This allows daemon's parent (shell or startup process) to receive exit notification and continue its normal execution.
    - Detaching from the invoking session, usually accomplished by a single operation, setsid():
    - Dissociating from the controlling tty.
    - Creating a new session and becoming the session leader of that session.
    - Becoming a process group leader.
    - If the daemon wants to ensure that it will not acquire a new controlling tty even by accident (which happens when a session leader without a controlling tty opens a free tty), it may fork and exit again. This means that it is no longer a session leader in the new session, and cannot acquire a controlling tty.
    - Setting the root directory (/) as the current working directory so that the process does not keep any directory in use that may be on a mounted file system (allowing it to be unmounted).
    - Changing the umask to 0 to allow open(), creat(), and other operating system calls to provide their own permission masks and not to depend on the umask of the caller.
    - Redirecting file descriptors 0, 1 and 2 for the standard streams (stdin, stdout and stderr) to /dev/null or a logfile, and closing all the other file descriptors inherited from the parent process.

## Files

### [File descriptors](https://en.wikipedia.org/wiki/File_descriptor)
- In Unix and Unix-like computer operating systems, a file descriptor (FD, less frequently fildes) is a process-unique identifier (handle) for a file or other input/output resource, such as a pipe or network socket.
- Each unix process (except perhaps daemons) should have three standard file descriptors open:
    - Standard input (stdin), with file descriptor 0, is used for input, by default from the keyboard.
    - Standard output (stdout), with file descriptor 1, is used for output, by default to the terminal window.
    - Standard error (stderr), with file descriptor 2, is used for output of error messages and diagnostics, by default to the terminal window.
- In the traditional implementation of Unix, file descriptors index into a per-process file descriptor table maintained by the kernel, that in turn indexes into a system-wide table of files opened by all processes, called the file table. This table records the mode with which the file (or other resource) has been opened: for reading, writing, appending, and possibly other modes. It also indexes into a third table called the inode table that describes the actual underlying files. To perform input or output, the process passes the file descriptor to the kernel through a system call, and the kernel will access the file on behalf of the process. The process does not have direct access to the file or inode tables.
- In Unix-like systems, file descriptors can refer to any Unix file type named in a file system.
- The seven standard Unix file types are regular, directory, symbolic link, FIFO special, block special, character special, and socket as defined by POSIX.

### [inode](https://en.wikipedia.org/wiki/Inode)
- An inode (index node) is a data structure in a Unix-style file system that describes a file-system object such as a file or a directory. Each inode stores the attributes and disk block locations of the object's data. File-system object attributes may include metadata (times of last change, access, modification), as well as owner and permission data.
- A directory is a list of inodes with their assigned names. The list includes an entry for itself, its parent, and each of its children.
- A file system relies on data structures about the files, as opposed to the contents of that file. The former are called metadata—data that describes data. Each file is associated with an inode, which is identified by an integer, often referred to as an i-number or inode number.
- Inodes store information about files and directories (folders), such as file ownership, access mode (read, write, execute permissions), and file type. The data may be called stat data, in reference to the stat system call that provides the data to programs.
- The inode number indexes a table of inodes on the file system. From the inode number, the kernel's file system driver can access the inode contents, including the location of the file, thereby allowing access to the file.
- File names and directory implications:
    - Inodes do not contain their hard link names, only other file metadata.
    - Unix directories are lists of association structures, each of which contains one filename and one inode number.
    - The file system driver must search a directory for a particular filename and then convert the filename to the correct corresponding inode number.

[Inodes and the Linux File System](https://www.redhat.com/en/blog/inodes-linux-filesystem)
- By definition, an inode is an index node. It serves as a unique identifier for a specific piece of metadata on a given filesystem. Each piece of metadata describes what we think of as a file.
- In short, each filesystem mounted to your computer has its own inodes. An inode number may be used more than once but never by the same filesystem. The filesystem id combines with the inode number to create a unique identification label.
- `df -i /dev/sda1` shows inode usage on that partition.

``` bash
shad@linux:~/linux/notes/os$ df -hi
Filesystem     Inodes IUsed IFree IUse% Mounted on
/dev/root        3.7M  116K  3.6M    4% /
```

### [Hard links and soft links in Linux explained](https://www.redhat.com/en/blog/linking-linux-explained)
#### Hard Links
- The concept of a hard link is the most basic we will discuss today. Every file on the Linux filesystem starts with a single hard link. The link is between the filename and the actual data stored on the filesystem.

``` bash
[tcarrigan@server demo]$ ls -l
total 4
-rw-rw-r--. 1 tcarrigan tcarrigan 12 Aug 29 14:27 link_test

[tcarrigan@server demo]$ ln link_test /tmp/link_new

[tcarrigan@server demo]$ ls -li link_test /tmp/link_new 
2730074 -rw-rw-r--. 2 tcarrigan tcarrigan 12 Aug 29 14:27 link_test
2730074 -rw-rw-r--. 2 tcarrigan tcarrigan 12 Aug 29 14:27 /tmp/link_new
```
- When changes are made to one filename, the other reflects those changes. The permissions, link count, ownership, timestamps, and file content are the exact same. If the original file is deleted, the data still exists under the secondary hard link. The data is only removed from your drive when all links to the data have been removed. If you find two files with identical properties but are unsure if they are hard-linked, use the ls -i command to view the inode number. Files that are hard-linked together share the same inode number.
- Hard links an only be created for regular files (not directories or special files). Also, a hard link cannot span multiple filesystems. They only work when the new hard link exists on the same filesystem as the original.

### Soft Links
- Commonly referred to as symbolic links, soft links link together non-regular and regular files. They can also span multiple filesystems. By definition, a soft link is not a standard file, but a special file that points to an existing file.

``` bash
[tcarrigan@server demo]$ ln -s /home/tcarrigan/demo/soft_link_test /tmp/soft_link_new
[tcarrigan@server demo]$ ls -l soft_link_test /tmp/soft_link_new 
-rw-rw-r--. 1 tcarrigan tcarrigan 17 Aug 30 11:59 soft_link_test
lrwxrwxrwx. 1 tcarrigan tcarrigan 35 Aug 30 12:09 /tmp/soft_link_new -> /home/tcarrigan/demo/soft_link_test
```
- The biggest concern is data loss and data confusion. If the original file is deleted, the soft link is broken. This situation is referred to as a dangling soft link. If you were to create a new file with the same name as the original, your dangling soft link is no longer dangling at all. It points to the new file created, whether this was your intention or not, so be sure to keep this in mind.
- Soft link has seperate inode.

To keep the two easily separated in your mind, I leave you with this:
- A hard link always points a filename to data on a storage device.
- A soft link always points a filename to another filename, which then points to information on a storage device.
