# nginx配置限流

### 1.1 正常限制访问频率

Nginx中我们使用**ngx_http_limit_req_module**模块来限制请求的访问频率，基于漏桶算法原理实现。接下来我们使用 nginx limit_req_zone 和 limit_req 两个指令，限制单个IP的请求处理速率：

key:定义限流对象，binary_remote_addr 是一种key，表示基于 remote_addr(客户端IP) 来做限流，binary_ 的目的是压缩内存占用量。

zone:定义共享内存区来存储访问信息， myRateLimit:10m 表示一个大小为10M，名字为myRateLimit的内存区域。1M能存储16000 IP地址的访问信息，10M可以存储16W IP地址访问信息。

rate:用于设置最大访问速率，rate=10r/s 表示每秒最多处理10个请求。Nginx 实际上以毫秒为粒度来跟踪请求信息，因此 10r/s 实际上是限制：每100毫秒处理一个请求。这意味着，自上一个请求处理完后，若后续100毫秒内又有请求到达，将拒绝处理该请求。

![WeChatb3d05bec3f11eabee4a56ac2a5daaa25](/Users/zhouqiting/Desktop/document/nginx/nginx限流/WeChatb3d05bec3f11eabee4a56ac2a5daaa25.jpg)



### 1.1.1 测试

![WeChat72e46c706bf685badc725cd12cc72ccc](/Users/zhouqiting/Desktop/document/nginx/nginx限流/WeChat72e46c706bf685badc725cd12cc72ccc.jpg)

一秒请求超过两次就会报错





### 1.2 突发流量缓冲

按上面的配置在流量突然增大时，超出的请求将被拒绝，无法处理突发流量，那么在处理突发流量的时候，该怎么处理呢？Nginx提供了 burst 参数来解决突发流量的问题，并结合 nodelay 参数一起使用。burst 译为突发、爆发，表示在超过设定的处理速率后能额外处理的请求数。

burst=5 nodelay表示这5个请求立马处理，不能延迟，相当于特事特办。不过，即使这5个突发请求立马处理结束，后续来了请求也不会立马处理。burst=5 相当于缓存队列中占了5个坑，即使请求被处理了，这5个位置这只能按 100ms一个来释放。这就达到了速率稳定，突发然流量也能正常处理的效果。
![WeChat23b60c35413f8197048edc29a35f6177](/Users/zhouqiting/Desktop/document/nginx/nginx限流/WeChat23b60c35413f8197048edc29a35f6177.jpg)



### 1.2.1测试

我这边设置1r/s，也就是1秒最多处理1个请求，加上nodelay的1个突发请求，正常1秒超过两个请求就会报错无法访问





### 1.3限制并发连接数

Nginx中的ngx_http_limit_conn_module模块提供了限制并发连接数的功能，可以使用limit_conn_zone指令以及limit_conn执行进行配置。接下来我们可以通过一个简单的例子来看下：

![WeChat8b5b98335f4bba8bc56b7ddd8b9ed9e5](/Users/zhouqiting/Desktop/document/nginx/nginx限流/WeChat8b5b98335f4bba8bc56b7ddd8b9ed9e5.jpg)

上面配置了单个IP同时并发连接数最多只能2个连接，并且设置了整个虚拟服务器同时最大并发数最多只能10个链接。当然，只有当请求的header被服务器处理后，虚拟服务器的连接数才会计数。刚才有提到过Nginx是基于漏桶算法原理实现的，实际上限流一般都是基于漏桶算法和令牌桶算法实现的。接下来我们来看看两个算法的介绍：



### 限流算法

#### 漏桶算法

漏桶算法是网络世界中流量整形或速率限制时经常使用的一种算法，它的主要目的是控制数据注入到网络的速率，平滑网络上的突发流量。漏桶算法提供了一种机制，通过它，突发流量可以被整形以便为网络提供一个稳定的流量。也就是我们刚才所讲的情况。漏桶算法提供的机制实际上就是刚才的案例：突发流量会进入到一个漏桶，漏桶会按照我们定义的速率依次处理请求，如果水流过大也就是突发流量过大就会直接溢出，则多余的请求会被拒绝。所以漏桶算法能控制数据的传输速率。

![WeChatc1e899fe6181b83887dbdaa5c9a02581](/Users/zhouqiting/Desktop/document/nginx/nginx限流/WeChatc1e899fe6181b83887dbdaa5c9a02581.jpg)



#### 令牌桶算法

令牌桶算法是网络流量整形和速率限制中最常使用的一种算法。典型情况下，令牌桶算法用来控制发送到网络上的数据的数目，并允许突发数据的发送。Google开源项目Guava中的RateLimiter使用的就是令牌桶控制算法。令牌桶算法的机制如下：存在一个大小固定的令牌桶，会以恒定的速率源源不断产生令牌。如果令牌消耗速率小于生产令牌的速度，令牌就会一直产生直至装满整个令牌桶。![WeChat73f33e0c4e0a2650b1391b5b15fb2354](/Users/zhouqiting/Desktop/document/nginx/nginx限流/WeChat73f33e0c4e0a2650b1391b5b15fb2354.jpg)