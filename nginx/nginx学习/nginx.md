# nginx

### 一、关于nginx

`Nginx`是一个开源的高性能的[Web服务器](https://so.csdn.net/so/search?q=Web服务器&spm=1001.2101.3001.7020)和反向代理服务器。其主要特点包括高性能、高并发处理能力、低内存消耗、事件驱动架构、模块化设计以及良好的可靠性和稳定性。它可以处理数千个并发连接，适用于大规模的互联网应用、负载均衡、反向代理以及静态资源缓存等场景。

`Nginx`通过使用异步非阻塞的事件驱动模型，有效地避免了传统Web服务器中一个线程只能处理一个请求的缺点。这使得`Nginx`在高并发情况下能够迅速响应并处理大量的请求。



### 二、nginx异步非阻塞理解

异步非阻塞我们拆开说，先说他这个非阻塞代表什么，nginx性能高因为nginx一个进程可以处理成千上万的请求，nginx为什么能用一个进程可以处理成千上万的请求呢？因为nginx网络io用的是多路复用器，多路复用器解决的就是阻塞问题，所以我们理解因为nginx使用了多路复用器模型所以叫非阻塞。怎么叫异步呢？比如有1万个请求，这一万个请求可以分为很多种，有的是请求静态资源，有的是转发请求到后端服务器，有的是直接请求动态数据，但是不管什么样的事其实都是io，你请求静态资源你要读静态资源吧，你转发请求到后端服务器你得读返回回来的数据吧，假设其中的一个请求是向后端服务器转发请求并返回，假如这时候网络情况不好5秒钟才能得到后端服务器的返回那其他9999个请求都要等着么？nginx肯定没这么傻对吧，nginx是处理到这个请求可能阻塞的地方（向后端服务器发送请求），然后就去处理另外一个请求了（比如处理其他请求静态资源的请求），那上一个请求怎么办呢？不可能不要了是吧，他会在注册一个事件（后端服务器5秒后返回后要做什么？把返回的数据响应给客户）注册事件的地方是一个链表也就是我前面说的event queue（事件队列），具体我也没看过注册进去的是什么，但是想一想应该是把请求后端服务器返回后要做什么注册进去，这里就是异步，为什么这就叫异步呢，nginx不会每次去轮询注册链表，问他有没有事件已经完成（后端服务器有没有返回），而是当请求后端服务器返回后直接就可以执行注册进去的事情，注册进去的事情其实就是把从后端服务器请求回来的数据响应给用户，这就是异步，组合起来就是异步非阻塞，其实也可以把顺序颠倒一下非阻塞异步但是这么读起来有点变扭，先这么理解吧，不理解话或者我写的不对的地方欢迎留言，一起学习。

异步非阻塞，还是拆成非阻塞和异步，非阻塞指的是网络io的非阻塞，实现方式是多路复用器，当多路复用器返回有数据的io时，如果同步的话就要循环这些io一个一个处理，但是万一有一个io响应很慢后面的都要等待，nginx会把io放到一个队列，异步去处理

原文链接：https://blog.csdn.net/yanchendage/article/details/108151110



### 三、nginx源码简单安装

```
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

 进入到/opt/software 目录(如果没有 software则自己创建) 

```
cd /opt/software #进入对应目录
```

```
mkdir software #如果目录不存在则在对应路径下创建文件夹 
```

接下来按如下顺序进行操作：

1、解压Nginx压缩包

```
 tar -xvf nginx-1.12.2.tar.gz
```

2、进入解压缩目录，依次输入以下命令

```
./configure

make && make install
```

3、进入/usr/local/nginx/sbin目录，启动Nginx

```
./nginx #启动nginx
```

设置直接nginx运行

格式为：
ln + -s + 源文件路径 +目标文件路径
目的是通过软连接将nginx程序连接到/use/local/sbin目录中

```
ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/nginx
```

#下面是nginx的一些其他常用命令、均要在/usr/lcoal/nginx/sbin目录下执行

```
./nginx -s stop #停止nginx服务

./nginx -s reload #nginx热部署，更新配置资源重载nginx
4.此时输入进程查询命令，可以看到Nginx已经启动

ps -ef|grep nginx
```



### 四、反向代理

#### 4.1.什么是正向代理和反向代理

> 正向代理就是一个人发送一个请求直接就到达了目标的服务器

![WeChatb62aee357f5d49f2ea3661f4723d6354](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChatb62aee357f5d49f2ea3661f4723d6354.jpg)

反方代理就是请求统一被Nginx接收，nginx反向代理服务器接收到之后，按照一定的规则分发给了后端的业务处理服务器进行处理了

![WeChat585adf872a78d9cb1ffb0a0ff7fc93d0](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat585adf872a78d9cb1ffb0a0ff7fc93d0.jpg)

>  正向代理服务器代理的是客户端，而反向代理服务器代理的是服务端



##### 4.1.1反向代理的优点

> 反向代理服务器可以隐藏源服务器的存在和特征。它充当互联网云和[web服务](https://so.csdn.net/so/search?q=web服务&spm=1001.2101.3001.7020)器之间的中间层。这对于安全方面来说是很好的，特别是当您使用web托管服务时。





#### 4.2.配置反向代理

修改nginx配置文件

```
vim /usr/local/nginx/conf/nginx.conf
```

添加server_name和proxy_pass

- **listen:** 表示我们检测80端口
- **server_name:** 访问域名为 www.666.com
- **proxy_pass：**代理转发模块,主要功能是把请求转发到其它服务，即将www.666.com的请求转发到http://192.168.137:8080;

![WeChatac15dcf6ad255ef344487d7e570efcd0](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChatac15dcf6ad255ef344487d7e570efcd0.jpg)

**配置完后记得nginx -s reload**



打开网页www.666.com查看效果，我这里的后端用的是tomcat的默认index

![WeChatec97314f6ddec3c81e26a575c290bc1d](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChatec97314f6ddec3c81e26a575c290bc1d.jpg)

**如果访问不了的话，记得检查服务器防火墙以及客户端windows系统hosts文件配置**





### 五、负载均衡

#### 5.1什么是负载均衡

为了避免服务器崩溃，大家会通过负载均衡的方式来分担服务器压力。将对台服务器组成一个集群，当用户访问时，先访问到一个转发服务器，再由转发服务器将访问分发到压力更小的服务器。



#### 5.2配置好tomcat

##### 5.2.1配置tomcat的html文件

> **准备两个页面，均命名为\**test.html\**,内容分别如下：**

```cobol
<!--apache8080-->
<html>
    <head></head>
    <body>
        welcome to service 8080
    </body>
</html>
```

```
<!--apache8081-->
<html>
    <head></head>
    <body>
        welcome to service 8081
    </body>
</html>
```



安装两个tomcat

```
[root@www tomcat]# ll
total 0
drwxr-x---. 9 root root 220 May 15 05:41 apache-tomcat-8.5.100
drwxr-xr-x. 9 root root 220 May 15 05:38 apache-tomcat-8.5.100-8081p
```

在webapps中新建test目录

```
cd webapps
mkdir test
```

把刚编辑好的test.index文件放到该目录下

```
[root@www test]# ll
total 4
-rw-r--r--. 1 root root 106 May 15 05:37 test.html
[root@www test]# cat test.html 
<!--apache8080-->
<html>
    <head></head>
    <body>
        welcome to service 8080
    </body>
</html>
[root@www test]# pwd
/opt/tomcat/apache-tomcat-8.5.100/webapps/test
[root@www test]# 
```

在另外一个tomcat中同样的操作

```
[root@www test]# ll
total 4
-rw-r--r--. 1 root root 106 May 15 05:45 test.html
[root@www test]# cat test.html 
<!--apache8081-->
<html>
    <head></head>
    <body>
        welcome to service 8081
    </body>
</html>
[root@www test]# pwd
/opt/tomcat/apache-tomcat-8.5.100-8081p/webapps/test
```



##### 5.2.2配置第二个tomcat的默认端口

进入*conf* 目录中修改*server.xml*

![WeChatea39e4afbbd7a0dcba44e7f5bac0a8a1](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChatea39e4afbbd7a0dcba44e7f5bac0a8a1.jpg)

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChatcc0456b56518083da11f038e6dcf49ec.jpg)

访问测试

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat4575122763dfdfad5d181b7d3527cb73.jpg)

![](/Users/zhouqiting/Library/Application Support/typora-user-images/image-20240516121939691.png)





#### 5.3配置文件

**修改 usr/local/nginx/conf目录中的nginx.conf文件**

```cobol
vi /usr/local/nginx/conf/nginx.conf 
```

**8按下图所示修改配置后\**wq\**保存退出** 

 ![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChata74a31b8ea1597e99ffa3f393ffa71b3.jpg)

配置完成后重启nginx



#### 5.4测试

访问www.666.com/test/test.html，显示8080，8081即成功，按照轮询算法匹配请求后发到代理服务器

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChatcb910e43dfeb2b1e0d3c5d1aab99cafd.jpg)

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat1483b01ff4f0bcf3f0d24426c6524c86.jpg)





#### 5.5nginx负载均衡的算法

##### 5.5.1轮询算法（默认）

就是上面演示的nginx负载均衡的默认算法，每个请求顺序轮流分配到不同的服务器上，实现请求均衡分配。



##### 5.5.2hash算法

根据客户端IP地址进行hash运算，将同一个客户端的请求分配到同一台服务器上，适用于需要会话保持的应用。

**当你服务端的一个特定url路径会被同一个用户连续访问时，如果负载均衡策略还是轮询的话，那该用户的多次访问会被打到各台服务器上，这显然并不高效（会建立多次http链接等问题）。甚至考虑一种极端情况，用户需要分片上传文件到服务器下，然后再由服务器将分片合并，这时如果用户的请求到达了不同的服务器，那么分片将存储于不同的服务器目录中，导致无法将分片合并。所以，此类场景可以考虑采用nginx提供的ip_hash策略。既能满足每个用户请求到同一台服务器，又能满足不同用户之间负载均衡。**

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChate36fb2250b3de6ded5ee2af0f62841c1.jpg)



##### 5.5.3Weighted算法

设置不同的权重值，将请求按照权重比例分配给不同的服务器，适用于服务器性能不同的情况。

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat9e895d525cdaf941171210f09067d8f7.jpg)



##### 5.5.4URLhash算法

根据请求URL进行hash运算，将同一URL的请求分配到同一台服务器上，适用于静态内容较多的应用。

**一般来讲，要用到urlhash，是要配合缓存命中来使用。举一个我遇到的实例：有一个服务器集群A，需要对外提供文件下载，由于文件上传量巨大，没法存储到服务器磁盘中，所以用到了第三方云存储来做文件存储。服务器集群A收到客户端请求之后，需要从云存储中下载文件然后返回，为了省去不必要的网络带宽和下载耗时，在服务器集群A上做了一层临时缓存（缓存一个月）。由于是服务器集群，所以同一个资源多次请求，可能会到达不同的服务器上，导致不必要的多次下载，缓存命中率不高，以及一些资源时间的浪费。在此类场景下，为了使得缓存命中率提高，很适合使用url_hash策略，同一个url（也就是同一个资源请求）会到达同一台机器，一旦缓存住了资源，再此收到请求，就可以从缓存中读取，既减少了带宽，也减少的下载时间。**

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat7207065e199118304a49fd3fb30d0680.jpg)



##### 5.5.5fair算法

对比 weight、ip_hash更加智能的负载均衡算法，fair算法可以根据页面大小和加载时间长短智能地进行负载均衡，响应时间短的优先分配。哪个服务器的响应速度快，就将请求分配到那个服务器上。

![WeChat5b864c4b084c25e92e6635e3e27a5cc9](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat5b864c4b084c25e92e6635e3e27a5cc9.jpg)















### 六、动静分离

- Nginx是当下最热的Web容器，网站优化的重要点在于静态化网站，网站静态化的关键点则是是动静分离，动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们则根据静态资源的特点将其做缓存操作。

- 让静态的资源只走静态资源服务器，动态的走动态的服务器
- Nginx的静态处理能力很强，但是动态处理能力不足，因此，在企业中常用动静分离技术。
- 对于静态资源比如图片，js，css等文件，我们则在反向代理服务器nginx中进行缓存。这样浏览器在请求一个静态资源时，代理服务器nginx就可以直接处理，无需将请求转发给后端服务器tomcat。
- 若用户请求的动态文件，比如servlet,jsp则转发给Tomcat服务器处理，从而实现动静分离。这也是反向代理服务器的一个重要的作用。



#### 6.1配置好静态资源

我这里准备一个html文件和一个image文件，新建了目录，jpg文件如果访问404的话，记得给jpg文件上权限

```
[root@www web-data]# pwd
/opt/web-data
[root@www web-data]# tree
.
├── html
│   └── test.html
└── image
    └── test.jpg

2 directories, 2 files
[root@www web-data]#
```



#### 6.2配置文件

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat087385ade40cec2078ebb17ced4505c2.jpg)

#### 6.3测试

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChatae4458672cc1ef8c21fd8fb9498be5e7.jpg)

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat3b713ede4d7bb3ebce5fb6e556131a34.jpg)







### 七、虚拟主机

#### 7.1基于域名虚拟主机配置

> 需要建立/opt/www /opt/bbs目录，windows本地hosts添加虚拟机ip地址对应的域名解析；对应域名网站目录下新增index.html文件；

```cobol
    server {
      listen      80;
      server_name www.lijie.com;
      location / {
          root      /opt/www;
          index     index.html index.htm;
      }
    }

    server {
      listen      80;
      server_name www.chenjie.com;
      location / {
          root      /opt/bbs;
          index     index.html index.htm;
      }
    }
```

测试

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat13dd43ae75245757eaa4126beca6ba97.jpg)





#### 7.2基于端口虚拟主机配置

```
    server {
      listen      192.168.31.29:81;
      server_name _;
      location / {
          root      /opt/www;
          index     index.html index.htm;
      }
    }

    server {
      listen      192.168.31.29:82;
      server_name _;
      location / {
          root      /opt/bbs;
          index     index.html index.htm;
      }
    }
```

测试：

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat3577b08f9cba17fb0a0e7214d9d62202.jpg)



也可以用相同域名，不同端口去设置

```
    server {
      listen      81;
      server_name www.lijie.com;
      location / {
          root      /opt/www;
          index     index.html index.htm;
      }ssss
    }

    server {
      listen      82;
      server_name www.lijie.com;
      location / {
          root      /opt/bbs;
          index     index.html index.htm;
      }
    }
```

测试：

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat55856d6158ccef0634a2916f76d55f62.jpg)







#### 7.3基于ip虚拟主机配置

先临时给网卡加多一个ip

```
ip addr add 192.168.31.30/24 dev ens33
[root@www ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:10:ec:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.29/24 brd 192.168.31.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet 192.168.31.30/24 scope global secondary ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::b272:7e4e:3769:1a4d/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

配置文件

```
    server {
      listen      192.168.31.29:80;
      server_name _;
      location / {
          root      /opt/www;
          index     index.html index.htm;
      }
    }

    server {
      listen      192.168.31.30:80;
      server_name _;
      location / {
          root      /opt/bbs;
          index     index.html index.htm;
      }
    }
```

测试

![](/Users/zhouqiting/Desktop/document/nginx/nginx学习/WeChat3645ff246ceae190076c1456dbb95326.jpg)







