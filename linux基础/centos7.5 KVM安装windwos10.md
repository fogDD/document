# centos7.5 KVM安装windwos10

### 1.准备vfd驱动

```
wget --no-check-certificate  https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.173-2/virtio-win-0.1.173_amd64.vfd
```

我放在了我的kvm目录下

```
[root@zbx-server windows1001]# ll
total 2880
-rw-r--r-- 1 root root 2949120 Jan  9 14:37 virtio-win-0.1.173_amd64.vfd
You have new mail in /var/spool/mail/root
[root@zbx-server windows1001]# pwd
/kvm_data/windows1001
```



### 2.创建虚拟机磁盘文件

```
[root@zbx-server windows1001]# qemu-img create -f qcow2 /kvm_data/windows1001/windows1001.qcow2 30G
Formatting '/kvm_data/windows1001/windows1001.qcow2', fmt=qcow2 size=32212254720 encryption=off cluster_size=65536 lazy_refcounts=off 
[root@zbx-server windows1001]# ll
total 3076
-rw-r--r-- 1 root root 2949120 Jan  9 14:37 virtio-win-0.1.173_amd64.vfd
-rw-r--r-- 1 root root  197120 Jan  9 14:48 windows1001.qcow2
```



### 3.创建虚拟机

```
[root@zbx-server ~]# virt-install --virt-type kvm \
> --name windows1001 \
> --memory 4096 \
> --vcpus 1 \
> --disk path=/kvm_data/windows1001/windows1001.qcow2 \
> --disk path=/kvm_data/windows1001/virtio-win-0.1.173_amd64.vfd,device=floppy \
> --cdrom /root/package/cn_windows_10_business_editions_version_1909_updated_jan_2020_x64_dvd_b3e1f3a6.iso \
> --network bridge=br0 \
> --graphics vnc,listen=0.0.0.0 \
> --noautoconsole
WARNING  No operating system detected, VM performance may suffer. Specify an OS with --os-variant for optimal results.

Starting install...
Domain installation still in progress. You can reconnect to 
the console to complete the installation process.
[root@zbx-server ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 5     windows1001                    running
 -     testcentos701                  shut off
```



### 4.VNC连接虚拟机