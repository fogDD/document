# Centos7.5部署ftp

### 一、基本安装以及配置文件

在线安装

```
yum install -y vsftpd
```

运行以下命令打开及查看etc/vsftpd

```
[root@zbx-server vsftpd]# ll /etc/vsftpd/
total 20
-rw------- 1 root root  125 Jun 10  2021 ftpusers
-rw------- 1 root root  361 Jun 10  2021 user_list
-rw------- 1 root root 5116 Jun 10  2021 vsftpd.conf
-rwxr--r-- 1 root root  338 Jun 10  2021 vsftpd_conf_migrate.sh
You have new mail in /var/spool/mail/root
[root@zbx-server vsftpd]# 
```

**说明：**
**/etc/vsftpd/vsftpd.conf 是核心配置文件。**
**/etc/vsftpd/ftpusers 是黑名单文件，此文件里的用户不允许访问 FTP 服务器。**
**/etc/vsftpd/user_list  是白名单文件，是允许访问 FTP 服务器的用户列表。**
**/etc/vsftpd/vsftpd_conf_migrate.sh  是vsftpd操作的一些变量和设置**
**备注：使用命令 rpm -ql vsftpd  可列出vsftpd中包含的文件**

启动服务以及开机自启

```
[root@zbx-server vsftpd]# systemctl start vsftpd
[root@zbx-server vsftpd]# 
[root@zbx-server vsftpd]# systemctl enable vsftpd
Created symlink from /etc/systemd/system/multi-user.target.wants/vsftpd.service to /usr/lib/systemd/system/vsftpd.service.
[root@zbx-server vsftpd]# 
```

默认占用21端口

```
[root@zbx-server vsftpd]# netstat -anp | grep vsftp
tcp6       0      0 :::21                   :::*                    LISTEN      95815/vsftpd        
unix  2      [ ]         DGRAM                    2733146  95815/vsftpd         
```



### 二、配置本地用户登录

本地用户登录就是指用户使用 Linux 操作系统中的用户账号和密码登录 FTP 服务器。
vsftpd 安装后默只支持匿名 FTP 登录，用户如果试图使用 Linux 操作系统中的账号登录服务器，将会被 vsftpd 拒绝，但可以在 vsftpd 里配置用户账号和密码登录。具体步骤如下：



#### 1.创建ftp用户

```
[root@zbx-server ~]# useradd ftpuser
[root@zbx-server ~]# passwd ftpuser
Changing password for user ftpuser.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@zbx-server ~]# 
```



#### 2.配置允许本地登录

记得备份配置文件

```
[root@zbx-server vsftpd]# vim /etc/vsftpd/vsftpd.conf
##############################
anonymous_enable=NO
local_enable=YES
pasv_enable=YES
pasv_min_port=8800
pasv_max_port=8899
##############################
[root@zbx-server vsftpd]# systemctl restart vsftpd
```

**c.将是否允许匿名登录 FTP 的参数修改为anonymous enable=NO。**
**d.将是否允许本地用户登录 FTP 的参数修改为local_enable=YES。**

完成vsftpd安装后发现无法远程连接，仍需要完成以下配置。

原因分析：

FTP连接方式分为：主动模式和被动模式。默认为被动模式。

FTP的连接一般是有两个连接的，一个是客户程和服务器传输命令的，另一个是数据传送的连接。FTP服务程序一般会支持两种不同的模式，一种是Port模式，一种是Passive模式(Pasv Mode),我先说说这两种不同模式连接方式的分别。
先假设客户端为C,服务端为S.
Port模式:
当客户端C向服务端S连接后，使用的是Port模式,那么客户端C会发送一条命令告诉服务端S(客户端C在本地打开了一个端口N在等着你进行数据连接),当服务端S收到这个Port命令后 就会向客户端打开的那个端口N进行连接，这种数据连接就生成了。
Pasv模式:

当客户端C向服务端S连接后，服务端S会发信息给客户端C,这个信息是(服务端S在本地打开了一个端口M,你现在去连接我吧),当客户端C收到这个信息后，就可以向服务端S的M端口进行连接,连接成功后，数据连接也建立了。

参考链接：https://blog.csdn.net/binsoft/article/details/44595677

**8800/8899 为上面安全组添加的端口号**
**pasv_enable=YES|NO**
**YES，允许数据传输时使用PASV模式。NO，不允许使用PASV模式。默认值为YES。**
**pasv_min_port=port number** 
**pasv_max_port=port number**

设定在PASV模式下，建立数据传输所可以使用port范围的下界和上界，0 表示任意。默认值为0。把端口范围设在比较高的一段范围内，比如50000-60000，将有助于安全性的提高。



#### 3.测试

```
[root@zbx-server vsftpd]# ftp 192.168.12.110
Connected to 192.168.12.110 (192.168.12.110).
220 (vsFTPd 3.0.2)
Name (192.168.12.110:root): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> pwd
257 "/home/ftpuser"
ftp> 
```

