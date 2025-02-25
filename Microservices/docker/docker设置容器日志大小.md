# docker设置容器日志大小

新建/etc/docker/daemon.json，若有就不用新建了。添加log-dirver和log-opts参数，如下：（**记得备份配置文件**）

# vim /etc/docker/daemon.json

{
  "registry-mirrors": ["http://f613ce8f.m.daocloud.io"],
  "log-driver":"json-file",
  "log-opts": {"max-size":"500m", "max-file":"3"}
}
max-size=500m，意味着一个容器日志大小上限是500M
max-file=3，意味着一个容器最多有三个日志，分别是：容器id-json.log、容器id-json.log.1、容器id-json.log.2, 当日志文件的大小达到500m时，自动划分文件保存，最多划分3个文件
这两个参数设置之后说明，一个容器最多保存1500m(3 * 500)日志，超过范围的日志不会被保存，文件中保存的是最新的日志，文件会自动滚动更新。
# 重启docker守护进程
systemctl daemon-reload
# 重启docker
systemctl restart docker

注意：设置的日志大小，只对新建的容器有效。