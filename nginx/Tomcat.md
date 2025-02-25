# Tomcat

### 1.安装jdk8

```
[root@localhost ~]# tar -zxvf jdk-8u411-linux-aarch64.tar.gz
[root@localhost ~]# mkdir -p /opt/java
[root@localhost ~]# tar -zxvf jdk-8u411-linux-aarch64.tar.gz -C /opt/java/
```



### 2.jdk环境变量设置

vim /etc/profile

```
export JAVA_HOME=/opt/java/jdk1.8.0_411
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

验证

```
[root@localhost jdk1.8.0_411]# java -version
java version "1.8.0_411"
Java(TM) SE Runtime Environment (build 1.8.0_411-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.411-b09, mixed mode)
```



### 3.安装tomcat8

```
[root@localhost ~]# wget --no-check-certificate  https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.100/bin/apache-tomcat-8.5.100.tar.gz
[root@localhost ~]# tar -zxvf apache-tomcat-8.5.100.tar.gz -C /opt/tomcat/
[root@localhost apache-tomcat-8.5.100]# cd /opt/tomcat/apache-tomcat-8.5.100/bin/
[root@localhost bin]# ./startup.sh 
```

验证

```
[root@localhost bin]# curl localhost:8080




<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Apache Tomcat/8.5.100</title>
        <link href="favicon.ico" rel="icon" type="image/x-icon" />
        <link href="tomcat.css" rel="stylesheet" type="text/css" />
    </head>

    <body>
        <div id="wrapper">
            <div id="navigation" class="curved container">
                <span id="nav-home"><a href="https://tomcat.apache.org/">Home</a></span>
                <span id="nav-hosts"><a href="/docs/">Documentation</a></span>
                <span id="nav-config"><a href="/docs/config/">Configuration</a></span>
                <span id="nav-examples"><a href="/examples/">Examples</a></span>
                <span id="nav-wiki"><a href="https://cwiki.apache.org/confluence/display/TOMCAT/">Wiki</a></span>
                <span id="nav-lists"><a href="https://tomcat.apache.org/lists.html">Mailing Lists</a></span>
                <span id="nav-help"><a href="https://tomcat.apache.org/findhelp.html">Find Help</a></span>
                <br class="separator" />
            </div>
            <div id="asf-box">
                <h1>Apache Tomcat/8.5.100</h1>
            </div>
            <div id="upper" class="curved container">
                <div id="congrats" class="curved container">
                    <h2>If you're seeing this, you've successfully installed Tomcat. Congratulations!</h2>
```

看到tomcat默认地址成功，如果在其他设备无法访问，记得检查防火墙
