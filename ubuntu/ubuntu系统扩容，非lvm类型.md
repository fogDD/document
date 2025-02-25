# linux系统扩容，非lvm类型

环境：ubuntu20.04 磁盘50G大小扩容到100G 文件系统类型ext4

### 1.查看信息

```
turing@turing-virtual-machine:~/Desktop$ df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs          tmpfs     792M  1.8M  790M   1% /run
/dev/sda5      ext4       49G  7.8G   39G  17% /
tmpfs          tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs          tmpfs     5.0M  4.0K  5.0M   1% /run/lock
tmpfs          tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/loop0     squashfs   55M   55M     0 100% /snap/core18/1705
/dev/loop3     squashfs   50M   50M     0 100% /snap/snap-store/433
/dev/loop2     squashfs  241M  241M     0 100% /snap/gnome-3-34-1804/24
/dev/loop4     squashfs   28M   28M     0 100% /snap/snapd/7264
/dev/loop1     squashfs   63M   63M     0 100% /snap/gtk-common-themes/1506
/dev/sda1      vfat      511M  4.0K  511M   1% /boot/efi
tmpfs          tmpfs     792M   20K  792M   1% /run/user/1000
/dev/sr0       iso9660   2.6G  2.6G     0 100% /media/turing/Ubuntu 20.04 LTS amd64
```

根目录只有59G，还有60G没用



### 2.扩展根目录

```
turing@turing-virtual-machine:~/Desktop$ sudo growpart /dev/sda5
CHANGED: partition=5 start=1052672 old: size=103802880 end=104855552 new: size=208662495 end=209715167
```



查看扩展效果

```
turing@turing-virtual-machine:~/Desktop$ sudo fdisk -l
Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x2c8eff85

Device     Boot   Start       End   Sectors  Size Id Type
/dev/sda1  *       2048   1050623   1048576  512M  b W95 FAT32
/dev/sda2       1052670 209715166 208662497 99.5G  5 Extended
/dev/sda5       1052672 209715166 208662495 99.5G 83 Linux


Disk /dev/sdb: 50 GiB, 53687091200 bytes, 104857600 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

可以看到分区已经扩展成功了



```
turing@turing-virtual-machine:~/Desktop$ df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs          tmpfs     792M  1.8M  790M   1% /run
/dev/sda5      ext4       49G  7.8G   39G  17% /
tmpfs          tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs          tmpfs     5.0M  4.0K  5.0M   1% /run/lock
tmpfs          tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/loop0     squashfs   55M   55M     0 100% /snap/core18/1705
/dev/loop3     squashfs   50M   50M     0 100% /snap/snap-store/433
/dev/loop2     squashfs  241M  241M     0 100% /snap/gnome-3-34-1804/24
/dev/loop4     squashfs   28M   28M     0 100% /snap/snapd/7264
/dev/loop1     squashfs   63M   63M     0 100% /snap/gtk-common-themes/1506
/dev/sda1      vfat      511M  4.0K  511M   1% /boot/efi
tmpfs          tmpfs     792M   20K  792M   1% /run/user/1000
/dev/sr0       iso9660   2.6G  2.6G     0 100% /media/turing/Ubuntu 20.04 LTS amd64
```

但是磁盘空间还没显示出来



### 3.刷新分区

```
turing@turing-virtual-machine:~/Desktop$ sudo resize2fs /dev/sda5
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/sda5 is mounted on /; on-line resizing required
old_desc_blocks = 7, new_desc_blocks = 13
The filesystem on /dev/sda5 is now 26082811 (4k) blocks long.
```



查看效果

```
turing@turing-virtual-machine:~/Desktop$ df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs          tmpfs     792M  1.8M  790M   1% /run
/dev/sda5      ext4       98G  7.8G   86G   9% /
tmpfs          tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs          tmpfs     5.0M  4.0K  5.0M   1% /run/lock
tmpfs          tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/loop0     squashfs   55M   55M     0 100% /snap/core18/1705
/dev/loop3     squashfs   50M   50M     0 100% /snap/snap-store/433
/dev/loop2     squashfs  241M  241M     0 100% /snap/gnome-3-34-1804/24
/dev/loop4     squashfs   28M   28M     0 100% /snap/snapd/7264
/dev/loop1     squashfs   63M   63M     0 100% /snap/gtk-common-themes/1506
/dev/sda1      vfat      511M  4.0K  511M   1% /boot/efi
tmpfs          tmpfs     792M   20K  792M   1% /run/user/1000
/dev/sr0       iso9660   2.6G  2.6G     0 100% /media/turing/Ubuntu 20.04 LTS amd64
```

扩展成功