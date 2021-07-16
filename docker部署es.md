# docker部署es

#### 1.从仓库拉取pull镜像

docker pull elasticsearch:5.6.8

#### 2.新建数据挂载目录和配置文件目录

```
[root@localhost ~]# mkdir -p ~/soft/ES/data{1..3}

[root@localhost ~]# mkdir -p ~/soft/ES/config

[root@localhost ~]# vim soft/ES/config/es1.yml
```

es1.yml文件

```
cluster.name: elasticsearch-cluster
node.name: es-node1
network.bind_host: 0.0.0.0
network.publish_host: 192.168.1.103
http.port: 9200
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["192.168.1.103:9300","192.168.1.103:9301","192.168.1.103:9302"]
discovery.zen.minimum_master_nodes: 2
```

cluster.name:节点名称

cluster.name：集群名

network.bind_host: 0.0.0.0 ：绑定的ip：默认只允许本机访问，修改为0.0.0.0后则可以远程访问

http.port：9200是http协议的web客户端RESTful端口

transport.tcp.port: 9300是集群节点指点的tcp通讯端口



es2.yml文件

```
cluster.name: elasticsearch-cluster
node.name: es-node2
network.bind_host: 0.0.0.0
network.publish_host: 192.168.1.103
http.port: 9201
transport.tcp.port: 9301
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["192.168.1.103:9300","192.168.1.103:9301","192.168.1.103:9302"]
discovery.zen.minimum_master_nodes: 2
```



es3.yml文件

```
cluster.name: elasticsearch-cluster
node.name: es-node3
network.bind_host: 0.0.0.0
network.publish_host: 192.168.1.103
http.port: 9202
transport.tcp.port: 9302
http.cors.enabled: true
http.cors.allow-origin: "*"
node.master: true
node.data: true
discovery.zen.ping.unicast.hosts: ["192.168.1.103:9300","192.168.1.103:9301","192.168.1.103:9302"]
discovery.zen.minimum_master_nodes: 2
```



#### 3.调高jvm线程数限制

```
vim /etc/sysctl.conf
加入
vm.max_map_count=262144
执行生效
[root@localhost ~]# sysctl -p
vm.max_map_count = 262144
```



#### 4.使用elasticsearch镜像创建elasticsearch容器

创建节点ES01容器

```
[root@localhost ~]# docker run -e ES_JAVA_OPTS="-Xms256m -Xms256m" -d -p 9200:9200 -p 9300:9300 -v /home/soft/ES/config/es1.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/soft/ES/data1:/usr/share/elasticsearch/data --name ES01 elasticsearch:5.6.8
```

-v 配置文件路径和数据文件路径挂载到主机，-p http的tcp专用端口要绑定

注意data1是目录路径不要/data1/



创建节点ES02容器

```
[root@localhost ~]# docker run -e ES_JAVA_OPTS="-Xms256m -Xms256m" -d -p 9201:9201 -p 9301:9301 -v /home/soft/ES/config/es2.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/soft/ES/data2:/usr/share/elasticsearch/data --name ES02 elasticsearch:5.6.8
1bb556a6db3b15e949bd3e7f9fb8f73fecb929294ce0cef20cf9f7e6d267859b
```





创建节点ES03容器

```
[root@localhost ~]# docker run -e ES_JAVA_OPTS="-Xms256m -Xms256m" -d -p 9202:9202 -p 9302:9302 -v /home/soft/ES/config/es3.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /home/soft/ES/data3:/usr/share/elasticsearch/data --name ES03 elasticsearch:5.6.8
408efe66caee8677e3dfcd5c50eed6ba49cec667e63da713fbd9dd0395b6b32c
```

