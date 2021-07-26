# rsync+crontab简单的定时备份案例

案例，客户端每1分钟备份backup_test目录下的文件到备份服务器的backup_test目录下

192.168.12.110--客户端

192.168.12.111--备份服务器（服务端）

#### 1.安装rsync

客户端和备份服务器（服务端）都需要

```
ps -ef | grep rsync
yum -y install rsync
```



#### 2.修改rsync配置文件（备份服务器（服务端））

```
# /etc/rsyncd: configuration file for rsync daemon mode

# See rsyncd.conf man page for more options.

# configuration example:


uid = rsync
gid = rsync
use chroot = no
max connections = 200
timeout = 300
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log

[backup_test]
path = /tmp/backup_test
ignore errors
read only = false
list = false
hosts allow = 192.168.12.0/24
hosts deny = 0.0.0.0/32
auth users = rsync_backup
secrets file = /etc/rsync.password

```

以下是对rsync配置文件的一些解释

```
######### 全局配置参数 ##########
port=888    # 指定rsync端口。默认873
uid = rsync # rsync服务的运行用户，默认是nobody，文件传输成功后属主将是这个uid
gid = rsync # rsync服务的运行组，默认是nobody，文件传输成功后属组将是这个gid
use chroot = no # rsync daemon在传输前是否切换到指定的path目录下，并将其监禁在内
max connections = 200 # 指定最大连接数量，0表示没有限制
timeout = 300         # 确保rsync服务器不会永远等待一个崩溃的客户端，0表示永远等待
motd file = /var/rsyncd/rsync.motd   # 客户端连接过来显示的消息
pid file = /var/run/rsyncd.pid       # 指定rsync daemon的pid文件
lock file = /var/run/rsync.lock      # 指定锁文件
log file = /var/log/rsyncd.log       # 指定rsync的日志文件，而不把日志发送给syslog
dont compress = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2  # 指定哪些文件不用进行压缩传输
 
###########下面指定模块，并设定模块配置参数，可以创建多个模块###########
[longshuai]        # 模块ID
path = /longshuai/ # 指定该模块的路径，该参数必须指定。启动rsync服务前该目录必须存在。rsync请求访问模块本质就是访问该路径。
ignore errors      # 忽略某些IO错误信息
read only = false  # 指定该模块是否可读写，即能否上传文件，false表示可读写，true表示可读不可写。所有模块默认不可上传
write only = false # 指定该模式是否支持下载，设置为true表示客户端不能下载。所有模块默认可下载
list = false       # 客户端请求显示模块列表时，该模块是否显示出来，设置为false则该模块为隐藏模块。默认true
hosts allow = 10.0.0.0/24 # 指定允许连接到该模块的机器，多个ip用空格隔开或者设置区间
hosts deny = 0.0.0.0/32   # 指定不允许连接到该模块的机器
auth users = rsync_backup # 指定连接到该模块的用户列表，只有列表里的用户才能连接到模块，用户名和对应密码保存在secrts file中，
                          # 这里使用的不是系统用户，而是虚拟用户。不设置时，默认所有用户都能连接，但使用的是匿名连接
secrets file = /etc/rsyncd.passwd # 保存auth users用户列表的用户名和密码，每行包含一个username:passwd。由于"strict modes"
                                  # 默认为true，所以此文件要求非rsync daemon用户不可读写。只有启用了auth users该选项才有效。
[xiaofang]    # 以下定义的是第二个模块
path=/xiaofang/
read only = false
ignore errors
comment = anyone can access
```



#### 3.在服务端给需要备份的目录设置权限，创建rsync用户

```
mkdir -p /tmp/backup_test //创建需要备份的目录
useradd rsync	//创建rsync用户
chown -R rsync:rsync /tmp/backup_test  //为备份目录授权
rsync --daemon	//启动服务
lsof -i:873
netstat | grep 873
ps -ef | grep rsync   //检查服务是否启动
```

附加：停止rsync

```
ps -ef | grep rsync
kill -9 进程号
rm -rf /var/rsync/rsync.pid
```



#### 4.在客户端给需要备份的目录设置权限，创建rsync用户

```
mkdir -p /tmp/backup_test //创建需要备份的目录
useradd rsync	//创建rsync用户
```



#### 5.为服务端创建密码文件

```
echo "rsync_backup:123456" >>/etc/rsync.password  //创建密码文件，注：rsync_backup、/etc/rsync.password必须与配置文件内容一致（建议剪贴信息），123456为密码
```

为密码文件设置权限

```
chmod 600 /etc/rsync.password   //为密码文件设置权限为600
```

auth users和secrets file这两行不是一定需要的，省略它们时将默认使用匿名连接。但是如果使用了它们，则secrets file的权限必须是600。客户端的密码文件也必须是600。



#### 6.为客户端创建密码文件

```
echo "123456" >>/etc/rsync.password  //创建密码文件，注：rsync_backup、/etc/rsync.password必须与配置文件内容一致（建议剪贴信息），123456为密码
这里只需要密码
```

为密码文件设置权限

```
chmod 600 /etc/rsync.password   //为密码文件设置权限为600
```

auth users和secrets file这两行不是一定需要的，省略它们时将默认使用匿名连接。但是如果使用了它们，则secrets file的权限必须是600。客户端的密码文件也必须是600



#### 7.在客户端需要备份的目录随便创建文件测试

```
cd /tmp/backup_test
touch test2
```



#### 8.用crontab添加定时任务

在客户端添加定时任务，每一分钟从客户端的/tmp/backup_tmp/目录下的文件备份到服务端的/tmp/backup_tmp/目录下

直接编辑/etc/crontab文件，添加一条定时任务即可

```
vim /etc/crontab
#########以下是crontab文件内##############
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
* * * * * root rsync -avz /tmp/backup_test/ rsync_backup@192.168.12.111::backup_test --password-file=/etc/rsync.password

```

此外，使用crontab -e命令也可以直接配置定时任务，但与vi /etc/crontab不同，不同点如下：
1./etc/crontab中的为系统任务，只有root可以设定，而crontab -e设置的定时任务为用户任务，设定完成后会将任务自动写入/var/spool/cron/usename文件。
2./etc/crontab中的任务需要指定用户名，crontab -e不需要。



#### 9.在备份服务器（服务端）查看备份过来的文件

```
ll /tmp/backup
total 20
-rwxrwxrwx. 1 rsync rsync 3 Jul 22 04:38 test

```



#### 10.error

rsync error: some files/attrs were not transferred (see previous errors) (code 23) at main.c(1518) [generator=3.0.9]

原因是有文件没有写的权限，导致备份数据库权限不够，两种解决办法：

1、将服务端rsyncd.conf配置文件的uid和gid分别修改成root，重载下，/etc/rc.d/init.d/xinetd reload，再次执行同步，同步成功
2、将需要同步的文件夹及下属文件赋予777权限（chmod -R 777 xxx），再次执行同步，同步成功

注意：如果使用第一种办法，那么在执行完同步后，为了安全，记得将uid和gid修改回来，或修改成nobody
————————————————
版权声明：本文为CSDN博主「slovyz」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/slovyz/article/details/38924967
