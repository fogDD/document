# 安装jenkins



## 一、安装jdk





## 二、安装maven

### 1.下载maven

### https://maven.apache.org/download.cgi

选择需要的版本，下载到linux服务器，我这里下载的是gz



### 2.解压应用，放置到应用目录

```
[root@zbx-server package]# tar -zxvf apache-maven-3.8.6-bin.tar.gz -C /usr/local/

[root@zbx-server package]# ll /usr/local/apache-maven-3.8.6/
total 36
drwxr-xr-x 2 root root    97 Oct 19 17:11 bin
drwxr-xr-x 2 root root    76 Oct 19 17:11 boot
drwxr-xr-x 3 root root    63 Jun  7 00:16 conf
drwxr-xr-x 4 root root  4096 Oct 19 17:11 lib
-rw-r--r-- 1 root root 17568 Jun  7 00:16 LICENSE
-rw-r--r-- 1 root root  5141 Jun  7 00:16 NOTICE
-rw-r--r-- 1 root root  2612 Jun  7 00:16 README.txt
```



### 3.配置系统环境变量

```
[root@zbx-server package]# vim /etc/profile
#######################################
JAVA_HOME=/usr/local/jdk1.8.0_301
JRE_HOME=/usr/local/jdk1.8.0_301/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
MAVEN_PATH=/usr/local/apache-maven-3.8.6                  ####maven的安装路径
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$MAVEN_PATH/bin   ####maven的bin路径
export JAVA_HOME JRE_HOME CLASS_PATH MAVEN_PATH PATH      ####export MAVEN_PATH
#######################################
[root@zbx-server package]# source /etc/profile
[root@zbx-server package]# mvn -version
Apache Maven 3.8.6 (84538c9988a25aec085021c365c560670ad80f63)
Maven home: /usr/local/apache-maven-3.8.6
Java version: 1.8.0_301, vendor: Oracle Corporation, runtime: /usr/local/jdk1.8.0_301/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1127.el7.x86_64", arch: "amd64", family: "unix"
```





## 三、安装jenkins

### 1.下载安装jenkins安装包

这边下载的是rpm包，war包有其他的安装方式

```
[root@zbx-server package]# wget https://repo.huaweicloud.com/jenkins/redhat-stable/jenkins-2.190.3-1.1.noarch.rpm
--2022-10-19 17:57:07--  https://repo.huaweicloud.com/jenkins/redhat-stable/jenkins-2.190.3-1.1.noarch.rpm
Resolving repo.huaweicloud.com (repo.huaweicloud.com)... 59.39.0.156, 59.39.0.153, 59.39.0.154, ...
Connecting to repo.huaweicloud.com (repo.huaweicloud.com)|59.39.0.156|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 78078992 (74M) [application/octet-stream]
Saving to: ‘jenkins-2.190.3-1.1.noarch.rpm’

100%[===============================================================================================================================>] 78,078,992  2.07MB/s   in 31s    

2022-10-19 17:57:38 (2.39 MB/s) - ‘jenkins-2.190.3-1.1.noarch.rpm’ saved [78078992/78078992]
[root@zbx-server package]# rpm -ivh jenkins-2.190.3-1.1.noarch.rpm 
warning: jenkins-2.190.3-1.1.noarch.rpm: Header V4 DSA/SHA1 Signature, key ID d50582e6: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:jenkins-2.190.3-1.1              ################################# [100%]
```



### 2.配置jenkins配置文件

安装完后不能直接启动，需要在jenkins配置文件中配置java文件路径

```
[root@zbx-server package]# ps -ef | grep java
root      30683      1  0 Oct16 ?        00:05:32 /usr/local/jdk1.8.0_301/jre/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/apache-tomcat-8.5.70/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/apache-tomcat-8.5.70/bin/bootstrap.jar:/usr/local/tomcat/apache-tomcat-8.5.70/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat/apache-tomcat-8.5.70 -Dcatalina.home=/usr/local/tomcat/apache-tomcat-8.5.70 -Djava.io.tmpdir=/usr/local/tomcat/apache-tomcat-8.5.70/temp org.apache.catalina.startup.Bootstrap start
[root@zbx-server package]# cp /etc/init.d/jenkins /etc/init.d/jenkins.bak        ##记得备份配置文件
[root@zbx-server package]# vim /etc/init.d/jenkins
####################################################
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/bin/java
/usr/local/jdk1.8.0_301/jre/bin/java        ##填写java文件路径
"i
####################################################
[root@zbx-server package]# systemctl start jenkins
Warning: jenkins.service changed on disk. Run 'systemctl daemon-reload' to reload units.
[root@zbx-server package]# systemctl daemon-reload
[root@zbx-server package]# systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: active (running) since Wed 2022-10-19 18:02:29 CST; 28s ago
     Docs: man:systemd-sysv-generator(8)
   CGroup: /system.slice/jenkins.service
           └─38194 /usr/local/jdk1.8.0_301/jre/bin/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenki...

Oct 19 18:02:27 zbx-server systemd[1]: Starting LSB: Jenkins Automation Server...
Oct 19 18:02:28 zbx-server runuser[38178]: pam_unix(runuser:session): session opened for user jenkins by (uid=0)
Oct 19 18:02:29 zbx-server jenkins[38173]: Starting Jenkins [  OK  ]
Oct 19 18:02:29 zbx-server systemd[1]: Started LSB: Jenkins Automation Server.

############测试是否安装成功############

[root@zbx-server package]# curl localhost:8080
<html><head><meta http-equiv='refresh' content='1;url=/login?from=%2F'/><script>window.location.replace('/login?from=%2F');</script></head><body style='background-color:white; color:white;'>


Authentication required
<!--
You are authenticated as: anonymous
Groups that you are in:
  
Permission you need to have (but didn't): hudson.model.Hudson.Administer
-->

</body></html>                                                                                                                                                                                                                                                                                                            [root@zbx-server package]# 
```

