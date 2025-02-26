## 常用记忆知识

[TOC]

**使用终端ssh登录Linux操作系统的控制台后，会出现一个提示符号（例如：#或~），在这个提示符号之后可以输入命令，Linux根据输入的命令会做回应，这一连串的动作是由一个所谓的Shell]来做处理。Shell是一个程序，最常用的就是Bash，这也是登录系统默认会使用的Shell。**

**用户HOME（家）目录/.bashrc**

```
vim ~/.bashrc
```

1. bashrc是在系统启动后就会自动运行。
2. profile是在用户登录后才会运行



**用户HOME（家）目录/.profile**

```
vim ~/.profile
```



## linux系统错误

### Linux文件系统只读Read-only file system

问题描述:

1、系统无法进行磁盘的读写操作（touch,cp,chmod）等等

2、服务器无法启动（也是因为无法创建文件）

3、只有涉及到系统磁盘的写操作，都会报错"Read-only file system"

问题原因：

1、系统没有正常关机，导致虚拟磁盘出现文件系统错误；

2、机器硬盘故障导致硬盘只读

一般情况是由于系统发现磁盘硬件（Riad卡，硬盘）故障或文件系统中文件被损坏后而采取的保护机制导致的。为了保护数据不破坏分区中已有内容，Linux在挂载文件系统是就只用read-only只读方式加载。

问题解决：

```
mount -o  remount rw /
```

-o remount：将一个已经挂下的档案系统重新用不同的方式挂上。例如原先是唯读的系统，现在用可读写的模式重新挂上





### Linux系统网卡无法启动 No suitable device found for this connection

Error:Connection activation failed: No suitable device found for this connection

在使用VMware workstation克隆[CentOS7](https://so.csdn.net/so/search?q=CentOS7&spm=1001.2101.3001.7020).X系列系统的虚拟机时，不管是链接克隆还是完整克隆，极可能出现下面网卡无法启动问题：

注：目前亲测VMware workstation 15不存在这种问题，在文章后面我会解释为什么不会以及我写这篇文章的目的。这个问题出现的原因是：MAC地址相同，导致冲突

克隆[虚拟机](https://so.csdn.net/so/search?q=虚拟机&spm=1001.2101.3001.7020)，默认它的网卡MAC地址依然是以前模板机的MAC，这样MAC地址就会冲突，当系统使用NetworkManager来管理网卡时

NetworkManager就不允许MAC地址相同，就导致网卡直接起不来，但是CentOS6.X就不会，因为它用的是network来管理。
解决方案：
1、如果不打算用NetworkManager，那么就直接关闭NetworkManager即可

```bash
1 systemctl stop NetworkManager      # 停止NetworkManager   
2 systemctl mask NetworkManager　　　 # 禁用NetworkManager，类似Windows的禁用
3 systemctl disable NetworkManager   # 开机不启动
```

2、更改MAC地址

1）通过VMware workstation 的虚拟机管理界面，直接删除以前的网卡，重新添加，或者点击高级，修改MAC地址和模板机不一样





## vim

### 1.批量注释

1、在 10 - 20 行添加 // 注释

```javascript
:10,20s#^#//#g
```

2、在 10 - 20 行删除 // 注释

```javascript
:10,20s#^//##g
```

3、在 10 - 20 行添加 # 注释

```javascript
:10,20s/^/#/g
```

4、在 10 - 20 行删除 # 注释

```javascript
:10,20s/#//g
```

### 2.批量删除

只删除该字符串，而不是删除它出现过的行

如，删除`ad123`，vi编辑器在命令模式下输入：

```
:%s/ad123//g
```



## rpm

### 1.--help

--force 即使覆盖属于其它包的文件也强迫安装
--nodeps 如果该RPM包的安装依赖其它包，即使其它包没装，也强迫安装





## curl

### 1.-L

有的网址是自动跳转的。使用 `-L` 参数，curl 就会跳转到新的网址

```crystal
$ curl -L www.sina.com
```

键入上面的命令，结果自动跳转为 [www.sina.com.cn](https://link.jianshu.com/?t=http://www.sina.com.cn)。



### 2.curl -fsSL

下载github中的项目文件时经常会用 curl -fsSL https://github.com/xxxxx ,请问一下这些参数总共要表达什么？

-f (--fail) 表示在服务器错误时，阻止一个返回的表示错误原因的 html 页面，而由 curl 命令返回一个错误码 22 来提示错误
-L,--location:如果服务器报告请求的页面已移动到其他位置（用location:header和3xx 响应代码），此选项将使curl在新位置上重新执行请求。
--show-error：当与-s，--silent一起使用时，它会使curl在失败时显示错误消息。
-s，--silent：安静模式。不显示进度表或错误信息。使curl静音。它仍然会输出您请求的数据，甚至可能输出到终端stdout，除非您对它进行重定向。





## tar

### 1.--strip-comonents=1

–strip-component=1 代表解压出来的文件，剥离前一个路径

–strip-component=2 代表解压出来的文件，剥离前两个路径



## docker

### 1.批量删除镜像或容器

```
docker rmi -f `docker images | grep nvcr | awk '{print $3}'
```

其中 grep nvcr就是你需要删除的关键字

### 2.删除所有容器

```
docker rm `docker ps -a -q`
```

### 3.删除所有镜像

```
docker rmi `docker images -q`
```