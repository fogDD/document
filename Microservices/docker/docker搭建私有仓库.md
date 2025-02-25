### docker搭建私有仓库

在 Docker 中，当我们执行 docker pull xxx 的时候，可能会比较好奇，docker 会去哪儿查找并下载镜像呢？

它实际上是从 registry.hub.docker.com 这个地址去查找，这就是Docker公司为我们提供的公共仓库，上面的镜像，大家都可以看到，也可以使用。

所以，我们也可以带上仓库地址去拉取镜像，如：docker pull registry.hub.docker.com/library/alpine，不过要注意，这种方式下载的镜像的默认名称就会长一些。

如果要在公司中使用 Docker，我们基本不可能把商业项目上传到公共仓库中，那如果要多个机器共享，又能怎么办呢？

正因为这种需要，所以私有仓库也就有用武之地了。

所谓私有仓库，也就是在本地（局域网）搭建的一个类似公共仓库的东西，搭建好之后，我们可以将镜像提交到私有仓库中。这样我们既能使用 Docker 来运行我们的项目镜像，也避免了商业项目暴露出去的风险。

#### 1.创建宿主机存储目录，即放容器镜像的路径（和私有仓库容器相互挂载）

```
mkdir -p /opt/data/registry
```



#### 2.下载并启动一个registry容器，-v指定镜像文件本地存放路径

```
docker run -d -p 5000:5000 --restart=always -v /opt/data/registry:/var/lib/registry --name private_registry registry
```



#### 3.配置http权限支持(在推送到的时候，可能会遇到问题：http: server gave HTTP response to HTTPS client，因为默认是提交到 https，但我们的仓库是使用的http，此时要么创建一个https映射，要么将仓库地址加入到不安全的仓库列表中)

```
vim /etc/docker/daemon.json

{
"insecure-registries":["192.168.12.102(你仓库的ip地址):5000"]
}
```

设置完之后重启服务，重启registry服务

```
systemctl daemon-reload
systemctl restart docker
docker restart private_registry
```



#### 4.测试上传和下载镜像到本地仓库

使用docker tag命令将镜像标记为192.168.12.102:5000（你本机的私有仓库ip地址）/test

```
docker tag ubuntu:18.04 192.168.12.102:5000/test
```

推送到本地仓库

```
docker push 192.168.12.102:5000/test
```

查看上传的镜像

```
curl http://192.168.92.134:5000/v2/
```

或者可以在宿主机查看刚刚registry容器挂载的目录

```
ll /opt/data/registry
```

可以删除本地的test镜像，然后测试从本地仓库重新下载镜像

```
docker rmi 容器镜像名字:版本
```

从本地仓库拉取镜像

```
docker pull 192.168.12.102:5000/test
```



#### 5.测试局域网其他服务器从本地仓库下载镜像

直接拉取会出现以下问题

Using default tag: latest
Error response from daemon: Get https://192.168.12.102:5000/v2/: http: server gave HTTP response to HTTPS client



Docker自从1.3.X之后docker registry交互默认使用的是HTTPS，但是搭建私有镜像默认使用的是HTTP服务，所以与私有镜像交互时出现以上错误，此时要么创建一个https映射，要么将仓库地址加入到不安全的仓库列表中

```
vim /etc/docker/daemon.json

{
"insecure-registries":["192.168.12.102(你仓库的ip地址):5000"]
}
```

然后重启服务

```
systemctl daemon-reload
systemctl restart docker
```

再拉取试试

```
docker pull 192.168.12.102:5000/test
```

Status: Downloaded newer image for 192.168.12.102:5000/test:latest

有这个显示表示成功