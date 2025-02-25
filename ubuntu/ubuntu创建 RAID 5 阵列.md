## 创建 RAID 5 阵列

RAID 5 阵列类型是通过跨可用设备条带化数据来实现的。每个条带的一个组成部分是计算出的奇偶校验块。如果一个设备出现故障，奇偶校验块和剩余的块可以用来计算丢失的数据。轮换接收奇偶校验块的设备，以便每个设备都具有平衡数量的奇偶校验信息。



- 要求：至少 3 个存储设备。
- 主要优势：具有更多可用容量的冗余。
- 注意事项：在分发奇偶校验信息时，一个磁盘的容量将用于奇偶校验。当处于降级状态时，RAID 5 的性能可能会非常差。



### 识别组件设备

首先，找到您将使用的原始磁盘的标识符：

```dos
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
OutputNAME     SIZE FSTYPE   TYPE MOUNTPOINT
sda      100G          disk 
sdb      100G          disk 
sdc      100G          disk 
vda       25G          disk 
├─vda1  24.9G ext4     part /
├─vda14    4M          part 
└─vda15  106M vfat     part /boot/efi
vdb      466K iso9660  disk 
```

您有三个没有文件系统的磁盘，每个磁盘大小为 100G。这些设备已被赋予此会话的 `/dev/sda`、`/dev/sdb` 和 `/dev/sdc` 标识符，并将成为用于构建阵列的原始组件。

### 创建阵列

要使用这些组件创建 RAID 5 阵列，请将它们传递到 `mdadm --create` 命令中。您必须指定要创建的设备名称、RAID 级别和设备数量。在此命令示例中，您将命名设备 `/dev/md0`，并包括将构建阵列的磁盘：

```sql
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda /dev/sdb /dev/sdc
```

`mdadm` 工具将开始配置阵列。出于性能原因，它使用恢复过程来构建阵列。这可能需要一些时间才能完成，但可以在这段时间内使用数组。您可以通过检查 `/proc/mdstat` 文件来监控镜像的进度：

```powershell
cat /proc/mdstat
OutputPersonalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid5 sdc[3] sdb[1] sda[0]
      209582080 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
      [>....................]  recovery =  0.9% (957244/104791040) finish=18.0min speed=95724K/sec
      
unused devices: <none>
```

在第一个突出显示的行中，`/dev/md0` 设备是使用 `/dev/sda` 在 RAID 5 配置中创建的，` /dev/sdb` 和 `/dev/sdc` 设备。第二个突出显示的行显示了构建的进度。

警告：由于 `mdadm` 构建 RAID 5 阵列的方式，当阵列仍在构建时，阵列中的备件数量将被不准确地报告。这意味着您必须等待阵列完成组装才能更新 `/etc/mdadm/mdadm.conf` 文件。如果您在阵列仍在构建时更新配置文件，系统将获得有关阵列状态的错误信息，并且将无法在启动时使用正确的名称自动组装它。

在此过程完成后，您可以继续本指南。

### 创建和挂载文件系统

接下来，在阵列上创建一个文件系统：

```powershell
sudo mkfs.ext4 -F /dev/md0
```

创建一个挂载点来附加新的文件系统：

```dos
sudo mkdir -p /mnt/md0
```

您可以使用以下命令挂载文件系统：

```powershell
sudo mount /dev/md0 /mnt/md0
```

检查新空间是否可用：

```undefined
df -h -x devtmpfs -x tmpfs
OutputFilesystem      Size  Used Avail Use% Mounted on
/dev/vda1        25G  1.4G   23G   6% /
/dev/vda15      105M  3.4M  102M   4% /boot/efi
/dev/md0        197G   60M  187G   1% /mnt/md0
```

新文件系统已挂载并可访问。

### 保存阵列布局

为确保阵列在启动时自动重新组装，您必须调整 `/etc/mdadm/mdadm.conf` 文件。

警告：如前所述，在调整配置之前，请再次检查以确保阵列已完成组装。在构建阵列之前完成以下步骤将阻止系统在重新启动时正确组装阵列。

您可以通过检查 `/proc/mdstat` 文件来监控镜像的进度：

```powershell
cat /proc/mdstat
OutputPersonalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid5 sdc[3] sdb[1] sda[0]
      209584128 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>
```

此输出表明重建已完成。现在，您可以自动扫描活动数组并附加文件：

```powershell
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```

之后，您可以更新 `initramfs` 或初始 RAM 文件系统，以便在早期启动过程中可以使用该阵列：

```powershell
sudo update-initramfs -u
```

将新的文件系统挂载选项添加到 `/etc/fstab` 文件，以便在启动时自动挂载：

```powershell
echo '/dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
```

您的 RAID 5 阵列现在将自动组装和安装每个启动。

您现在已完成 RAID 设置。如果您想尝试不同的 RAID，请按照本教程开头的重置说明继续创建新的 RAID 阵列类型。