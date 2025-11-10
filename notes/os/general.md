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
