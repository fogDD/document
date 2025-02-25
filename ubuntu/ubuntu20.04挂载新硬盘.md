# ubuntu20.04挂载新硬盘

### 1.查看硬盘分区情况

首先查看硬盘的分区情况，可以看到sde是没有挂载没有数据的，我们就用这个新盘去操作

```
lsblk
```

![](/Users/zhouqiting/Desktop/document/nvidia/ubuntu/ubuntu20.04挂载新硬盘/1712629871865.jpg)



查看硬盘当前信息

```
fdisk -l
```

![image-20240409103409134](/Users/zhouqiting/Desktop/document/nvidia/ubuntu/ubuntu20.04挂载新硬盘/image-20240409103409134.png)





### 2.对硬盘进行分区

在Command (m for help)提示符后面输入n，执行 add a new partition 指令给硬盘增加一个新分区。
出现Command action时，输入e，指定分区为扩展分区（extended）。
输入p(默认)，指定为主分区（primary） 出现Partition number(1-4)时，输入１表示只分一个区。
后续指定起启柱面（cylinder）号完成分区（输入default内容即可）。

在Command (m for help)提示符后面输入p，显示分区表。

在Command (m for help)提示符后面输入w，保存分区表。
系统提示：The partition table has been altered!

```
fdisk /dev/sde
```

![image-20240409105409597](/Users/zhouqiting/Desktop/document/nvidia/ubuntu/ubuntu20.04挂载新硬盘/image-20240409105409597.png)



```
fdisk -l
```

可以查看新建出来的分区

![image-20240409105715751](/Users/zhouqiting/Desktop/document/nvidia/ubuntu/ubuntu20.04挂载新硬盘/image-20240409105715751.png)

这里解释下为什么6T的硬盘只能最大分出2T，因为MBR分区只能最大分2T空间，GPT分区类型就可以突破2T



### 3.格式化分区

格式化成ext4

```
root@gpu1:~# mkfs -t ext4 /dev/sde
mke2fs 1.45.5 (07-Jan-2020)
Found a dos partition table in /dev/sde
Proceed anyway? (y,N) y
Discarding device blocks: adone                            
Creating filesystem with 1875366486 4k blocks and 234422272 inodes
Filesystem UUID: 4622dfa6-2c6e-43a7-9bb5-9dff13798989
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
        102400000, 214990848, 512000000, 550731776, 644972544

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done  
```



### 4.挂载硬盘到需要的目录中

我这边挂载到mnt

```
root@gpu1:~# mount -t ext4 /dev/sde /mnt
```

![image-20240409110731741](/Users/zhouqiting/Desktop/document/nvidia/ubuntu/ubuntu20.04挂载新硬盘/image-20240409110731741.png)



#### 4.1永久挂载

进入/etc/fstab文件挂载

![image-20240409110848054](/Users/zhouqiting/Desktop/document/nvidia/ubuntu/ubuntu20.04挂载新硬盘/image-20240409110848054.png)