# Ubuntu20.04安装ssh服务

### 1.安装ssh服务

```
sudo apt-get install openssh-server
```

安装openssh-server,如果顺利的话会安装成功,如果遇到以下问题：

openssh-server : Depends: openssh-client (= 1:7.2p2-4ubuntu2.1)
Depends: openssh-sftp-server but it is not going to be installed
Recommends: ncurses-term but it is not installable
Recommends: ssh-import-id but it is not installable
E: Unable to correct problems, you have held broken packages.

原因如下：

这是因为,openssh-server是依赖于openssh-clien的,那ubuntu不是自带了openssh-client吗?

原由是自带的openssh-clien与所要安装的openssh-server所依赖的版本不同,这里所依赖的版本是：

1:7.2p2-4ubuntu2.1

所以要安装对应版本的openssh-clien,来覆盖掉ubuntu自带的

```
sudo apt-get install openssh-client=1:7.2p2-4ubuntu2.1 
```

​     ## 这里需要安装你刚系统提示的对应openssh-client版本

这样再安装openssh-server就可以成功了。



### 2.安装ssh-server

```
sudo apt-get install openssh-server
```



### 3.检查启动服务

上述对应服务都安装好后可直接查看端口占用

```
ps -ef | grep ssh
```

