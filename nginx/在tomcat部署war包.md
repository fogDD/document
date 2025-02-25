## 在tomcat部署war包

### 一、安装tomcat

#### 1.安装jdk

```
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
```

![image-20210827090757282](C:\Users\h1172\AppData\Roaming\Typora\typora-user-images\image-20210827090757282.png)

#### 2.解压

```
tar -zxvf jdk-8u301-linux-x64.tar.gz
```

#### 3.修改linux系统环境变量文件

先创建存放jdk的目录

```
mkdir /usr/local/java
mv mv 1.8.0_301 /usr/local/java/
```

编辑环境变量文件，在最底下添加

```
export JAVA_HOME=/usr/local/java/jdk1.8.0_301
export CLASSPATH=.:$JAVA_HOME/lib/dt.tar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

source一下脚本

```
source /etc/profile
```

测试jdk是否安装成功

```
java -version 
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
```

### 二、安装tomcat

#### 1.官网下载

![image-20210827091813443](C:\Users\h1172\AppData\Roaming\Typora\typora-user-images\image-20210827091813443.png)

解压

```
tar -zxvf apache-tomcat-8.5.70.tar.gz
```

创建存放tomcat的目录

```
mkdir -p /usr/local/tomcat
mv apache-tomcat-8.5.70 /usr/local/tomcat/
```

#### 2.启动tamcat

进入/bin目录启动tomcat服务

```
cd /usr/local/tomcat/apache-tomcat-8.5.70/bin
./startup.sh
Using CATALINA_BASE:   /usr/local/tomcat/apache-tomcat-8.5.70
Using CATALINA_HOME:   /usr/local/tomcat/apache-tomcat-8.5.70
Using CATALINA_TMPDIR: /usr/local/tomcat/apache-tomcat-8.5.70/temp
Using JRE_HOME:        /usr/local/java/jdk1.8.0_301
Using CLASSPATH:       /usr/local/tomcat/apache-tomcat-8.5.70/bin/bootstrap.jar:/usr/local/tomcat/apache-tomcat-8.5.70/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
```

这代表启动成功

### 三、部署war包

#### 1.把war包放到webapps文件夹下

```
cp ***.war /usr/local/tomcat/apache-tomcat-8.5.70/webapps
```

#### 2.设置直接访问tomcat网址即可显示web页面

如果不想改server.xml，那么更直接的是把war包解压到ROOT目录下，或者把wlf.war放到webapps目录下并改名为ROOT.war，再删掉ROOT目录重启tomcat即可。

```
cp ***.war /usr/local/tomcat/apache-tomcat-8.5.70/webapps
mv ***.war ROOT.war
```

```
cp ***.war /usr/local/tomcat/apache-tomcat-8.5.70/webapps/ROOT
```

