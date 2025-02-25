### 查看证书日期

```
kubeadm alpha certs check-expiration
```



### 备份kubernetes目录（原有的证书目录）

```
cp /etc/kubernetes /etc/kubernetes-2021-**-**   ///证书目录
cp /root/.kube /root/.kube--**-** //日期		///客户端配置文件
```



### 开始更新

生成集群的配置文件

```
kubeadm config view > xxxx/kubeadm-20211115.yaml  ///xxxx替换成那些k8s yaml的存放目录		///日期换成更换当天的日期
```

更新证书

```
kubeadm alpha certs renew all --config xxxx/kubeadm-20211115.yaml			////xxxx替换成那些k8s yaml的存放目录		///后面的yaml文件替换成你刚刚存下来的yaml，我这里是kubeadm-20211115.yaml
```

提示renewed之后更新完后再查看一遍证书日期，可以看到已经更新



更新客户端配置文件

```
cp /etc/kubernetes/admin.conf ~/.kube/config
```

更新完后输入kubectl命令检查集群是否正常