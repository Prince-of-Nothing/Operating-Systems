# Laboratory Work 2 — Meet the OS You Will Build
### Course: Operating Systems
### Author: Pleșu Dinu FAF-241
## Part 1. Build it and boot it
```bash
make qemu
```
## Part 2. Use xv6 as the Unix it is

1. Three programs xv6 ships with: `echo`, `cat`, `console`.

2. For a pipe to work, two OS features must exist: (a) process creation, so the kernel can run the two ends of the pipe as separate processes (via `fork`); and (b) an inter-process communication mechanism, where the kernel keeps a shared buffer accessible through file descriptors so one process's `write` can be read by the other process via `read`.

3. The xv6 shell implements only the bare minimum — parsing, `fork`/`exec`, pipes, and I/O redirection — while the Linux shell (bash) builds on the same core ideas but adds job control, scripting, variables, and many built-in commands.

## Part 3 — Memory

While analysing cat.c, I have noticed that the c program makes numerous system calls, including read() - gets user input, exit() - ends the program with a status, write() - writes to the output stream, open() - opens a file and close() - closes a file. The sys_read returns a uint64 and is defined in kernel/sysfile.c, on line 68. The files in the kernel define the instructions that the kernel is able to do, and how it does them, while the files defined in user allow the user to interact with the kernel's implemented behavior using programs.
## Part 4 — Devices and storage

```bash
df -h
lsblk
du -sh ~/os-lab1
ls -l /dev | head
mount | head
```

Output:

```text
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           1.5G  1.9M  1.5G   1% /run
/dev/sda2        51G  6.9G   42G  15% /
tmpfs           3.7G     0  3.7G   0% /dev/shm
tmpfs           3.7G  8.0K  3.7G   1% /tmp
none            1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
none            1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
Shared          699G  535G  164G  77% /home/vboxuser
tmpfs           742M   80K  742M   1% /run/user/1000
```

```text
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0  66.8M  1 loop /snap/core24/1643
loop2    7:2    0    20M  1 loop /snap/desktop-security-center/151
loop3    7:3    0 260.3M  1 loop /snap/firefox/8763
loop4    7:4    0  16.5M  1 loop /snap/firmware-updater/226
loop5    7:5    0 614.5M  1 loop /snap/gnome-46-2404/164
loop6    7:6    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop7    7:7    0   1.5M  1 loop /snap/hwctl/123
loop8    7:8    0   402M  1 loop /snap/mesa-2404/1839
loop9    7:9    0  18.8M  1 loop /snap/prompting-client/222
loop10   7:10   0  11.8M  1 loop /snap/snap-store/1390
loop11   7:11   0  50.1M  1 loop /snap/snapd/27710
loop12   7:12   0   828K  1 loop /snap/snapd-desktop-integration/391
loop13   7:13   0  44.7M  1 loop /snap/snapd/28254
loop14   7:14   0   1.5M  1 loop /snap/hwctl/131
loop16   7:16   0  14.2M  1 loop /snap/desktop-security-center/206
sda      8:0    0  51.7G  0 disk 
├─sda1   8:1    0     1M  0 part 
└─sda2   8:2    0  51.7G  0 part /
sr0     11:0    1  1024M  1 rom  

```

```text
512	/home/vboxuser/os-lab1
```

`df -h` showed mounted filesystems and their free space. `lsblk` showed block devices such as disks and partitions. `du -sh` showed how much disk space my lab directory used.

```text
total 0
crw-r--r--  1 root     root     10, 235 Oct  5 11:25 autofs
drwxr-xr-x  2 root     root        460 Oct  5 11:26 block
drwxr-xr-x  2 root     root         80 Oct  5 11:17 bsg
crw-------  1 root     root     10, 234 Oct  5 11:17 btrfs-control
drwxr-xr-x  3 root     root         60 Oct  5 11:17 bus
lrwxrwxrwx+ 1 root     root          3 Oct  5 11:17 cdrom -> sr0
drwxr-xr-x  2 root     root       4020 Oct  5 11:25 char
crw-------  1 root     tty       5,   1 Oct  5 11:25 console
lrwxrwxrwx  1 root     root         11 Oct  5 11:16 core -> /proc/kcore
tmpfs on /run type tmpfs (rw,nosuid,nodev,size=1519616k,nr_inodes=819200,mode=755,inode64)
/dev/sda2 on / type ext4 (rw,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,size=2704484k,nr_inodes=676121,mode=755,inode64)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,inode64,usrquota)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=600,ptmxmode=000)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot,memory_hugetlb_accounting)
none on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)

```

`mount` showed which filesystem was attached to which point in the directory tree. In my case `/dev/sda2` was mounted at `/` and used the `ext4` filesystem.

`/dev` contained special device entries rather than normal text files. These entries give programs a file-like way to interact with devices and kernel drivers, which helped me understand the Unix idea that "everything is a file".

### Observations

The root filesystem `/` was mounted on `/dev/sda2`. One useful device example is the real-time clock device, such as `/dev/rtc` or `/dev/rtc0`, which gives access to the system hardware clock. The idea of "everything is a file" means that many devices and system resources are exposed through file-like interfaces, so programs can interact with them using similar operations to normal files. This part showed the difference between filesystems, disks, partitions, mount points and device interfaces.

## Conclusion

This lab reinforced how an operating system manages files, processes, memory, and devices/I/O. Using commands such as `ls`, `ps`, `free`, `df`, and `lsblk` allowed hands-on verification of concepts like mount points, device files in `/dev`, and resource reporting. The exercises clarified how these components interact and prepared me to apply system diagnostics in future labs.