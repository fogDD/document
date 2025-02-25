# Centos7升级驱动

### 1.卸载旧驱动

```
sudo /usr/bin/nvidia-uninstall
```



### 2.屏蔽nouveau

#### 1.检查是否屏蔽掉了默认的nouveau

如果有内容则表示没有屏蔽

```
lsmod | grep nouveau
```

#### 2.屏蔽掉了默认的nouveau

```
vim /lib/modprobe.d/dist-blacklist.conf
 
#将nvidiafb注释掉:
#blacklist nvidiafb 
 
 
#然后添加以下语句：
blacklist nouveau
options nouveau modeset=0
```

####  3.重建initramfs image步骤

```bash
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
dracut /boot/initramfs-$(uname -r).img $(uname -r)
```

#### 4.修改运行级别为文本模式

```
systemctl set-default multi-user.target
```

#### 5.重启

```
reboot
```



### 3.安装依赖

#### 1.安装kernel-devel

安装当前内核版本的kernel-devel

```
yum install -y "kernel-devel-uname-r == $(uname -r)"
```

否则会提示-> Unable to determine if Secure Boot is enabled: No such file or directory
ERROR: Unable to load the kernel module 'nvidia.ko'.  This happens most frequently when this kernel module was built against the wrong or improperly configured kernel sources, with a version of gcc that differs from the one used to build the target kernel, or if another driver, such as nouveau, is present and prevents the NVIDIA kernel module from obtaining ownership of the NVIDIA GPU(s), or no NVIDIA GPU installed in this system is supported by this NVIDIA Linux graphics driver release.

#### 2.安装gcc

```
yum -y install gcc dkms
```



### 4.安装驱动脚本

将下载好的驱动脚本传输到服务器中

```
./NVIDIA-Linux-x86_64-440.64.00.run
```

如果报错Unable to find the kernel source tree for the currently running kernel.Please make sure you have installed the kernel source files for your kernel and that they are properly configured;on Red Hat

Linux systems,for example,be sure you have the 'kernel-source'or 'kernel-devel'RPM installed.If you know the correct kernel source files are installed,you may specify the kernel source path with the '--kernel-source-path'command line option.



则需要加入内核版本号

```
./NVIDIA-Linux-x86_64-375.39.run --kernel-source-path=/usr/src/kernels/3.10.0-693.el7.x86_64
```

**注意：这里填你当前的内核版本号**

然后重启后女 nvidia-smi后有内容即安装成功



### 遇到的问题

如果遇到You appear to be running an X server； please exit X before installing的报错，需要检查是否安装了远程软件，比如vnc

，如果安装了VNC则需要关闭VNC服务，参考

```
systemctl stop ****
```

或者

```
ps aux|grep vnc
```

