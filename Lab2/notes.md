# Laboratory Work 2 — Meet the OS You Will Build
### Course: Operating Systems
### Author: Pleșu Dinu FAF-241
## Part 1. Build it and boot it
```bash
make qemu
```
## Part 2. Use xv6 as the Unix it is


### Observations

PID 1 on my system was `systemd`, which is the main system manager started during boot. `ps aux | wc -l` gave roughly 370 running processes, although the exact number changed between runs. The `sleep` process had the state `S (sleeping)`, which made sense because it was waiting instead of actively using the CPU. This part showed how the OS keeps track of processes, assigns PIDs and exposes process information through `/proc` directories.

### Observe

1. Three programs xv6 ships with: `ls`, `cat`, `grep`.

2. For a pipe to work, two OS features must exist: (a) process creation, so the kernel can run the two ends of the pipe as separate processes (via `fork`); and (b) an inter-process communication mechanism, where the kernel keeps a shared buffer accessible through file descriptors so one process's `write` can be read by the other process via `read`.

3. The xv6 shell implements only the bare minimum — parsing, `fork`/`exec`, pipes, and I/O redirection — while the Linux shell (bash) builds on the same core ideas but adds job control, scripting, variables, and many built-in commands.

## Part 3 — Memory

```bash
free -h
cat /proc/meminfo | head -6
```

Output:

```text
sleep 300 & PID=$!
grep VmRSS /proc/$PID/status # real RAM used by this process
kill $PID
               total        used        free      shared  buff/cache   available
Mem:           7.2Gi       1.6Gi       4.0Gi        41Mi       1.9Gi       5.6Gi
Swap:             0B          0B          0B
MemTotal:        7598072 kB
MemFree:         4184516 kB
MemAvailable:    5877904 kB
Buffers:           51672 kB
Cached:          1872036 kB
SwapCached:            0 kB
[1] 13599
VmRSS:	     912 kB
[1]+  Terminated                 sleep 300
```

`free -h` gave a readable summary of RAM and swap. `/proc/meminfo` showed the same information in more detail. I noticed that `free` memory was roughly 2/3  of the `available` memory because Linux was using some RAM for cache that could be reused when needed.

```bash
sleep 300 & PID=$!
grep VmRSS /proc/$PID/status
kill $PID
```

Output:

```text
[1] 13632
VmRSS:	     204 kB
[1]+  Terminated                 sleep 300
```

`VmRSS` showed how much physical RAM was resident for the process. Even a simple sleeping process still used several megabytes for its executable, libraries, stack and other mappings.

### Observations

The VM had about 7.2 GiB of RAM. Around 4 GiB was completely free at one point, but much more was still available because cached memory can be reassigned. No swap was configured, so the swap size was `0B`. The `sleep` process used about 0.2 MB of resident memory, which was more than I thought considering that Nasa's space shuttle ran on only one MB of RAM. 

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