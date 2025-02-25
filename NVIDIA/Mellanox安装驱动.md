# 安装驱动Mellanox

驱动官网网址：https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed

下载时注意官网版本对照说明，CX3 4 5 6 对应的驱动版本

**Note:** By downloading and installing MLNX_OFED package for Oracle Linux (OL) OS, you may be violating your operating system’s support matrix. Please consult with your operating system support before installing.

**Note:** MLNX_OFED 4.9-x LTS should be used by customers who would like to utilize one of the following:

- NVIDIA ConnectX-3 Pro
- NVIDIA ConnectX-3
- NVIDIA Connect-IB
- RDMA experimental verbs library (mlnx_lib)
- OSs based on kernel version lower than 3.10

**Note:** All of the above are not available on MLNX_OFED 5.x branch.

**Note:** MLNX_OFED 5.4/5.8-x LTS should be used by customers who would like to utilize NVIDIA ConnectX-4 onwards adapter cards and keep using stable 5.4/5.8-x deployment and get:

- Critical bug fixes

- Support for new major OSs

  ##########################################################################################

注意：通过下载和安装适用于Oracle Linux（OL）OS的MLNX_OFED软件包，您可能违反了操作系统的支持矩阵。安装前，请咨询操作系统支持人员。

注意：MLNX_OFED 4.9-x LTS应由希望使用以下功能之一的客户使用：

NVIDIA ConnectX-3 Pro

NVIDIA ConnectX-3

NVIDIA Connect IB

RDMA实验动词库（mlnx.lib）

基于低于3.10的内核版本的操作系统

注意：以上所有内容在MLNX_OFED5.x分支上都不可用。

注意：MLNX_OFED 5.4/5.8-x LTS应由希望使用NVIDIA ConnectX-4及以上适配器卡并继续使用稳定的5.4/5.8-x部署的客户使用，并获得：

关键错误修复

支持新的主要操作系统



### 1.通过查看pci检查网卡状态

通过pci查看网卡信息

```
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads/mft-4.22.0-96-x86_64-deb$ lspci | grep ConnectX
65:00.0 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6]
65:00.1 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6]
```

```
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads/mft-4.22.0-96-x86_64-deb$ lspci -s 65:00.0 -vv
65:00.0 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6]
        Subsystem: Mellanox Technologies MT28908 Family [ConnectX-6]
        Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr+ Stepping- SERR+ FastB2B- DisINTx+
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0, Cache Line Size: 32 bytes
        Interrupt: pin A routed to IRQ 42
        NUMA node: 0
        Region 0: Memory at 4bffe000000 (64-bit, prefetchable) [size=32M]
        Expansion ROM at d3200000 [disabled] [size=1M]
        Capabilities: <access denied>
        Kernel driver in use: mlx5_core
        Kernel modules: mlx5_core
```



### 2.开始安装驱动

```
[root@io01 mellanox]# tar -xf MLNX_OFED_LINUX-4.9-4.1.7.0-rhel7.6-x86_64.tgz 
[root@io01 mellanox]# ls
MLNX_OFED_LINUX-4.9-4.1.7.0-rhel7.6-x86_64
MLNX_OFED_LINUX-4.9-4.1.7.0-rhel7.6-x86_64.tgz
[root@io01 mellanox]# cd MLNX_OFED_LINUX-4.9-4.1.7.0-rhel7.6-x86_64
[root@io01 MLNX_OFED_LINUX-4.9-4.1.7.0-rhel7.6-x86_64]# ls
common_installers.pl            mlnx_add_kernel_support.sh
common.pl                       mlnxofedinstall
create_mlnx_ofed_installers.pl  RPM-GPG-KEY-Mellanox
distro                          RPMS
docs                            RPMS_UPSTREAM_LIBS
is_kmp_compat.sh                src
[root@io01 MLNX_OFED_LINUX-4.9-4.1.7.0-rhel7.6-x86_64]#sudo ./mlnxofedinstall
```

回车后如没有安装依赖包脚本会自动帮安装

```
[root@io01 MLNX_OFED_LINUX-4.9-4.1.7.0-rhel7.6-x86_64]#/etc/init.d/openibd restart      # 安装完后重启openibd
Unloading HCA driver:                                      [  OK  ]
Loading HCA driver and Access Layer:                       [  OK  ]
```



### 3.查看网卡状态

```
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads$ ibstat
CA 'mlx5_0'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.35.2000
        Hardware version: 0
        Node GUID: 0xe8ebd303007236dc
        System image GUID: 0xe8ebd303007236dc
        Port 1:
                State: Down                            
                # 网卡逻辑状态： Active表示网卡状态正常运行
                Physical state: Disabled               
                # 网卡物理状态： LinkUp表示连接成功（连接交换机或其他主机）
                Rate: 200                              
                # 网卡速率
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0xeaebd3fffe7236dc
                Link layer: Ethernet                   
                # Ethernet：以太网状态    InfiniBand： ib状态
CA 'mlx5_1'
        CA type: MT4123
        Number of ports: 1
        Firmware version: 20.35.2000
        Hardware version: 0
        Node GUID: 0xe8ebd303007236dd
        System image GUID: 0xe8ebd303007236dc
        Port 1:
                State: Down
                Physical state: Disabled
                Rate: 200
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0xeaebd3fffe7236dd
                Link layer: Ethernet
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads$ ibstatus
Infiniband device 'mlx5_0' port 1 status:
        default gid:     fe80:0000:0000:0000:eaeb:d3ff:fe72:36dc
        base lid:        0x0
        sm lid:          0x0
        state:           1: DOWN
        phys state:      3: Disabled
        rate:            200 Gb/sec (4X HDR)
        link_layer:      Ethernet

Infiniband device 'mlx5_1' port 1 status:
        default gid:     fe80:0000:0000:0000:eaeb:d3ff:fe72:36dd
        base lid:        0x0
        sm lid:          0x0
        state:           1: DOWN
        phys state:      3: Disabled
        rate:            200 Gb/sec (4X HDR)
        link_layer:      Ethernet
```

查看已安装的驱动版本

```
[root@nodeb ~]# ofed_info -s
MLNX_OFED_LINUX-5.1-0.6.6.0:
```

卸载驱动则可以通过到安装驱动的文件夹下运行脚本卸载驱动程序。



### 4.安装MFT工具

MFT官网：https://network.nvidia.com/products/adapter-software/firmware-tools/

下载完后，解压工具，进入目录运行安装脚本即可

```
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads$ cd mft-4.22.0-96-x86_64-deb/
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads/mft-4.22.0-96-x86_64-deb$ ll
total 68
drwxr-xr-x 4 turing turing  4096 10月 26 23:50 ./
drwxr-xr-x 5 turing turing  4096 1月   4 11:36 ../
drwxr-xr-x 2 turing turing  4096 10月 26 23:50 DEBS/
-rwxr-xr-x 1 turing turing 22988 10月 26 23:50 install.sh*
-rwxr-xr-x 1 turing turing 13841 10月 26 23:50 LICENSE.txt*
-rwxr-xr-x 1 turing turing  7622 10月 26 23:50 old-mft-uninstall.sh*
drwxr-xr-x 2 turing turing  4096 10月 26 23:50 SDEBS/
-rwxr-xr-x 1 turing turing  1727 10月 26 23:50 uninstall.sh*
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads/mft-4.22.0-96-x86_64-deb$ sudo ./install.sh
```



#### 4.1MFT工具使用

安装完后需要启动服务

```
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads/mft-4.22.0-96-x86_64-deb$ sudo mst start
[sudo] password for turing: 
Starting MST (Mellanox Software Tools) driver set
Loading MST PCI module - Success
[warn] mst_pciconf is already loaded, skipping
Create devices

Unloading MST PCI module (unused) - Success
```

使用MFT工具查看ib网卡信息

```
(base) turing@turing-Pro-E800-G4-WS950T:~/Downloads/mft-4.22.0-96-x86_64-deb$ sudo mst status
MST modules:
------------
    MST PCI module is not loaded
    MST PCI configuration module loaded

MST devices:
------------
/dev/mst/mt4123_pciconf0         - PCI configuration cycles access.
                                   domain:bus:dev.fn=0000:65:00.0 addr.reg=88 data.reg=92 cr_bar.gw_offset=-1
                                   Chip revision is: 00
```



### 5.如两台服务器使用ib网卡互连，也就是服务器连服务器，不通过交换机，需要开启opensmd服务

```
systemctl start opensmd
systemctl enable opensmd
```



### 6.检查网卡运行环境

使用命令 mlnx_tune 检查，所有状态都是 OK，则网卡运行环境OK

```
(base) turing@turing-Pro-E800-G4-WS950T:~$ sudo mlnx_tune
[sudo] password for turing: 
2023-01-05 15:12:21,172 INFO Collecting node information
2023-01-05 15:12:21,172 INFO Collecting OS information
2023-01-05 15:12:21,277 INFO Collecting cpupower information
2023-01-05 15:12:21,279 WARNING Failed to run cmd: ls -l /etc/init.d/cpupower
2023-01-05 15:12:21,280 WARNING Unable to check service cpupower existence
2023-01-05 15:12:21,280 INFO Collecting watchdog information
2023-01-05 15:12:21,284 WARNING Failed to run cmd: ls -l /etc/init.d/watchdog
2023-01-05 15:12:21,284 WARNING Unable to check service watchdog existence
2023-01-05 15:12:21,284 INFO Collecting abrt-ccpp information
2023-01-05 15:12:21,288 WARNING Failed to run cmd: ls -l /etc/init.d/abrt-ccpp
2023-01-05 15:12:21,288 WARNING Unable to check service abrt-ccpp existence
2023-01-05 15:12:21,288 INFO Collecting abrtd information
2023-01-05 15:12:21,292 WARNING Failed to run cmd: ls -l /etc/init.d/abrtd
2023-01-05 15:12:21,292 WARNING Unable to check service abrtd existence
2023-01-05 15:12:21,292 INFO Collecting abrt-oops information
2023-01-05 15:12:21,296 WARNING Failed to run cmd: ls -l /etc/init.d/abrt-oops
2023-01-05 15:12:21,296 WARNING Unable to check service abrt-oops existence
2023-01-05 15:12:21,297 INFO Collecting alsa-state information
2023-01-05 15:12:21,300 WARNING Failed to run cmd: ls -l /etc/init.d/alsa-state
2023-01-05 15:12:21,301 WARNING Unable to check service alsa-state existence
2023-01-05 15:12:21,301 INFO Collecting anacorn information
2023-01-05 15:12:21,305 WARNING Failed to run cmd: ls -l /etc/init.d/anacorn
2023-01-05 15:12:21,305 WARNING Unable to check service anacorn existence
2023-01-05 15:12:21,305 INFO Collecting atd information
2023-01-05 15:12:21,309 WARNING Failed to run cmd: ls -l /etc/init.d/atd
2023-01-05 15:12:21,309 WARNING Unable to check service atd existence
2023-01-05 15:12:21,309 INFO Collecting avahi-daemon information
2023-01-05 15:12:21,374 INFO Collecting bluetooth information
2023-01-05 15:12:21,440 INFO Collecting certmonger information
2023-01-05 15:12:21,444 WARNING Failed to run cmd: ls -l /etc/init.d/certmonger
2023-01-05 15:12:21,445 WARNING Unable to check service certmonger existence
2023-01-05 15:12:21,445 INFO Collecting cups information
2023-01-05 15:12:21,512 INFO Collecting halddaemon information
2023-01-05 15:12:21,516 WARNING Failed to run cmd: ls -l /etc/init.d/halddaemon
2023-01-05 15:12:21,517 WARNING Unable to check service halddaemon existence
2023-01-05 15:12:21,517 INFO Collecting hidd information
2023-01-05 15:12:21,520 WARNING Failed to run cmd: ls -l /etc/init.d/hidd
2023-01-05 15:12:21,521 WARNING Unable to check service hidd existence
2023-01-05 15:12:21,521 INFO Collecting iprdump information
2023-01-05 15:12:21,525 WARNING Failed to run cmd: ls -l /etc/init.d/iprdump
2023-01-05 15:12:21,525 WARNING Unable to check service iprdump existence
2023-01-05 15:12:21,525 INFO Collecting iprinit information
2023-01-05 15:12:21,529 WARNING Failed to run cmd: ls -l /etc/init.d/iprinit
2023-01-05 15:12:21,529 WARNING Unable to check service iprinit existence
2023-01-05 15:12:21,530 INFO Collecting iprupdate information
2023-01-05 15:12:21,533 WARNING Failed to run cmd: ls -l /etc/init.d/iprupdate
2023-01-05 15:12:21,534 WARNING Unable to check service iprupdate existence
2023-01-05 15:12:21,534 INFO Collecting mdmonitor information
2023-01-05 15:12:21,538 WARNING Failed to run cmd: ls -l /etc/init.d/mdmonitor
2023-01-05 15:12:21,538 WARNING Unable to check service mdmonitor existence
2023-01-05 15:12:21,538 INFO Collecting polkit information
2023-01-05 15:12:21,542 WARNING Failed to run cmd: ls -l /etc/init.d/polkit
2023-01-05 15:12:21,542 WARNING Unable to check service polkit existence
2023-01-05 15:12:21,542 INFO Collecting rsyslog information
2023-01-05 15:12:21,930 INFO Collecting CPU information
2023-01-05 15:12:22,068 INFO Collecting memory information
2023-01-05 15:12:22,068 INFO Collecting hugepages information
2023-01-05 15:12:22,094 INFO Collecting IRQ Balancer information
2023-01-05 15:12:22,158 INFO Collecting Firewall information
2023-01-05 15:12:22,162 WARNING Failed to run cmd: ls -l /etc/init.d/firewalld
2023-01-05 15:12:22,163 WARNING Unable to check service firewalld existence
2023-01-05 15:12:22,163 INFO Collecting IP table information
2023-01-05 15:12:22,167 WARNING Failed to run cmd: ls -l /etc/init.d/iptables
2023-01-05 15:12:22,167 WARNING Unable to check service iptables existence
2023-01-05 15:12:22,167 INFO Collecting IPv6 table information
2023-01-05 15:12:22,171 WARNING Failed to run cmd: ls -l /etc/init.d/ip6tables
2023-01-05 15:12:22,171 WARNING Unable to check service ip6tables existence
2023-01-05 15:12:22,171 INFO Collecting IP forwarding information
2023-01-05 15:12:22,178 INFO Collecting hyper threading information
2023-01-05 15:12:22,178 INFO Collecting IOMMU information
2023-01-05 15:12:22,181 INFO Collecting driver information
2023-01-05 15:12:22,237 INFO Collecting Mellanox devices information

Mellanox Technologies - System Report

Operation System Status
Ubuntu20.04
5.14.0-1054-oem

CPU Status
GenuineIntel Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz Skylake
Warning: Frequency 2300.0MHz

Memory Status
Total: 125.40 GB
Free: 118.97 GB

Hugepages Status
On NUMA 0:
Transparent enabled: madvise
Transparent defrag: madvise

Hyper Threading Status
ACTIVE

IRQ Balancer Status
ACTIVE

Firewall Status
NOT PRESENT
IP table Status
NOT PRESENT
IPv6 table Status
NOT PRESENT

Driver Status
OK: MLNX_OFED_LINUX-5.8-1.1.2.1 (OFED-5.8-1.1.2)

ConnectX-6 Device Status on PCI 65:00.0
FW version 20.35.2000
OK: PCI Width x16
Warning: PCI Speed 8GT/s >>> PCI width status is below PCI capabilities. Check PCI configuration in BIOS.
PCI Max Payload Size 256
PCI Max Read Request 512
Local CPUs list [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31]

ens1f0np0 (Port 1) Status
Link Type eth
OK: Link status Up
Speed 200.0GbE 
MTU 1500
OK: TX nocache copy 'off'

ConnectX-6 Device Status on PCI 65:00.1
FW version 20.35.2000
OK: PCI Width x16
Warning: PCI Speed 8GT/s >>> PCI width status is below PCI capabilities. Check PCI configuration in BIOS.
PCI Max Payload Size 256
PCI Max Read Request 512
Local CPUs list [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31]

ens1f1np1 (Port 1) Status
Link Type eth
OK: Link status Up
Speed 200.0GbE 
MTU 1500
OK: TX nocache copy 'off'


2023-01-05 15:12:24,400 INFO System info file: /tmp/mlnx_tune_230105_151218.log
```



### 7.使用mlxcnfig命令切换ib网卡模式

其中，-d0表示IB网卡。

通过以下命令即可修改网卡的模式（把/dev/mst/mt4099_pciconf0改为你通过mst status查询得到的设备位置）：

双口网卡

```
mlxconfig -d /dev/mst/mt4099_pciconf0 set LINK_TYPE_P1=1 LINK_TYPE_P2=1
```

单口网卡

```
mlxconfig -d /dev/mst/mt4099_pciconf0 set LINK_TYPE_P1=1
```

备注：1：Infiniband模式；2：Ethernet模式

更改完后重启电脑





### 遇到的问题

#### 1.两台服务器通过ib网卡互联，但网卡开机运行一段时间后会自动关闭，网卡显示灯不亮，ibstat无显示内容

排查思路，检查驱动版本，MFT工具版本，硬件方面排查线缆的可用性，网卡是否插好（可用lspci命令检查）

上述都没检查出问题，想从网卡层面检查，我用lspci命令能看到网卡状态正常插入

```
terry@pai-worker1:~$ lspci | grep ConnectX
01:00.0 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6]
01:00.1 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6]
```

但是我查看pci设备的详细情况时发现了问题

```
terry@pai-worker1:~$ lspci -s 01:00.0 -vv
01:00.0 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6] (rev ff) (prog-if ff)
        !!! Unknown header type 7f
        Kernel driver in use: mlx5_core
        Kernel modules: mlx5_core
```

网卡正常运行的状态如下，能看到pci设备的详细信息

```
65:00.0 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6]
        Subsystem: Mellanox Technologies MT28908 Family [ConnectX-6]
        Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr+ Stepping- SERR+ FastB2B- DisINTx+
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0, Cache Line Size: 32 bytes
        Interrupt: pin A routed to IRQ 42
        NUMA node: 0
        Region 0: Memory at 4bffe000000 (64-bit, prefetchable) [size=32M]
        Expansion ROM at d3200000 [disabled] [size=1M]
        Capabilities: <access denied>
        Kernel driver in use: mlx5_core
        Kernel modules: mlx5_core
```

网上翻了各种资料发现是设备的温度异常，所以刚开机时网卡是能正常使用，但过几分钟就会异常，系统检测不到pci设备，去网卡口摸了一下光膜，确实温度非常烫，导致的问题

详见：https://community.intel.com/t5/Software-Archive/Unknown-header-type-7f/m-p/1014591