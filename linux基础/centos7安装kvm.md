# centos7安装kvm

### 1.关闭iptables或者firewalld，关闭selinux。并且检查cpu参数是否支持虚拟化

```
[root@zbx-server ~]# systemctl stop firewalld
[root@zbx-server ~]# systemctl stop iptables
Failed to stop iptables.service: Unit iptables.service not loaded.
[root@zbx-server ~]# systemctl stop iptable
Failed to stop iptable.service: Unit iptable.service not loaded.
[root@zbx-server ~]# systemctl disable firewalld   
[root@zbx-server ~]# getenforce 
Disabled
[root@zbx-server ~]# grep -Ei 'vmx|smx' /proc/cpuinfo 
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single ssbd ibrs ibpb stibp tpr_shadow vnmi ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 invpcid rdseed adx smap clflushopt xsaveopt xsavec xgetbv1 arat md_clear spec_ctrl intel_stibp flush_l1d arch_capabilities
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single ssbd ibrs ibpb stibp tpr_shadow vnmi ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 invpcid rdseed adx smap clflushopt xsaveopt xsavec xgetbv1 arat md_clear spec_ctrl intel_stibp flush_l1d arch_capabilities
```

如果有过滤出vmx或svm关键字就代表支持虚拟化，vmx是Intel的CPU，svm是AMD的CPU



### 2.添加新磁盘供kvm使用

格式化新磁盘

```
[root@zbx-server ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part 
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0   15G  0 disk 
sr0              11:0    1  4.5G  0 rom  
[root@zbx-server ~]# mkfs.ext4 /dev/sdb
mke2fs 1.42.9 (28-Dec-2013)
/dev/sdb is entire device, not just one partition!
Proceed anyway? (y,n) y
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
983040 inodes, 3932160 blocks
196608 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2151677952
120 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done   

You have new mail in /var/spool/mail/root
[root@zbx-server ~]# blkid /dev/sdb
/dev/sdb: UUID="437b61e4-86c6-4e35-a2a7-1a53a370b5a9" TYPE="ext4" 
```

新建目录，并把新磁盘挂载上

```
[root@zbx-server ~]# mkdir /kvm_data
[root@zbx-server ~]# vim /etc/fstab
####################################################################################################
# /etc/fstab
# Created by anaconda on Wed Jul 21 06:28:54 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=de2af4db-e5d6-4169-ad59-39cc0bb56f79 /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
/dev/sdb        /kvm_data       ext4     defaults        0 0
######################################################################################################
[root@zbx-server ~]# mount -a
```



### 3.yum安装kvm

```
yum -y install virt-* libvirt bridge-utils qemu-img
```



### 4.配置虚拟网卡

拷贝虚拟网卡（记得备份原网卡文件）

```
[root@localhost ~]# cd /etc/sysconfig/network-scripts/  
# 拷贝当前的网卡文件，作为一个桥接网卡
[root@localhost /etc/sysconfig/network-scripts]# cp ifcfg-ens33 ifcfg-br0  
# 修改原来的网卡内容
[root@localhost /etc/sysconfig/network-scripts]# vim ifcfg-ens33
####################################################################################
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPADDR=192.168.12.110
GATEWAY=192.168.12.2
NETMASK=255.255.255.0
DNS1=114.114.114.114
IPADDR1=192.168.12.109
NETMASK=255.255.255.0
NAME=ens33
DEVICE=ens33
ONBOOT=yes
BRIDGE=br0               ##主要加了这个选项，网卡
#####################################################################################
[root@zbx-server network-scripts]# vim ifcfg-br0
#####################################################################################
TYPE=Bridge               ##网卡类型改成桥
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPADDR=192.168.12.110
GATEWAY=192.168.12.2
NETMASK=255.255.255.0
DNS1=114.114.114.114
IPADDR1=192.168.12.109
NETMASK=255.255.255.0
NAME=br0                 ##修改网卡名称
DEVICE=br0               ##修改网卡设备
ONBOOT=yes
#####################################################################################
[root@localhost ~]# systemctl restart network        ##重启网络
```

修改完后，ip a查看是否修改成功

网络服务重启之后使用ifconfig命令可以看到此时ens33网卡的IP到br0网卡上了

```
[root@zbx-server network-scripts]# ip a
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br0 state UP group default qlen 1000
    link/ether 00:0c:29:80:24:f0 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::20c:29ff:fe80:24f0/64 scope link 
       valid_lft forever preferred_lft forever
44: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:0c:29:80:24:f0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.12.110/24 brd 192.168.12.255 scope global br0
       valid_lft forever preferred_lft forever
    inet 192.168.12.109/24 brd 192.168.12.255 scope global secondary br0
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe80:24f0/64 scope link 
       valid_lft forever preferred_lft forever
```



### 5.启动kvm服务

```
[root@zbx-server ~]# lsmod | grep kvm    # 检查KVM模块是否加载
kvm_intel             188688  0 
kvm                   636969  1 kvm_intel
irqbypass              13503  1 kvm
[root@zbx-server ~]# 
[root@zbx-server ~]# systemctl start libvirtd    # 启动libvirtd服务
You have new mail in /var/spool/mail/root
[root@zbx-server ~]# ps -ef | grep libvirtd
root      11657      1  1 10:21 ?        00:00:00 /usr/sbin/libvirtd
root      11754  11610  0 10:21 pts/1    00:00:00 grep --color=auto libvirtd
[root@zbx-server ~]# brctl show     # 显示当前所有桥接接口
bridge name     bridge id               STP enabled     interfaces
br-942a790d0e1b         8000.0242fb84eb8f       no              veth156693c
                                                        veth31a350c
                                                        veth350ab76
                                                        veth621a3fa
                                                        veth66807d1
                                                        veth68e5074
                                                        veth748a67d
                                                        veth93c43a8
                                                        veth9847d3a
                                                        vetha352bd9
                                                        vethb612786
                                                        vethdefe4aa
                                                        vethe29da1e
                                                        vethe4b09f4
br0             8000.000c298024f0       no              ens33
docker0         8000.02423352cf49       no
virbr0          8000.525400538b59       yes             virbr0-nic
```

brctl详解：https://blog.csdn.net/weixin_44983653/article/details/103138223



### 6.创建虚拟机

将服务成功启动后，我们就可以使用KVM安装虚拟机了，首先需要准备一个操作系统的镜像文件，我这里用的是CentOS7的镜像文件：

使用命令行安装这个CentOS7镜像文件：

```
[root@localhost ~]# virt-install --name=testcentos701 --memory=512,maxmemory=1024 --vcpus=1,maxvcpus=2 --os-type=linux --os-variant=rhel7 --location=/root/package/CentOS-7-x86_64-DVD-2003.iso --disk path=/kvm_data/testcentos701,size=10 --bridge=br0 --graphics=none --console=pty,target_type=serial  --extra-args="console=tty0 console=ttyS0"
```

命令说明：

–name 指定虚拟机的名称
–memory 指定分配给虚拟机的内存资源大小
maxmemory 指定可调节的最大内存资源大小，因为KVM支持热调整虚拟机的资源
–vcpus 指定分配给虚拟机的CPU核心数量
maxvcpus 指定可调节的最大CPU核心数量
–os-type 指定虚拟机安装的操作系统类型
–os-variant 指定系统的发行版本
–location 指定ISO镜像文件所在的路径，支持使用网络资源路径，也就是说可以使用URL
–disk path 指定虚拟硬盘所存放的路径及名称，size 则是指定该硬盘的可用大小，单位是G
–bridge 指定使用哪一个桥接网卡，也就是说使用桥接的网络模式
–graphics 指定是否开启图形
–console 定义终端的属性，target_type 则是定义终端的类型

–extra-args 定义终端额外的参数

开始安装后会显示安装引导界面（其实跟普通安装CentOS一样，只不过平时是用图形界面安装，这里是用命令行来展现），[!]代表你需要配置的，[x]代表你已经设置了

```
Starting installer, one moment...
anaconda 21.48.22.158-1 for CentOS 7 started.
 * installation log files are stored in /tmp during the installation
 * shell is available on TTY2
 * when reporting a bug add logs from /tmp as separate text/plain attachments
09:00:45 Not asking for VNC because we don't have a network
================================================================================
================================================================================
Installation

 1) [x] Language settings                 2) [!] Time settings
        (English (United States))                (Timezone is not set.)
 3) [!] Installation source               4) [!] Software selection
        (Processing...)                          (Processing...)
 5) [!] Installation Destination          6) [x] Kdump
        (No disks selected)                      (Kdump is enabled)
 7) [ ] Network configuration             8) [!] Root password
        (Not connected)                          (Password is not set.)
 9) [!] User creation
        (No user will be created)
  Please make your choice from above ['q' to quit | 'b' to begin installation |
  'r' to refresh]: 
```

根据数字去选择要配置的选项，我们按1回车进入语言设置

```
1)  Afrikaans             25)  Hindi                 48)  Oriya
 2)  Amharic               26)  Croatian              49)  Punjabi
 3)  Arabic                27)  Hungarian             50)  Polish
 4)  Assamese              28)  Interlingua           51)  Portuguese
 5)  Asturian              29)  Indonesian            52)  Romanian
 6)  Belarusian            30)  Icelandic             53)  Russian
 7)  Bulgarian             31)  Italian               54)  Sinhala
 8)  Bengali               32)  Japanese              55)  Slovak
 9)  Bosnian               33)  Georgian              56)  Slovenian
10)  Catalan               34)  Kazakh                57)  Albanian
11)  Czech                 35)  Kannada               58)  Serbian
12)  Welsh                 36)  Korean                59)  Swedish
13)  Danish                37)  Lithuanian            60)  Tamil
14)  German                38)  Latvian               61)  Telugu
15)  Greek                 39)  Maithili              62)  Tajik
16)  English               40)  Macedonian            63)  Thai
17)  Spanish               41)  Malayalam             64)  Turkish
18)  Estonian              42)  Marathi               65)  Ukrainian
19)  Basque                43)  Malay                 66)  Urdu
20)  Persian               44)  Norwegian Bokmål      67)  Vietnamese
21)  Finnish               45)  Nepali                68)  Chinese
22)  French                46)  Dutch                 69)  Zulu
Press ENTER to continue
[anaconda] 1:main* 2:shell  3:log  4:storage-lo> Switch tab: Alt+Tab | Help: F1 
```

这里我选择68中文，后会让你选择繁体还是简体

```
Please select language support to install.
[b to return to language list, c to continue, q to quit]: 68
================================================================================
================================================================================
Language settings

Available locales
 1)  Simplified Chinese     3)  Traditional Chinese    4)  Simplified Chinese
     (China)                    (Hong Kong)                (Singapore)
 2)  Traditional Chinese
     (Republic of China)
Please select language support to install.
[b to return to language list, c to continue, q to quit]: 1
```

1是简体

接下来设置时区

```
Please make your choice from above ['q' to quit | 'b' to begin installation |
  'r' to refresh]: 2
================================================================================
================================================================================
Time settings

Timezone: not set

NTP servers:not configured

 1)  Set timezone
 2)  Configure NTP servers
  Please make your choice from above ['q' to quit | 'c' to continue |
  'r' to refresh]: 
```

1设置时区

65选择上海

```
================================================================================
================================================================================
Timezone settings

Available regions
 1)  Europe                 6)  Pacific               10)  Arctic
 2)  Asia                   7)  Australia             11)  US
 1)  Aden                  29)  Hong_Kong             56)  Pontianak
 2)  Almaty                30)  Hovd                  57)  Pyongyang
 3)  Amman                 31)  Irkutsk               58)  Qatar
 4)  Anadyr                32)  Jakarta               59)  Qostanay
 5)  Aqtau                 33)  Jayapura              60)  Qyzylorda
 6)  Aqtobe                34)  Jerusalem             61)  Riyadh
 7)  Ashgabat              35)  Kabul                 62)  Sakhalin
 8)  Atyrau                36)  Kamchatka             63)  Samarkand
 9)  Baghdad               37)  Karachi               64)  Seoul
10)  Bahrain               38)  Kathmandu             65)  Shanghai
11)  Baku                  39)  Khandyga              66)  Singapore
12)  Bangkok               40)  Kolkata               67)  Srednekolymsk
13)  Barnaul               41)  Krasnoyarsk           68)  Taipei
14)  Beirut                42)  Kuala_Lumpur          69)  Tashkent
15)  Bishkek               43)  Kuching               70)  Tbilisi
16)  Brunei                44)  Kuwait                71)  Tehran
17)  Chita                 45)  Macau                 72)  Thimphu
18)  Choibalsan            46)  Magadan               73)  Tokyo
19)  Colombo               47)  Makassar              74)  Tomsk
20)  Damascus              48)  Manila                75)  Ulaanbaatar
21)  Dhaka                 49)  Muscat                76)  Urumqi
22)  Dili                  50)  Nicosia               77)  Ust-Nera
Press ENTER to continue65
```

5设置系统安装盘

```
================================================================================
================================================================================
Installation

 1) [x] Language settings                 2) [x] Time settings
        (Simplified Chinese (China))             (Asia/Shanghai timezone)
 3) [x] Installation source               4) [x] Software selection
        (Local media)                            (Minimal Install)
 5) [!] Installation Destination          6) [x] Kdump
        (No disks selected)                      (Kdump is enabled)
 7) [ ] Network configuration             8) [!] Root password
        (Not connected)                          (Password is not set.)
 9) [!] User creation
        (No user will be created)
  Please make your choice from above ['q' to quit | 'b' to begin installation |
  'r' to refresh]: 5
```

```
================================================================================

Probing storage...
Installation Destination

[x] 1) : 10 GiB (vda)

1 disk selected; 10 GiB capacity; 10 GiB free ...

  Please make your choice from above ['q' to quit | 'c' to continue |
  'r' to refresh]: c
```

c回车，默认选择

```
================================================================================

Autopartitioning Options

[ ] 1) Replace Existing Linux system(s)

[x] 2) Use All Space

[ ] 3) Use Free Space

Installation requires partitioning of your hard drive. Select what space to use
for the install target.

  Please make your choice from above ['q' to quit | 'c' to continue |
  'r' to refresh]: c
```

继续默认

```
================================================================================
Partition Scheme Options

[ ] 1) Standard Partition

[ ] 2) Btrfs

[x] 3) LVM

[ ] 4) LVM Thin Provisioning

Select a partition scheme configuration.

  Please make your choice from above ['q' to quit | 'c' to continue |
  'r' to refresh]: 1
```

不需要lvm，需要默认分区，选1

```
================================================================================
Partition Scheme Options

[x] 1) Standard Partition

[ ] 2) Btrfs

[ ] 3) LVM

[ ] 4) LVM Thin Provisioning

Select a partition scheme configuration.

  Please make your choice from above ['q' to quit | 'c' to continue |
  'r' to refresh]: c
```

默认

```
================================================================================
Installation

 1) [x] Language settings                 2) [x] Time settings
        (Simplified Chinese (China))             (Asia/Shanghai timezone)
 3) [x] Installation source               4) [x] Software selection
        (Local media)                            (Minimal Install)
 5) [x] Installation Destination          6) [x] Kdump
        (Automatic partitioning                  (Kdump is enabled)
        selected)                         8) [!] Root password
 7) [ ] Network configuration                    (Password is not set.)
        (Not connected)
 9) [!] User creation
        (No user will be created)
  Please make your choice from above ['q' to quit | 'b' to begin installation |
  'r' to refresh]: 8
```

8设置root密码

```
================================================================================
Please select new root password. You will have to type it twice.

Password: 
Password (confirm):
```

输入密码即可

最后开始安装系统

```
================================================================================
Installation

 1) [x] Language settings                 2) [x] Time settings
        (Simplified Chinese (China))             (Asia/Shanghai timezone)
 3) [x] Installation source               4) [x] Software selection
        (Local media)                            (Minimal Install)
 5) [x] Installation Destination          6) [x] Kdump
        (Automatic partitioning                  (Kdump is enabled)
        selected)                         8) [x] Root password
 7) [ ] Network configuration                    (Password is set.)
        (Not connected)
 9) [ ] User creation
        (No user will be created)
  Please make your choice from above ['q' to quit | 'b' to begin installation |
  'r' to refresh]: b
```

安装完后回车重启

```
CentOS Linux 7 (Core)
Kernel 3.10.0-1127.el7.x86_64 on an x86_64

localhost login: 
```

有登录界面就代表安装成功了

此时我们所在的是一个KVM虚拟机的终端，如果想要切换回原本那个系统终端的话可以使用Ctrl+]命令

















### 遇到的问题

#### 1.kvm创建虚拟机时提示没有权限打开ios文件

```
[root@zbx-server package]# virt-install --name=testcentos701 --memory=512,maxmemory=1024 --vcpus=1,maxvcpus=2 --os-type=linux --os-variant=rhel7 --location=/root/package/CentOS-7-x86_64-DVD-2003.iso --disk path=/kvm_data/testcentos701,size=10 --bridge=br0 --graphics=none --console=pty,target_type=serial  --extra-args="console=tty0 console=ttyS0"

Starting install...
Retrieving file .treeinfo...                                                                                                                      |  353 B  00:00:00     
Retrieving file vmlinuz...                                                                                                                        | 6.4 MB  00:00:00     
Retrieving file initrd.img...                                                                                                                     |  53 MB  00:00:00     
Allocating 'testcentos701'                                                                                                                        |  10 GB  00:00:00     
ERROR    internal error: qemu unexpectedly closed the monitor: 2023-01-06T08:43:16.327733Z qemu-kvm: -drive file=/root/package/CentOS-7-x86_64-DVD-2003.iso,format=raw,if=none,id=drive-ide0-0-0,readonly=on: could not open disk image /root/package/CentOS-7-x86_64-DVD-2003.iso: Could not open '/root/package/CentOS-7-x86_64-DVD-2003.iso': Permission denied
Removing disk 'testcentos701'                                                                                                                     |    0 B  00:00:00     
Domain installation does not appear to have been successful.
If it was, you can restart your domain by running:
  virsh --connect qemu:///system start testcentos701
otherwise, please restart your installation.
```

报错提示很明显了，没有权限打开iso文件，去把iso文件权限给满777，root:root属主和属组都不行，需要在/etc/libvirt/qemu.conf，更改配置文件，全局配置，通过服务的配置文件放开root权限

```
[root@zbx-server ~]# vim /etc/libvirt/qemu.conf
#######################################################################################
#
#       user = "qemu"   # A user named "qemu"
#       user = "+0"     # Super user (uid=0)
#       user = "100"    # A user named "100" or a user with uid=100
#
user = "root"

# The group for QEMU processes run by the system instance. It can be
# specified in a similar way to user.
group = "root"

# Whether libvirt should dynamically change file ownership
# to match the configured user/group above. Defaults to 1.
# Set to 0 to disable file ownership changes.
#dynamic_ownership = 1
#######################################################################################
[root@zbx-server ~]# systemctl restart libvirtd
[root@zbx-server ~]# systemctl status libvirtd
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2023-01-06 16:59:33 CST; 5s ago
     Docs: man:libvirtd(8)
           https://libvirt.org
 Main PID: 7691 (libvirtd)
    Tasks: 19 (limit: 32768)
   Memory: 43.9M
   CGroup: /system.slice/libvirtd.service
           ├─1540 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
           ├─1541 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
           └─7691 /usr/sbin/libvirtd

Jan 06 16:59:33 zbx-server systemd[1]: Starting Virtualization daemon...
Jan 06 16:59:33 zbx-server systemd[1]: Started Virtualization daemon.
Jan 06 16:59:33 zbx-server dnsmasq[1540]: read /etc/hosts - 3 addresses
Jan 06 16:59:33 zbx-server dnsmasq[1540]: read /var/lib/libvirt/dnsmasq/default.addnhosts - 0 addresses
Jan 06 16:59:33 zbx-server dnsmasq-dhcp[1540]: read /var/lib/libvirt/dnsmasq/default.hostsfile
```

