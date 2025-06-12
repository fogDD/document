# helm-chart实战：升级与回滚



## 一、hellm应用升级

### 1.修改应用的values应用配置文件

```
##
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
    password: "123456789"
    #password: "123456"
  ## Compatibility adaptations for Kubernetes platforms
  ###
```

比如我这边修改了password，之前是`password`: "123456"，现在是`password`: "123456789"



### 2.开始使用helm更新redis

```
helm upgrade redis ./redis/ -n redis
```

`upgrade` 更新  `redis` 应用名  ./redis/ 更新当前应用目录  `-n redis` 指定命令空间

```
[root@k8s-master redis-all]# helm upgrade redis ./redis/ -n redis
Release "redis" has been upgraded. Happy Helming!
NAME: redis
LAST DEPLOYED: Wed Jun 11 23:15:40 2025
NAMESPACE: redis
STATUS: deployed
REVISION: 2
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

更新完会提示刚开始安装成功的显示信息



```
[root@k8s-master redis-all]# 
[root@k8s-master redis-all]# kubectl get pod -n redis
NAME               READY   STATUS    RESTARTS      AGE
redis-master-0     1/1     Running   0             40s
redis-replicas-0   0/1     Running   2 (17s ago)   7d18h
redis-replicas-1   0/1     Running   1 (14s ago)   7d18h
redis-replicas-2   1/1     Running   0             40s
```

可以getpod看到master已经更新完毕，但是三个从节点还在更新。这里可以看到sts的特性，有序部署更新，先从2更新到1到0

```
[root@k8s-master redis-all]# kubectl get pod -n redis
NAME               READY   STATUS    RESTARTS   AGE
redis-master-0     1/1     Running   0          2m37s
redis-replicas-0   1/1     Running   0          75s
redis-replicas-1   1/1     Running   0          101s
redis-replicas-2   1/1     Running   0          2m37s
```

注意看这三个从节点的部署时间，sts有序更新



### 3.进入redis master以及replice容器测试

进入redis master容器测试

```
[root@k8s-master redis-all]# kubectl exec -it -n redis redis-master-0 -- bash
I have no name!@redis-master-0:/$ redis-cli
127.0.0.1:6379> auth 123456
(error) WRONGPASS invalid username-password pair or user is disabled.
127.0.0.1:6379> auth 123456789
OK
127.0.0.1:6379> get name
"xiaoliu"
```

可以看到密码修改成了我们刚更新的密码



进入redis replice容器测试

```
[root@k8s-master redis-all]# kubectl exec -it -n redis redis-replicas-2 -- bash
I have no name!@redis-replicas-2:/$ redis-cli
127.0.0.1:6379> auth 123456789
OK
127.0.0.1:6379> get name
"xiaoliu"
```

可以看到即使使用helm更新了redis，也保证了redis数据也不会丢失，这就是sts的特性，持久化机制



## 二、helm应用回滚

### 1.查看历史版本

```
[root@k8s-master redis-all]# helm history -n redis redis
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION     
1               Wed Jun  4 04:49:17 2025        superseded      redis-21.0.3    8.0.0           Install complete
2               Wed Jun 11 23:15:40 2025        deployed        redis-21.0.3    8.0.0           Upgrade complete
```



### 2.开始回滚

```
[root@k8s-master redis-all]# helm rollback redis 1 -n redis
Rollback was a success! Happy Helming!
```



### 3.进入redis master以及replice容器测试

进入redis-master节点测试

```
[root@k8s-master redis-all]# kubectl exec -it -n redis redis-master-0 -- bash
I have no name!@redis-master-0:/$ 
I have no name!@redis-master-0:/$ redis-cli
127.0.0.1:6379> auth 123456789
(error) WRONGPASS invalid username-password pair or user is disabled.
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> get name
"xiaoliu"
127.0.0.1:6379> set age 18
OK
127.0.0.1:6379> get age
"18"
```

可以看到又恢复到上一个版本的密码123456，而且数据也没丢失



进入redis-replice节点测试

```
[root@k8s-master redis-all]# kubectl exec -it -n redis redis-replicas-2 -- bash
I have no name!@redis-replicas-2:/$ redis-cli 
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> get name
"xiaoliu"
127.0.0.1:6379> get age
"18"
127.0.0.1:6379> 
```



## 三、helm删除redis应用以及pvc、pv

首先用helm删除应用

```
[root@k8s-master redis-all]# helm list -n redis
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
redis   redis           3               2025-06-11 23:51:47.263870462 -0400 EDT deployed        redis-21.0.3    8.0.0      
[root@k8s-master redis-all]# helm delete redis -n redis
release "redis" uninstalled
```



然后把绑定的pvc也删除掉（helm删除应用后，需要手动删除pvc）

```
[root@k8s-master redis-all]# kubectl get pvc -n redis
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
redis-data-redis-master-0     Bound    pvc-a86cca4f-448d-4ed8-884b-825550d3cad7   8Gi        RWO            managed-nfs-storage   7d23h
redis-data-redis-replicas-0   Bound    pvc-b8da9fe6-c9df-467d-a076-25a72e569176   1Gi        RWO            managed-nfs-storage   7d23h
redis-data-redis-replicas-1   Bound    pvc-4cdf354a-47ac-4bee-8826-3790c92f23c2   1Gi        RWO            managed-nfs-storage   7d23h
redis-data-redis-replicas-2   Bound    pvc-21f251b8-b06e-4152-9ad7-f02b1eebd3fe   1Gi        RWO            managed-nfs-storage   7d22h
[root@k8s-master redis-all]# kubectl get pv -n redis
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS          REASON   AGE
pvc-1a81b395-12ac-4ced-9cde-fe3bc4c1a588   300Mi      RWO            Retain           Bound    default/auto-pv-test-pvc               managed-nfs-storage            84d
pvc-21f251b8-b06e-4152-9ad7-f02b1eebd3fe   1Gi        RWO            Retain           Bound    redis/redis-data-redis-replicas-2      managed-nfs-storage            7d22h
pvc-2fb326a2-3625-4936-9682-51d36ab24163   1Gi        RWX            Retain           Bound    default/nginx-sc-test-pvc-nginx-sc-0   managed-nfs-storage            84d
pvc-4cdf354a-47ac-4bee-8826-3790c92f23c2   1Gi        RWO            Retain           Bound    redis/redis-data-redis-replicas-1      managed-nfs-storage            7d23h
pvc-a86cca4f-448d-4ed8-884b-825550d3cad7   8Gi        RWO            Retain           Bound    redis/redis-data-redis-master-0        managed-nfs-storage            7d23h
pvc-b8da9fe6-c9df-467d-a076-25a72e569176   1Gi        RWO            Retain           Bound    redis/redis-data-redis-replicas-0      managed-nfs-storage            7d23h
```

```
[root@k8s-master redis-all]# kubectl delete pvc -n redis redis-data-redis-master-0 redis-data-redis-replicas-0 redis-data-redis-replicas-1 redis-data-redis-replicas-2
persistentvolumeclaim "redis-data-redis-master-0" deleted
persistentvolumeclaim "redis-data-redis-replicas-0" deleted
persistentvolumeclaim "redis-data-redis-replicas-1" deleted
persistentvolumeclaim "redis-data-redis-replicas-2" deleted
[root@k8s-master redis-all]# kubectl get pvc -n redis
No resources found in redis namespace.
```

```
[root@k8s-master redis-all]# kubectl get pv -A
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                                  STORAGECLASS          REASON   AGE
pvc-1a81b395-12ac-4ced-9cde-fe3bc4c1a588   300Mi      RWO            Retain           Bound      default/auto-pv-test-pvc               managed-nfs-storage            84d
pvc-21f251b8-b06e-4152-9ad7-f02b1eebd3fe   1Gi        RWO            Retain           Released   redis/redis-data-redis-replicas-2      managed-nfs-storage            7d23h
pvc-2fb326a2-3625-4936-9682-51d36ab24163   1Gi        RWX            Retain           Bound      default/nginx-sc-test-pvc-nginx-sc-0   managed-nfs-storage            84d
pvc-4cdf354a-47ac-4bee-8826-3790c92f23c2   1Gi        RWO            Retain           Released   redis/redis-data-redis-replicas-1      managed-nfs-storage            7d23h
pvc-a86cca4f-448d-4ed8-884b-825550d3cad7   8Gi        RWO            Retain           Released   redis/redis-data-redis-master-0        managed-nfs-storage            7d23h
pvc-b8da9fe6-c9df-467d-a076-25a72e569176   1Gi        RWO            Retain           Released   redis/redis-data-redis-replicas-0      managed-nfs-storage            7d23h
[root@k8s-master redis-all]# 
[root@k8s-master redis-all]# kubectl delete pv pvc-21f251b8-b06e-4152-9ad7-f02b1eebd3fe pvc-4cdf354a-47ac-4bee-8826-3790c92f23c2 pvc-a86cca4f-448d-4ed8-884b-825550d3cad7 pvc-b8da9fe6-c9df-467d-a076-25a72e569176 -n redis
warning: deleting cluster-scoped resources, not scoped to the provided namespace
persistentvolume "pvc-21f251b8-b06e-4152-9ad7-f02b1eebd3fe" deleted
persistentvolume "pvc-4cdf354a-47ac-4bee-8826-3790c92f23c2" deleted
persistentvolume "pvc-a86cca4f-448d-4ed8-884b-825550d3cad7" deleted
persistentvolume "pvc-b8da9fe6-c9df-467d-a076-25a72e569176" deleted
```

把剩下释放掉的pv删除（这里为什么删除pvc之后pv没有自动删除，因为pv的回收策略设置了retain模式） 

Retained（保留）、Recycled（回收）或 Deleted（删除）