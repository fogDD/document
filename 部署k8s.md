

### 1.安装docker

#### 1.1（安装过docker）清除已安装的docker

```
systemctl stop docker
rm -rf /etc/docker
rm -rf /run/docker
rm -rf /var/lib/dockershim
rm -rf /var/lib/docker
rm -rf /usr/bin/docker
```

#### 1.2安装docker（离线）

```
unzip docker.zip
rpm -ivh docker/*.rpm --force
systemctl enable docker
systemctl start docker
echo "{\"exec-opts\":[\"native.cgroupdriver=systemd\"],\"insecure-registries\":[\"192.168.1.88:5000\"]}" >> /etc/docker/daemon.json
systemctl daemon-reload
systemctl restart docker
```

### 2.安装k8s(离线)

```
unzip k8s-1.18.5.zip
rpm -ivh k8s-1.18.5/*.rpm --force
systemctl enable kubelet
```



### 3.安装nfs

```
unzip nfs.zip
rpm -ivh nfs/*.rpm --force
systemctl enable rpcbind
systemctl start rpcbind
```



### 4.注意事项

#### 1.在集群建立时

```
vim  /etc/docker/daemon.json文件的时候加上
"exec-opts": ["native.cgroupdriver=systemd"]
```

 Kubernetes 推荐使用 `systemd` 来代替 `cgroupfs`

` 因为`systemd是Kubernetes自带的cgroup管理器, 负责为每个进程分配cgroups, 

  但docker的cgroup driver默认是cgroupfs,这样就同时运行有两个cgroup控制管理器, 

  当资源有压力的情况时,有可能出现不稳定的情况

#### 

### 5.运行demo.sh脚本

```
systemctl stop firewalld
systemctl disable firewalld
# 关闭 SeLinux
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
# 关闭 swap
swapoff -a
yes | cp /etc/fstab /etc/fstab_bakcat /etc/fstab_bak
cat /etc/fstab_bak |grep -v swap > /etc/fstab
# 修改 /etc/sysctl.conf# 如果有配置，则修改
sed -i "s#^net.ipv4.ip_forward.*#net.ipv4.ip_forward=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-ip6tables.*#net.bridge.bridge-nf-call-ip6tables=1#g"  /etc/sysctl.conf
sed -i "s#^net.bridge.bridge-nf-call-iptables.*#net.bridge.bridge-nf-call-iptables=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.all.disable_ipv6.*#net.ipv6.conf.all.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.default.disable_ipv6.*#net.ipv6.conf.default.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.lo.disable_ipv6.*#net.ipv6.conf.lo.disable_ipv6=1#g"  /etc/sysctl.conf
sed -i "s#^net.ipv6.conf.all.forwarding.*#net.ipv6.conf.all.forwarding=1#g"  /etc/sysctl.conf
# 可能没有，追加
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding = 1"  >> /etc/sysctl.conf
# 执行命令以应用
sysctl -p
```





### 6.在k8s集群建立时，vim  /etc/docker/daemon.json文件的时候加上

```
"exec-opts": ["native.cgroupdriver=systemd"]
```



### 7./root/bc-deploy/base/3_k8s 目录下的 k8s.tar.gz weave.tar.gz,把k8s需要的组件镜像导入到仓库

的文件: docker load < k8s.tar.gz  &&docker load weave.tar.gz

```
docker load < k8s.tar.gz && docker load < weave.tar.gz
```



### errors

rpm nfs 包时候报错

![image-20210712122106759](C:\Users\h1172\AppData\Roaming\Typora\typora-user-images\image-20210712122106759.png)

解决方法：

```
vim init.sh
rpm -ivh nfs/*.rpm --force --nodeps
```

nodeps的意思是忽视依赖关系。因为各个软件之间会有多多少少的联系。有了这两个设置选项就忽略了这些依赖关系，强制安装或者卸载
