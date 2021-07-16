## docker的介绍

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的[镜像](https://baike.baidu.com/item/%E9%95%9C%E5%83%8F/1574)中，然后发布到任何流行的[Linux](https://baike.baidu.com/item/Linux)或[Windows](https://baike.baidu.com/item/Windows/165458)机器上，也可以实现[虚拟化](https://baike.baidu.com/item/%E8%99%9A%E6%8B%9F%E5%8C%96/547949)。容器是完全使用[沙箱](https://baike.baidu.com/item/%E6%B2%99%E7%AE%B1/393318)机制，相互之间不会有任何接口。

## docker安装

### 移除旧的版本

```plain
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
### 指定版本安装

#### 安装依赖

```plain
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
#### 添加软件源信息

```plain
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
#### 更新 yum 缓存

```plain
sudo yum makecache fast
```
#### 查看可用的Docker-CE版本

```plain
yum list docker-ce --showduplicates | sort -r
```
#### 安装指定版本的docker-ce,默认是安装最新版本

```plain
sudo yum install -y docker-ce
```
#### 配置阿里云仓库镜像加速

```plain
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://4dcje9sc.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload&&
sudo systemctl restart docker
```
## Docker的核心概念

docker的三大核心概念

* 镜像（image）

Docker镜像类似于虚拟机镜像，可以将它理解为一个只读的模板。例如，一个镜像可以包含一个基本的操作系统环境，里面仅安装了Apache应用程序（或用户需要的其他软件）。可以把它称为一个Apache镜像。镜像是创建Docker容器的基础。通过版本管理和增量的文件系统，Docker提供了一套十分简单的机制来创建和更新现有的镜像，用户甚至可以从网上下载一个已经做好的应用镜像，并直接使用。

* 容器（container）

Docker容器类似于一个轻量级的沙箱，Docker利用容器来运行和隔离应用。容器是从镜像创建的应用运行实例。它可以启动、开始、停止、删除，而这些容器都是彼此相互隔离、互不可见的。可以把容器看作一个简易版的Linux系统环境（包括root用户权限、进程空间、用户空间和网络空间等）以及运行在其中的应用程序打包而成的盒子。

* 仓库（Repository）

Docker仓库类似于代码仓库，是Docker集中存放镜像文件的场所。有时候我们会将Docker仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器是存放仓库的地方，其上往往存放着多个仓库。每个仓库集中存放某一类镜像，往往包括多个镜像文件，通过不同的标签（tag）来进行区分。例如存放Ubuntu操作系统镜像的仓库，被称为Ubuntu仓库，其中可能包括16.04、18.04等不同版本的镜像。

## docker镜像的使用

### 获取镜像

格式如下：

docker  pull [镜像名]:TAG

其中，NAME是镜像仓库名称（用来区分镜像）, TAG是镜像的标签（往往用来表示版本信息）。通常情况下，描述一个镜像需要包括“名称+标签”信息。

例如,获取一个nginx版本为1.14.2的镜像:

```plain
docker pull nginx:1.14.2
```
对于Docker镜像来说，如果不显式指定TAG，则默认会选择latest标签，这会下载仓库中最新版本的镜像。
### 查看镜像信息

#### images命令列出镜像

使用`docker images`命令列出本地主机上的docker镜像信息如下:

```plain
[root@womr-c ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ubuntu       18.04     2c047404e52d   2 months ago    63.3MB
nginx        1.14.2    295c7be07902   22 months ago   109MB
```
* REPOSITORY 来自那个仓库
* TAG 镜像的标签信息
* IMAGE ID 镜像的ID（唯一标识镜像）
* CREATED 创建时间
* SIZE 镜像大小

**更多参数如下：**

```plain
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs
```
### tag命令添加镜像标签

为了方便在后续工作中使用特定镜像，还可以使用docker tag命令来为本地镜像任意添加新的标签。例如，添加一个新的myubuntu:latest镜像标签：

```plain
[root@womr-c ~]# docker tag nginx:1.14.2 mynginx:latest
[root@womr-c ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ubuntu       18.04     2c047404e52d   2 months ago    63.3MB
mynginx      latest    295c7be07902   22 months ago   109MB
nginx        1.14.2    295c7be07902   22 months ago   109MB
```
### 使用inspect命令查看详细信息

获取该镜像的详细信息，作者信息，适应的架构，各层的数字摘要等：

```plain
docker inspect mynginx:latest 
```
### history命令查看镜像历史

例如，查看nginx:1.14.2镜像的创建过程如下

```plain
[root@womr-c ~]# docker history nginx:1.14.2 
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
295c7be07902   22 months ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B        
<missing>      22 months ago   /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B        
<missing>      22 months ago   /bin/sh -c #(nop)  EXPOSE 80                    0B        
<missing>      22 months ago   /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B       
<missing>      22 months ago   /bin/sh -c set -x  && apt-get update  && apt…   53.8MB    
<missing>      22 months ago   /bin/sh -c #(nop)  ENV NJS_VERSION=1.14.2.0.…   0B        
<missing>      22 months ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.14.2-…   0B        
<missing>      22 months ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B        
<missing>      22 months ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      22 months ago   /bin/sh -c #(nop) ADD file:4fc310c0cb879c876…   55.3MB
```
### 搜寻镜像

例如，搜寻官方提供的带nginx关键字的镜像，如下所示：

```plain
[root@womr-c ~]# docker search --filter=is-official=true nginx
NAME      DESCRIPTION                STARS     OFFICIAL   AUTOMATED
nginx     Official build of Nginx.   14356     [OK]      
```
例如，搜索所有收藏数超过4的关键字包括mysql的镜像
```plain
[root@womr-c ~]# docker search --filter=stars=4 mysql
NAME                         DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                        MySQL is a widely used, open-source relation…   10428     [OK]       
mariadb                      MariaDB is a community-developed fork of MyS…   3870      [OK]
```
### 删除和清理镜像

#### 标签删除镜像

使用docker rmi或docker image rm命令可以删除镜像，命令格式为docker rmiIMAGE [IMAGE...]，其中IMAGE可以为标签或ID。

* -f, -force：强制删除镜像，即使有容器依赖它；
* -no-prune：不要清理未带标签的父镜像。

例如，要删除掉myubuntu:latest镜像，而nginx:1.14.2则不会被删除。可以使用如下命令:

```plain
[root@womr-c ~]# docker rmi mynginx:latest 
Untagged: mynginx:latest
```
#### 镜像ID删除镜像

```plain
[root@womr-c ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ubuntu       18.04     2c047404e52d   2 months ago    63.3MB
nginx        1.14.2    295c7be07902   22 months ago   109MB
[root@womr-c ~]# docker rmi -f 2c047404e52d
Untagged: ubuntu:18.04
Untagged: ubuntu@sha256:fd25e706f3dea2a5ff705dbc3353cf37f08307798f3e360a13e9385840f73fb3
Deleted: sha256:2c047404e52d7f17bdac4121a13cd844447b74e13063f8cb8f8b314467feed06
```
#### 清理镜像

使用Docker一段时间后，系统中可能会遗留一些临时的镜像文件，以及一些没有被使用的镜像，可以通过docker image prune命令来进行清理。

* -a, -all：删除所有无用镜像，不光是临时镜像；
* -filter filter：只清理符合给定过滤器的镜像；
* -f, -force：强制删除镜像，而不进行提示确认

例如自动清理临时遗留镜像文件层，最后会提示释放的存储空间

```plain
[root@womr-c ~]# docker image prune -f
Total reclaimed space: 0B
```
### 创建镜像

创建镜像的方法主要有三种：基于已有镜像的容器创建、基于本地模板导入、基于Dockerfile创建。

#### 基于已有容器创建

例如，启动一个镜像，并在其中进行修改操作。例如，创建一个test文件，之后退出，代码如下：

```plain
[root@womr-c ~]# docker run -it ubuntu:18.04 /bin/bash
root@d6b569374955:/# to 
toe    top    touch  
root@d6b569374955:/# touch test
root@d6b569374955:/# exit
[root@womr-c ~]# docker commit -m 'add a new file' -a 'worm-c' d6 test:0.1
sha256:e631dedb401061613cf759f80b6b62d83576f4c8b21511c5c3b5b24c7948e107
[root@womr-c ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
test         0.1       e631dedb4010   3 seconds ago   63.3MB
ubuntu       18.04     2c047404e52d   2 months ago    63.3MB
nginx        1.14.2    295c7be07902   22 months ago   109MB
```
#### 文件模板导入

例如，要直接导入一个镜像，可以使用OpenVZ提供的模板来创建，或者用其他已导出的镜像模板来创建。OPENVZ模板的下载地址为[http://openvz.org/Download/templates/precreated](http://openvz.org/Download/templates/precreated)

```plain
[root@womr-c ~]# cat centos-7-x86_64-minimal-20170709.tar.xz |docker import - centos:7.0
sha256:1a1cbfa97aa871e243a92dc8fcc16d8df1f421e1443c210416698bb44fb3bd33
[root@womr-c ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
centos       7.0       1a1cbfa97aa8   4 seconds ago   387MB
test         0.1       e631dedb4010   7 minutes ago   63.3MB
ubuntu       18.04     2c047404e52d   2 months ago    63.3MB
nginx        1.14.2    295c7be07902   22 months ago   109MB
```
### Dockerfile创建

创建Dockerfile

```plain
FROM debian:stretch-slim
LABEL version="1.0" maintainer="docker user <docker_user@github>"
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```
执行如下命令
```plain
docker build -t pthon:3 .
Removing intermediate container 955940f9c9aa
 ---> 5db817c293e5
Successfully built 5db817c293e5
Successfully tagged python3:latest
```
### 导出和导入镜像

#### 导出镜像

例如导出nginx1.14.2镜像为文件nginx.tar

```plain
docker save -o nginx.tar nginx:1.14.2
```
#### 导入镜像

例如将nginx.tar导入镜像

```plain
[root@womr-c ~]# docker load -i nginx.tar 
#也可 docker load <  nginx.tar
Loaded image: nginx:1.14.2
```
## docker操作容器

### 创建容器

可以使用docker create命令新建一个容器，例如：

```plain
[root@womr-c ~]# docker create -it nginx:1.14.2 
b9ff064ef24be4a6a2651b888b090e08a6c0866e54597e8d9fd3fe5087d13355
[root@womr-c ~]# docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS    PORTS     NAMES
b9ff064ef24b   nginx:1.14.2   "nginx -g 'daemon of…"   20 seconds ago   Created             exciting_ramanujan
[root@womr-c ~]# docker restart b9
b9
```
### 守护进程

更多的时候，需要让Docker容器在后台以守护态（Daemonized）形式运行。此时，可以通过添加-d参数来实现

```plain
[root@womr-c ~]# docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo helloworld;sleep 1;done"
6011437625d867a83d8ccd062f09fccb2b53ed8ad33e12b0716dd0a04edb8742
[root@womr-c ~]# docker logs -f 60
helloworld
helloworld
helloworld
helloworld
helloworld
helloworld
helloworld
helloworld
helloworld
helloworld
^C
```
### 停止容器

如下：

```plain
[root@womr-c ~]# docker stop 60
60
```
### 进入容器

如下：

```plain
[root@womr-c ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
b9ff064ef24b   nginx:1.14.2   "nginx -g 'daemon of…"   8 minutes ago   Up 8 minutes   80/tcp    exciting_ramanujan
[root@womr-c ~]# docker exec -it b9 /bin/bash
root@b9ff064ef24b:/# curl
```
## docker数据持久化

* 数据卷（Data Volumes）：容器内数据直接映射到本地主机环境；
* 数据卷容器（Data Volume Containers）：使用特定容器维护数据卷。

数据卷（Data Volumes）是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于Linux中的mount行为。

### 创建数据卷

```plain
[root@womr-c ~]# docker volume create -d local test
test
[root@womr-c ~]# cd /var/lib/docker/volumes
```
除了create子命令外，docker volume还支持inspect（查看详细信息）、ls（列出已有数据卷）、prune（清理无用数据卷）、rm（删除数据卷）等,例如：
```plain
Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove all unused local volumes
  rm          Remove one or more volumes
```
### 绑定数据卷

```plain
[root@womr-c www]# docker run --rm -d -p 8081:80 --name nginx-test-web \
  -v /home/nginx/www:/usr/share/nginx/html \
  nginx:1.14.2
[root@womr-c www]# docker restart nginx-test-web
[root@womr-c www]# curl localhost:8081
<h1>hello nginx</h1> 
```
### 数据卷容器

如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。数据卷容器也是一个容器，但是它的目的是专门提供数据卷给其他容器挂载。

首先，创建一个数据卷容器dbdata，并在其中创建一个数据卷挂载到/dbdata：

```plain
docker run -it -v /dbdata --name dbdata ubuntu:18.04
243  docker restart  dbdata
244  docker run -it --volumes-from dbdata --name db1 ubuntu:18.04
245  docker run -it --volumes-from dbdata --name db2 ubuntu:18.04
```
## 容器端口映射

### 端口映射实现容器访问

当容器中运行一些网络应用，要让外部访问这些应用时，可以通过-P或-p参数来指定端口映射。当使用-P（大写的）标记时，Docker会随机映射一个49000～49900的端口到内部容器开放的网络端口，例如：

```plain
docker run -d -P nginx:1.14.2
```
### 映射本地接口地址到容器接口

```plain
docker run --rm -d -p 8081:80 --name nginx-test-web
```
### 映射到指定地址的任意接口

使用IP::ContainerPort绑定localhost的任意端口到容器的80端口，本地主机会自动分配一个端口

```plain
docker run -d -p ::80 nginx:1.14.2 
```
### 查看映射端口配置

```plain
[root@womr-c ~]# docker port eager_lalande
80/tcp -> 0.0.0.0:49155
```
## Dockerfile创建镜像

Dockerfile由一行行命令语句组成，并且支持以#开头的注释行。一般而言，Dockerfile主体内容分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

例如写入一个go web程序，如下:

```plain
package main
import "net/http"
func main() {
   http.HandleFunc("/", func(writer http.ResponseWriter, request *http.Request) {
       writer.Write([]byte("hello"))
   })
   http.ListenAndServe(":9999",nil)
}
```
Dcokerfile如下信息：
```plain
FROM golang
MAINTAINER worm-c
WORKDIR $GOPATH/src/godocker
COPY . $GOPATH/src/godocker
RUN go build main.go
EXPOSE 9999
ENTRYPOINT ["./main"]
```
生成docker镜像，如下：
```plain
docker build -t worm/godemo:v1 .
docker run -it -d -p 80:9999 worm/godemo:v1 /bin/bash
curl localhost:80
//out hello
```
## 容器编排工具DockerCompose

### 安装

[下载地址](https://github.com/docker/compose/releases/download/1.28.2/docker-compose-Linux-x86_64)

文件权限：

```plain
sudo chmod a+x /usr/local/bin/docker-compose
```
验证是否成功：
```plain
docker-compose version
[root@worm-c ~]# docker-compose version
docker-compose version 1.28.2, build 67630359
docker-py version: 4.4.1
CPython version: 3.7.9
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```
运行一个简单的mysql应用，新建一个文件mysql-service.yaml，如下：
```plain
version: '3'
services:
  mysql-db:
    container_name: mysql-docker        # 指定容器的名称
    image: mysql:5.7                 # 指定镜像和版本
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_ROOT_HOST: ${MYSQL_ROOT_HOST}
    volumes:
      - "${MYSQL_DIR}/data:/var/lib/mysql"           # 挂载数据目录
      - "${MYSQL_DIR}/config:/etc/mysql/conf.d"      # 挂载配置文件目
```



## errors

[root@localhost ~]# rpm -ivh docker/*.rpm --force
warning: docker/containerd.io-1.2.6-3.3.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
warning: package docker-ce-3:19.03.9-3.el7.x86_64 was already added, skipping docker-ce-selinux-17.03.3.ce-1.el7.noarch
error: Failed dependencies:
	containerd conflicts with containerd.io-1.2.6-3.3.el7.x86_64
	runc conflicts with containerd.io-1.2.6-3.3.el7.x86_64
	containerd conflicts with (installed) containerd.io-1.2.6-3.3.el7.x86_64
	runc conflicts with (installed) containerd.io-1.2.6-3.3.el7.x86_64



![image-20210706095535080](C:\Users\h1172\AppData\Roaming\Typora\typora-user-images\image-20210706095535080.png)

```
yum -y remove containerd.io-1.2.6-3.3.el7.x86_64
```

