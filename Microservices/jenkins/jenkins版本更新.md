# jenkins版本更新



## 一、更新jdk11（可选）

Oct 21 11:29:59 zbx-server jenkins[63447]: Jenkins requires Java versions [17, 11] but you are running with Java 1.8 from /usr/local/jdk1.8.0_301/jre

因2.361版本以上的jenkins需要jdk11或者17



### 1.删除旧版本jdk

我这边旧版本的jdk是用tar装的，安装在/usr/local/，所以直接删除目录即可

```
[root@zbx-server]# rm -rf /usr/local/jdk1.8.0_301/
```

### 2.安装jdk11

我这边用的是tar包的形式安装

```
[root@zbx-server package]# tar -zxvf jdk-11.0.17_linux-x64_bin.tar.gz -C /usr/local/
[root@zbx-server package]# cp /etc/profile /etc/profile.bak        ##记得备份配置文件

####################################
JAVA_HOME=/usr/local/jdk-11.0.17               ##java目录
JRE_HOME=/usr/local/jdk-11.0.17/lib            ##jre目录
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
MAVEN_PATH=/usr/local/apache-maven-3.8.6
JENKINS_HOME=/usr/local/jenkins
PATH=$PATH:$JAVA_HOME/bin:$MAVEN_PATH/bin
export JAVA_HOME JRE_HOME CLASS_PATH MAVEN_PATH JENKINS_HOME PATH
####################################

[root@zbx-server local]# source /etc/profile
[root@zbx-server local]# java -version
java version "11.0.17" 2022-10-18 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.17+10-LTS-269)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.17+10-LTS-269, mixed mode)
```



## 二、更新jenkins

### 1.替换war包版本

```
[root@zbx-server ~]#cp /usr/lib/jenkins/jenkins.war /usr/lib/jenkins/jenkins.war.bak        ##记得备份原来的war包
```

关闭jenkins服务

```
[root@zbx-server ~]# ps -ef | grep jenkins
root      65885      1  0 12:04 ?        00:00:00 runuser -s /bin/bash jenkins -c ulimit -S -c 0 >/dev/null 2>&1 ; /usr/local/jdk-11.0.17/bin/java -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
jenkins   65886  65885  0 12:04 ?        00:00:00 bash -c ulimit -S -c 0 >/dev/null 2>&1 ; /usr/local/jdk-11.0.17/bin/java -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
jenkins   65887  65886  0 12:04 ?        00:02:04 /usr/local/jdk-11.0.17/bin/java -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
root      81778  80479  0 16:06 pts/0    00:00:00 grep --color=auto jenkins
[root@zbx-server ~]# kill -9 ****
```

```
[root@zbx-server ~]#systemctl stop jenkins
```

替换war文件

```
[root@zbx-server ~]#cp ~/package/jenkins.war /usr/lib/jenkins.war
```

启动jenkins服务

```
[root@zbx-server ~]#systemctl start jenkins
```

