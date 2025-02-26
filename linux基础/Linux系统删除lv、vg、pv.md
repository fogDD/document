# Linux系统删除lv、vg、pv

环境：系统ubuntu20.04

### 1.查看准备信息

查看现有卷组的相关信息

```
turing@ubuntu:~$ sudo vgscan
  Found volume group "ubuntu" using metadata type lvm2
```

查看卷组ubuntu包含的PV、LV信息。

```
turing@ubuntu:~$ sudo vgdisplay -v ubuntu
[sudo] password for turing: 
  --- Volume group ---
  VG Name               ubuntu
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               60.00 GiB
  PE Size               4.00 MiB
  Total PE              15360
  Alloc PE / Size       15360 / 60.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               kBsq7t-bsLX-Sd8H-09Gh-uUib-Uqeq-KHaVVq
   
  --- Logical volume ---
  LV Path                /dev/ubuntu/root
  LV Name                root
  VG Name                ubuntu
  LV UUID                g2RE2H-SN7O-b8he-GvZF-W2Xr-Fiia-ZGgeOO
  LV Write Access        read/write
  LV Creation host, time ubuntu, 2023-02-06 01:38:55 -0800
  LV Status              available
  # open                 0
  LV Size                60.00 GiB
  Current LE             15360
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
  --- Physical volumes ---
  PV Name               /dev/sda3     
  PV UUID               4ycGZh-H0cK-0rjJ-pY5S-UUti-gSor-UkZv3I
  PV Status             allocatable
  Total PE / Free PE    15360 / 0
```



### 2.删除逻辑卷lv

先卸载卷组上的逻辑卷LV

```
turing@ubuntu:~$ umount /dev/ubuntu/root
umount: /dev/mapper/ubuntu-root: not mounted.
```

这里没挂载



删除逻辑卷lv

```
turing@ubuntu:~$ sudo lvremove /dev/ubuntu/root
Do you really want to remove and DISCARD active logical volume ubuntu/root? [y/n]: y
  Logical volume "root" successfully removed
turing@ubuntu:~$ 
```

如果删除提示出错，那么检查下当前使用该逻辑卷的进程，然后结束该进程或者服务即可



验证是否删除

```
turing@ubuntu:~$ sudo lvdisplay
turing@ubuntu:~$ 
```



### 3.删除卷组vg

```
turing@ubuntu:~$ sudo vgremove ubuntu
  Volume group "ubuntu" successfully removed
```

vgchange -a n VolGroup00 ：安全删除



验证是否删除

```
turing@ubuntu:~$ sudo vgscan
turing@ubuntu:~$ 
```



### 4.删除物理卷pv

```
turing@ubuntu:~$ sudo pvremove /dev/sda3
  Labels on physical volume "/dev/sda3" successfully wiped.
turing@ubuntu:~$ 
```

检查是否删除

```
turing@ubuntu:~$ sudo pvscan
  No matching physical volumes found
```

