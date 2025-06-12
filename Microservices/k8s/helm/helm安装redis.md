# helm安装redis

### 1.安装helm

**这里推荐安装3.8版本以上的helm，如果是3.8版本之前的helm安装chart会提示OCI问题**

```
wget https://get.helm.sh/helm-v3.8.0-linux-amd64.tar.gz
tar -zxvf helm-v3.8.0-linux-amd64.tar.gz 
mv helm /usr/local/bin/
```



### 2.添加bitnami源仓库

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

**搜索redis chart**

```
helm search repo redis
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/redis                           21.0.3          8.0.0           Redis(R) is an open source, advanced key-value ...
bitnami/redis-cluster                   12.0.2          8.0.0           Redis(R) is an open source, scalable, distribut...
```

**下载**

```
helm pull bitnami/redis
```

如果网络没有问题应该是正常下载成功



### 3.redis配置文件

```
###
global:
  imageRegistry: ""
  ## E.g.
  ## imagePullSecrets:
  ##   - myRegistryKeySecretName
  ##
  imagePullSecrets: []
  defaultStorageClass: ""
  storageClass: "managed-nfs-storage"
  ## Security parameters
  ##
  security:
    ## @param global.security.allowInsecureImages Allows skipping image verification
    allowInsecureImages: false
  redis:
    password: "123456"
###
```

修改redis的sc选择器，使用本地自己创建的sc选择器。**sc：通过 StorageClass 可以实现仅仅配置 PVC，然后交由 StorageClass 根据 PVC 的需求动态创建 PV**。

**storageClass: "managed-nfs-storage"**

修改redis的密码

**password: "123456"**



```
###
architecture: replication
###
```

架构默认就是集群模式



```
###
  persistence:
    ## @param master.persistence.enabled Enable persistence on Redis(R) master nodes using Persistent Volume Claims
    ##
    enabled: true
    ## @param master.persistence.medium Provide a medium for `emptyDir` volumes.
    ##
    medium: ""
    ## @param master.persistence.sizeLimit Set this to enable a size limit for `emptyDir` volumes.
    ##
    sizeLimit: ""
    ## @param master.persistence.path The path the volume will be mounted at on Redis(R) master containers
    ## NOTE: Useful when using different Redis(R) images
    ##
    path: /data
    ## @param master.persistence.subPath The subdirectory of the volume to mount on Redis(R) master containers
    ## NOTE: Useful in dev environments
    ##
    subPath: ""
    ## @param master.persistence.subPathExpr Used to construct the subPath subdirectory of the volume to mount on Redis(R) master containers
    ##
    subPathExpr: ""
    ## @param master.persistence.storageClass Persistent Volume storage class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is set, choosing the default provisioner
    ##
    storageClass: ""
    ## @param master.persistence.accessModes Persistent Volume access modes
    ##
    accessModes:
      - ReadWriteOnce
    ## @param master.persistence.size Persistent Volume size
    ##
    size: 1Gi
    ## @param master.persistence.annotations Additional custom annotations for the PVC
    ##
    annotations: {}
    ## @param master.persistence.labels Additional custom labels for the PVC
    ##
    labels: {}
    ## @param master.persistence.selector Additional labels to match for the PVC
    ## e.g:
    ## selector:
    ##   matchLabels:
    ##     app: my-app
    ##
    selector: {}
    ## @param master.persistence.dataSource Custom PVC data source
    ##
    dataSource: {}
    ## @param master.persistence.existingClaim Use a existing PVC which must be created manually before bound
    ## NOTE: requires master.persistence.enabled: true
    ##
    existingClaim: ""
###
```

这里可以修改持久化设置的比如路径、sc、selector选择器、size以及pv的访问模式等功能，我这边暂时只设置size为1G



```
###
  service:
    ## @param master.service.type Redis(R) master service type
    ##
    type: ClusterIP
    ## @param master.service.portNames.redis Redis(R) master service port name
    ##
    portNames:
      redis: "tcp-redis"
    ## @param master.service.ports.redis Redis(R) master service port
    ##
    ports:
      redis: 6379
    ## @param master.service.nodePorts.redis Node port for Redis(R) master
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ## NOTE: choose port between <30000-32767>
    ##
    nodePorts:
      redis: ""
    ## @param master.service.externalTrafficPolicy Redis(R) master service external traffic policy
    ## ref: https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip
    ##
    externalTrafficPolicy: Cluster
    ## @param master.service.extraPorts Extra ports to expose (normally used with the `sidecar` value)
    ##
    extraPorts: []
    ## @param master.service.internalTrafficPolicy Redis(R) master service internal traffic policy (requires Kubernetes v1.22 or greater to be usable)
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service-traffic-policy/
    ##
    internalTrafficPolicy: Cluster
    ## @param master.service.clusterIP Redis(R) master service Cluster IP
    ##
    clusterIP: ""
    ## @param master.service.loadBalancerIP Redis(R) master service Load Balancer IP
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
###
```

这里可以修改redis集群的网络访问模式，默认是ClusterIP（集群内部访问）。如果需要集群外部网络访问，需要把ClusterIP设置成NodePort，然后在ports中设置一个端口在30000-32767之间



### 4.开始创建redis

```
[root@k8s-master redis]# kubectl create namespace redis
```

k8s创建命令空间

```
NAME: redis
LAST DEPLOYED: Wed Jun  4 04:49:17 2025
NAMESPACE: redis
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 21.0.3
APP VERSION: 8.0.0

Did you know there are enterprise versions of the Bitnami catalog? For enhanced secure software supply chain features, unlimited pulls from Docker, LTS support, or application customization, see Bitnami Premium or Tanzu Application Catalog. See https://www.arrow.com/globalecs/na/vendors/bitnami for more information.

** Please be patient while the chart is being deployed **

Redis&reg; can be accessed on the following DNS names from within your cluster:

    redis-master.redis.svc.cluster.local for read/write operations (port 6379)
    redis-replicas.redis.svc.cluster.local for read-only operations (port 6379)



To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace redis redis -o jsonpath="{.data.redis-password}" | base64 -d)

To connect to your Redis&reg; server:

1. Run a Redis&reg; pod that you can use as a client:

   kubectl run --namespace redis redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:8.0.0-debian-12-r0 --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client \
   --namespace redis -- bash

2. Connect using the Redis&reg; CLI:
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h redis-master
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h redis-replicas

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace redis svc/redis-master 6379:6379 &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p 6379

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - replica.resources
  - master.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
[root@k8s-master redis-all]# 
```

创建成功



### 5.检查redis的k8s资源

```
[root@k8s-master clash]# kubectl get all -n redis
NAME                   READY   STATUS    RESTARTS        AGE
pod/redis-master-0     1/1     Running   0               6m21s
pod/redis-replicas-0   1/1     Running   1 (5m25s ago)   6m21s
pod/redis-replicas-1   1/1     Running   0               5m
pod/redis-replicas-2   1/1     Running   0               4m8s

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/redis-headless   ClusterIP   None            <none>        6379/TCP   6m21s
service/redis-master     ClusterIP   10.103.46.175   <none>        6379/TCP   6m21s
service/redis-replicas   ClusterIP   10.105.172.39   <none>        6379/TCP   6m21s

NAME                              READY   AGE
statefulset.apps/redis-master     1/1     6m22s
statefulset.apps/redis-replicas   3/3     6m22s
```

创建了1个masterpod节点➕3个从pod节点

为headless、master、replicas创建了三个svc供平台内部访问

为master、replicas从创建了两个statefulset控制器

```
[root@k8s-master clash]# kubectl get pvc -n redis
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
redis-data-redis-master-0     Bound    pvc-a86cca4f-448d-4ed8-884b-825550d3cad7   8Gi        RWO            managed-nfs-storage   6m50s
redis-data-redis-replicas-0   Bound    pvc-b8da9fe6-c9df-467d-a076-25a72e569176   1Gi        RWO            managed-nfs-storage   6m50s
redis-data-redis-replicas-1   Bound    pvc-4cdf354a-47ac-4bee-8826-3790c92f23c2   1Gi        RWO            managed-nfs-storage   5m29s
redis-data-redis-replicas-2   Bound    pvc-21f251b8-b06e-4152-9ad7-f02b1eebd3fe   1Gi        RWO            managed-nfs-storage   4m37s
```

为master和从节点每个都创建了pvc并且绑定了之前创建的sc，可以通过使用sc动态制备pv



如果没创建成功原因是因为sc导致的话，需要检查sc是否创建成功以及nfs的pod

```
[root@k8s-master clash]# kubectl get sc
NAME                  PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
managed-nfs-storage   fuseim.pri/ifs   Retain          Immediate           false                  76d
[root@k8s-master clash]# kubectl get pod -A | grep nfs
kube-system   nfs-client-provisioner-8464755bfc-cjphg    1/1     Running   2 (5h16m ago)   68d
```



### 6.进入redis主节点容器检查

输入密码后进入redis主节点。植入数据并且获取测试

```
[root@k8s-master ~]# kubectl exec -it redis-master-0 -n redis -- bash
I have no name!@redis-master-0:/$ redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> set name xiaoliu
OK
127.0.0.1:6379> get name
"xiaoliu"
127.0.0.1:6379> 
```



### 7.进入redis从节点容器检查

输入密码后进入redis从节点，获取数据并进行数据植入测试

```
[root@k8s-master ~]# kubectl exec -it -n redis redis-replicas-0 -- bash
I have no name!@redis-replicas-0:/$ 
I have no name!@redis-replicas-0:/$ 
I have no name!@redis-replicas-0:/$ redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> get name
"xiaoliu"
127.0.0.1:6379> set name xiaoliu
(error) READONLY You can't write against a read only replica.
127.0.0.1:6379> 
```

