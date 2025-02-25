# ubuntu



## apt

### Could not get lock /var/lib/dpkg/lock-frontend - open (11: Resource temporarily unavailable) 

这表明当前有某个进程正在apt-get，导致资源被锁不可用。资源被锁的原因可能是上次运行安装或更新时没有正常完成。

您可以使用以下命令删除锁定文件：

```
sudo rm /var/lib/apt/lists/lock
```

您可能还需要删除缓存目录中的锁定文件

```
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock*
```

更新

```
sudo dpkg --configure -a
sudo apt update
```

