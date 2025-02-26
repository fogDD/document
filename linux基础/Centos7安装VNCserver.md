### Centos7安装VNCserver

### 1.安装服务

```
yum install tigervnc-server
```

配置vncservers文件

```
vim /etc/sysconfig/vncservers 
# THIS FILE HAS BEEN REPLACED BY /lib/systemd/system/vncserver@.service
VNCSERVERS="1:root"          #设置登录用户，可设置多个用户，这里的数值需与下列数值匹配
VNCSERVERARGS[1]="-geometry 1024*768"  #设置vnc桌面的分辨率（这个可以不设置）
```

拷贝启动文件

```
cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@\:1.service
```

修改配置文件

```
[root@vncserver ~]# vim /etc/systemd/system/vncserver@\:1.service
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
User=root

# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=-/usr/bin/vncserver -kill %i
ExecStart=/usr/bin/vncserver %i
PIDFile=/root/.vnc/%H%i.pid
ExecStop=-/usr/bin/vncserver -kill %i

[Install]
WantedBy=multi-user.target
```

### 2.修改vnc密码

```
vncpasswd
```

### 3.启动服务

```
systemctl daemon-reload       #需先执行刷新服务
systemctl start vncserver@:1.service  #启动vncserver服务，或者执行 vncserver :1
systemctl enable vncserver@:1.service  #将vncserver桌面1加入开机自启动项
```

关闭防火墙

```
[root@vncserver ~]# systemctl stop firewalld.service
[root@vncserver ~]# systemctl status firewalld.service 
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)

```

