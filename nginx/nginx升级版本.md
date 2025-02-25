### nginx升级版本

### 1.下载nginx包

#### 下载好后解压

```
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

```
tar -zxvf nginx-1.16.1.tar.gz
```

记住现在版本make的config信息

```
[root@localhost nginx-1.16.1]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/TWS
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --user=nginx --group=nginx
```

./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --user=nginx --group=nginx



### 2.更换新版本nginx

#### 在当前新版本的目录中make新nginx版本

```
[root@localhost nginx-1.16.1]# ll
total 752
drwxr-xr-x. 6 1001 1001   4096 Dec 20 08:06 auto
-rw-r--r--. 1 1001 1001 296463 Aug 13  2019 CHANGES
-rw-r--r--. 1 1001 1001 452171 Aug 13  2019 CHANGES.ru
drwxr-xr-x. 2 1001 1001    168 Dec 20 08:06 conf
-rwxr-xr-x. 1 1001 1001   2502 Aug 13  2019 configure
drwxr-xr-x. 4 1001 1001     72 Dec 20 08:06 contrib
drwxr-xr-x. 2 1001 1001     40 Dec 20 08:06 html
-rw-r--r--. 1 1001 1001   1397 Aug 13  2019 LICENSE
drwxr-xr-x. 2 1001 1001     21 Dec 20 08:06 man
-rw-r--r--. 1 1001 1001     49 Aug 13  2019 README
drwxr-xr-x. 9 1001 1001     91 Dec 20 08:06 src
[root@localhost nginx-1.16.1]# ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --user=nginx --group=nginx
checking for OS
 + Linux 3.10.0-1127.el7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
checking for gcc -pipe switch ... found
checking for -Wl,-E switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
checking for gcc builtin 64 bit byteswap ... found
checking for unistd.h ... found
checking for inttypes.h ... found
checking for limits.h ... found
checking for sys/filio.h ... not found
checking for sys/param.h ... found
checking for sys/mount.h ... found
checking for sys/statvfs.h ... found
checking for crypt.h ... found
checking for Linux specific features
checking for epoll ... found
checking for EPOLLRDHUP ... found
checking for EPOLLEXCLUSIVE ... not found
checking for O_PATH ... found
checking for sendfile() ... found
checking for sendfile64() ... found
checking for sys/prctl.h ... found
checking for prctl(PR_SET_DUMPABLE) ... found
checking for prctl(PR_SET_KEEPCAPS) ... found
checking for capabilities ... found
checking for crypt_r() ... found
checking for sys/vfs.h ... found
checking for poll() ... found
checking for /dev/poll ... not found
checking for kqueue ... not found
checking for crypt() ... not found
checking for crypt() in libcrypt ... found
checking for F_READAHEAD ... not found
checking for posix_fadvise() ... found
checking for O_DIRECT ... found
checking for F_NOCACHE ... not found
checking for directio() ... not found
checking for statfs() ... found
checking for statvfs() ... found
checking for dlopen() ... not found
checking for dlopen() in libdl ... found
checking for sched_yield() ... found
checking for sched_setaffinity() ... found
checking for SO_SETFIB ... not found
checking for SO_REUSEPORT ... found
checking for SO_ACCEPTFILTER ... not found
checking for SO_BINDANY ... not found
checking for IP_TRANSPARENT ... found
checking for IP_BINDANY ... not found
checking for IP_BIND_ADDRESS_NO_PORT ... found
checking for IP_RECVDSTADDR ... not found
checking for IP_SENDSRCADDR ... not found
checking for IP_PKTINFO ... found
checking for IPV6_RECVPKTINFO ... found
checking for TCP_DEFER_ACCEPT ... found
checking for TCP_KEEPIDLE ... found
checking for TCP_FASTOPEN ... found
checking for TCP_INFO ... found
checking for accept4() ... found
checking for eventfd() ... found
checking for int size ... 4 bytes
checking for long size ... 8 bytes
checking for long long size ... 8 bytes
checking for void * size ... 8 bytes
checking for uint32_t ... found
checking for uint64_t ... found
checking for sig_atomic_t ... found
checking for sig_atomic_t size ... 4 bytes
checking for socklen_t ... found
checking for in_addr_t ... found
checking for in_port_t ... found
checking for rlim_t ... found
checking for uintptr_t ... uintptr_t found
checking for system byte ordering ... little endian
checking for size_t size ... 8 bytes
checking for off_t size ... 8 bytes
checking for time_t size ... 8 bytes
checking for AF_INET6 ... found
checking for setproctitle() ... not found
checking for pread() ... found
checking for pwrite() ... found
checking for pwritev() ... found
checking for sys_nerr ... found
checking for localtime_r() ... found
checking for clock_gettime(CLOCK_MONOTONIC) ... found
checking for posix_memalign() ... found
checking for memalign() ... found
checking for mmap(MAP_ANON|MAP_SHARED) ... found
checking for mmap("/dev/zero", MAP_SHARED) ... found
checking for System V shared memory ... found
checking for POSIX semaphores ... not found
checking for POSIX semaphores in libpthread ... found
checking for struct msghdr.msg_control ... found
checking for ioctl(FIONBIO) ... found
checking for struct tm.tm_gmtoff ... found
checking for struct dirent.d_namlen ... not found
checking for struct dirent.d_type ... found
checking for sysconf(_SC_NPROCESSORS_ONLN) ... found
checking for sysconf(_SC_LEVEL1_DCACHE_LINESIZE) ... found
checking for openat(), fstatat() ... found
checking for getaddrinfo() ... found
checking for PCRE library ... found
checking for PCRE JIT support ... found
checking for OpenSSL library ... found
checking for zlib library ... found
creating objs/Makefile

Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
  [root@localhost nginx-1.16.1]# make
<前面省略make的内容省略>
sed -e "s|%%PREFIX%%|/usr/local/nginx|" \
        -e "s|%%PID_PATH%%|/usr/local/nginx/logs/nginx.pid|" \
        -e "s|%%CONF_PATH%%|/usr/local/nginx/conf/nginx.conf|" \
        -e "s|%%ERROR_LOG_PATH%%|/usr/local/nginx/logs/error.log|" \
        < man/nginx.8 > objs/nginx.8
make[1]: Leaving directory `/root/nginx-1.16.1'
##看到这个/root/nginx-16.1的目录就没问题
[root@localhost nginx-1.16.1]# make
```

**make完记得不可以make install**



#### 备份nginx二进制启动文件

```
[root@localhost nginx-1.16.1]# cd /usr/local/nginx/sbin/
[root@localhost sbin]# ll
total 4896
-rwxr-xr-x. 1 root root 5012344 Dec  6 03:57 nginx
[root@localhost sbin]# mv nginx nginx.old
[root@localhost sbin]# ll
total 4896
-rwxr-xr-x. 1 root root 5012344 Dec  6 03:57 nginx.old
[root@localhost sbin]# ps -ef | grep nginx
root      69411      1  0 06:48 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx     69424  69411  0 06:56 ?        00:00:00 nginx: worker process
root      72479  72441  0 08:20 pts/1    00:00:00 grep --color=auto nginx
```



#### 替换nginx二进制文件

```
[root@localhost nginx-1.16.1]# pwd
/root/nginx-1.16.1       
[root@localhost nginx-1.16.1]# cp objs/ng
nginx               nginx.8             ngx_auto_config.h   ngx_auto_headers.h  ngx_modules.c       ngx_modules.o       
[root@localhost nginx-1.16.1]# cp objs/nginx /usr/local/nginx/sbin/
[root@localhost nginx-1.16.1]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --user=nginx --group=nginx
[root@localhost nginx-1.16.1]# 
```



### 3.nginx更新完检查

##### 检查新版本nginx配置文件状态

```
[root@localhost nginx-1.16.1]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```



#### 平滑重启nginx

```
root@localhost nginx-1.16.1]# cat /usr/local/nginx/logs/nginx.pid 
69411
[root@localhost nginx-1.16.1]# ps -ef | grep nginx
root      69411      1  0 06:48 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx     69424  69411  0 06:56 ?        00:00:00 nginx: worker process
root      72489  72441  0 08:26 pts/1    00:00:00 grep --color=auto nginx
[root@localhost nginx-1.16.1]# kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
[root@localhost nginx-1.16.1]# cat /usr/local/nginx/logs/nginx.pid
69411
[root@localhost nginx-1.16.1]# ps -ef | grep nginx
root      69411      1  0 06:48 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx     69424  69411  0 06:56 ?        00:00:00 nginx: worker process
root      72496  72441  0 08:29 pts/1    00:00:00 grep --color=auto nginx
```

