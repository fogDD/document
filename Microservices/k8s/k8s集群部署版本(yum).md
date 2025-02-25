# k8s集群部署v2版本

#### kubernetes部署环境要求

（1）二台以上的虚拟机，操作系统CentSO7.x-86_x64

（2）硬件配置：内存2GB或2G+，CPU2核或CPU2核+；

（3）集群内各个虚拟机之间能相互通信；

（4）禁止swap分区

如果环境不满足要求，会出现报错

注意：docker版本跟kubernetes版本问题，版本不兼容会出现报错

准备两台结节点（master节点和node节点）

## 一、k8s环境部署

### （1）关闭防火墙和修改主机名

```
systemctl stop firewalld
systemctl disable firewalld
hostnamectl set-hostname master			#marter

hostnamectl set-hostname node			#node
```

### （2）关闭seLinux

```
sed -i ' s/enforcing/disabled/' /etc/selinux/config 
setenforce 0       
```

### （3）关闭swap(k8s禁止虚拟内存以提高性能)

```
sed -ri ' s/.*swap.*/#&/' /etc/fstab   
swapoff -a  
```

### （4）在master添加hosts

```
echo "127.0.0.1   $(hostname)" >> /etc/hosts
```

### （5）设置网桥参数

```
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding = 1"  >> /etc/sysctl.conf
# 执行命令以应用
sysctl --system
sysctl -p
```

### (6)时间同步

```
yum -y install ntpdate
ntpdate time.windows.com
```

## 二、docker安装

### 1.1、安装依赖包

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 1.2、添加软件源信息

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 1.3、更新yum缓存

```
sudo yum makecache fast
```

### 1.4、安装指定版本的docker-ce

```
yum install docker-ce-19.03.5 docker-ce-cli-19.03.5 containerd.io -y
```

## 三、部署k8s环境

### 1.1、配置k8s的yum源

```
# 配置k8s的yum源（注：复制全部一起执行）

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 1.2、指定安装kubelet、kubeadm、kubectl版本包

```
yum install -y kubelet-1.18.5 kubeadm-1.18.5 kubectl-1.18.5

crictl config runtime-endpoint /run/containerd/containerd.sock

sed -i "s#^ExecStart=/usr/bin/dockerd.*#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd#g" /usr/lib/systemd/system/docker.service
```

```
# 重启 docker，并启动 kubelet
systemctl daemon-reload
systemctl restart docker
systemctl enable kubelet && systemctl start kubelet
```

### 1.3、导入k8s.tar.gz、weave.tar.gz镜像

```
#拷k8s、weaver镜像到master节点、node节点(检查docker)
docker load <k8s.tar.gz
docker load <weave.tar.gz
```

### 1.4初始化k8s（只在K8S执行）

```
###master节点执行###(注意swapoff服务是否关闭 free查看)
kubeadm init --kubernetes-version=1.18.5（版号） \
--apiserver-advertise-address=192.168.100.50 （本机ip） \
--service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
```

![](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210803102315683.png)

```
###master节点执行###
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

###node节点执行### （从master初始化token出来的内容，拷贝到node节点执行）注：以下请勿复制粘贴到（node节点）
   kubeadm join 192.168.100.50:6443 --token m0iuds.3wluditcwfsmerm5     --discovery-token-ca-cert-hash sha256:97a190e68167f5b898bf0a75510437acec81584dffcc8ab897bafa6f091a1782
```

### 1.5、重启 docker，并启动 kubelet

```
# 重启 docker，并启动 kubelet
systemctl daemon-reload
systemctl restart docker
systemctl enable kubelet && systemctl start kubelet

docker version
```

### 1.6、把weave-net-yaml导入到（master节点） 

```
[root@master ~]# kubectl apply -f weave-net.yaml 
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

### 1.7、查看 master 节点初始化结果

```
[root@master ~]# kubectl get node -A -o wide
NAME     STATUS   ROLES    AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
master   Ready    master   19m     v1.18.5   192.168.100.50   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://20.10.7
node    Ready    <none>   9m22s   v1.18.5   192.168.100.51   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://20.10.7


注：master和node出现ready就成功啦
```

