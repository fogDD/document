# ubuntu20.04安装kvm

### 1.检查系统是否支持

```
grep -Eoc '(vmx|svm)' /proc/cpuinfo
sudo apt install cpu-checker
kvm-ok
```

如果未在BIOS中禁用处理器虚拟化功能，则输出将如下所示：

```typescript
INFO: /dev/kvm exists
KVM acceleration can be used
```



### 2.安装kvm

运行以下命令以安装KVM和其他虚拟化管理软件包：

```sql
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
```

- `qemu-kvm` -为KVM管理程序提供硬件仿真的软件。
- `libvirt-daemon-system` -用于将libvirt守护程序作为系统服务运行的配置文件。
- `libvirt-clients` -用于管理虚拟化平台的软件。
- `bridge-utils` -一组用于配置以太网桥的命令行工具。
- `virtinst` -一组用于创建虚拟机的命令行工具。
- `virt-manager` -易于使用的GUI界面和支持命令行工具，用于通过libvirt管理虚拟机。

安装软件包后，libvirt守护程序将自动启动。您可以通过键入以下内容进行验证：

```delphi
sudo systemctl is-active libvirtd
```

输出：

```undefined
active
```

为了能够创建和管理虚拟机，您需要将用户添加到“ libvirt”和“ kvm”组中。为此，请输入：

```swift
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```

`$USER` 是一个环境变量，其中包含当前登录用户的名称。

注销并重新登录，以便刷新组成员身份。



### 3.网络设置

在安装过程中会创建一个名为“ virbr0”的网桥。该设备使用NAT将来宾计算机连接到外界。

您可以使用该`brctl`工具列出当前网桥及其连接的接口：

```dart
brctl show
```

输出：

```bash
bridge name	bridge id		      STP enabled	interfaces
virbr0		  8000.52540089db3f	yes		      virbr0-nic
```

“ virbr0”网桥未添加任何物理接口。“ virbr0-nic”是虚拟设备，没有流量通过该虚拟设备。该设备的唯一目的是避免更改“ virbr0”网桥的MAC地址。

此网络设置适用于大多数Ubuntu桌面用户，但有局限性。如果要从本地网络外部访问来宾，则需要创建一个新的网桥并对其进行配置，以便来宾计算机可以通过主机物理接口连接到外部世界



### 4.启动kvm

```
sudo virt-manager
```