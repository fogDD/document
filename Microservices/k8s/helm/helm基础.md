# helm基础

## 一、安装helm(tar包)

在你要安装的目录中

```
wget  https://get.helm.sh/helm-v3.2.3-linux-amd64.tar.gz
```

解压

```
tar -zxvf helm-v3.2.3-linux-amd64.tar.gz
```

把二进制文件helm移动到/usr/local/bin/helm

```
[root@k8s-master linux-amd64]# cp helm /usr/local/bin/helm
[root@k8s-master linux-amd64]# helm version
version.BuildInfo{Version:"v3.2.3", GitCommit:"8f832046e258e2cb800894579b1b3b50c2d83492", GitTreeState:"clean", GoVersion:"go1.13.12"}
```



## 二、添加helm仓库

我们这里以安装ingress-nginx为例子

```
[root@k8s-master linux-amd64]# helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories
[root@k8s-master linux-amd64]# helm repo list
NAME            URL                                       
ingress-nginx   https://kubernetes.github.io/ingress-nginx
[root@k8s-master linux-amd64]# helm search repo ingress-nginx
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
ingress-nginx/ingress-nginx     4.12.0          1.12.0          Ingress controller for Kubernetes using NGINX a...
```

下载软件包

```
[root@k8s-master linux-amd64]# helm pull ingress-nginx/ingress-nginx --version=4.4.2
[root@k8s-master linux-amd64]# ll
-rw-r--r-- 1 root root    45268 Feb 28 16:51 ingress-nginx-4.4.2.tgz
```

