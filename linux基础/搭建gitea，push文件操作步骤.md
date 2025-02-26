## gitea操作步骤

#### 1.安装docker-compose

https://github.com/docker/compose/releases



##### 	1.1将下载下来的“**docker-compose-Linux-x86_64**”文件上传到服务器上，然后执行如下命令将其移动到 **/usr/local/bin**，并改名为“docker-compose”

```
mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
```



##### 	1.2执行如下命令添加可执行权限

```
chmod +x /usr/local/bin/docker-compose
```



##### 	1.3使用 **docker-compose -v** 命令测试是否安装成功

```
docker-compose -v
```



#### 2.docker compose安装gitea

docker-compose.yml

```
version: "3.4"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:latest
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=mysql
      - DB_HOST=db:3306
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
    ports:
      - "3000:3000"
      - "222:22"
    depends_on:
      - db

  db:
    image: mysql:5.7.27
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=gitea
      - MYSQL_USER=gitea
      - MYSQL_PASSWORD=gitea
      - MYSQL_DATABASE=gitea
    networks:
      - gitea
    volumes:
      - ./mysql:/var/lib/mysql
```



##### 	2.2安装并运行配置

```
docker-compose up -d
```

1）访问http://IP:3000/

2） 点击登录

3）更新配置：

a、SSH 服务器域名：IP

b、SSH Port:222

c、Gitea URL：http://IP:3000/

d、配置gitea管理员账号密码并点击安装



## gitea把文件导入仓库

#### 1.上传文件push

##### 	1.1初始化仓库

```
git init
```



##### 	1.2把要上传的文件添加到暂存区

```
git add README.md
```

添加之后可以用docker status -s 查看暂存区的文件



##### 	1.3git commit  -m提交暂存区注释

```
git commit -m "first commit"
```



##### 	1.4添加远程版本库：

```
git remote add origin http://192.168.1.104:3000/zkrl/test1.git（远程仓库地址）
```

![](C:\Users\h1172\Desktop\document\微信图片_20210716091615.png)

##### 1.5提交，本地的 master 分支推送到 origin 主机的 master 分支

```
git push -u origin master
```





## gitea创建分支

git分支就是在版本控制过程中，使用多条线同时推进多个任务

一个项目里，主分支master分为几部分分支feature，处理不同的部分，如，一个游戏项目，可以分出搞图像的、搞运算的、操作的等分支。在公司里，每个团队处理属于自己的分支，不同部门分支互不干扰，也不影响主分支，假如某个分支出现严重错误，只需要删除这个分支重新来过，其他分支没影响。当一个分支开发完成，就把他合并到主干中，全部分支开发完后，最终形成整个项目。

然而，主干master也可能出现bug，这个时候就有一个host_fix分支（临时分支，用来修复bug，修复完及时合并为主干）
————————————————
版权声明：本文为CSDN博主「ZS_Wang_Blogs」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_40535588/article/details/90521872

##### 	1.创建分支

```
git branch -v	查看分支
git branch test(分支名)
```



##### 	2.切换分支

```
git checkout 分支名
```



##### 	3.合并简历

 第一步：切换到接收修改的分支上（这里是主干master，假如在test已经增加新内容）
 第二步：执行merge命令+分支名（这里是test）

```
git merge test(需要和合并的分支)
```



##### 	4.删除分支

```
git branch -d （分支名）用于已经合并的分支（未合并会提示错误）
git branch -D (分支名)强制删除分支
```





## gitea删除文件

##### 	1.先删除工作区也就是本地的文件

```
rm xxx
```



##### 	2.查看暂存区（版本库）中的文件依然存在，并未删除

```
git ls-files
```

操作完看状态git status

$ git status
On branch dd
Your branch is up to date with 'origin/dd'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    xxxx

##### 	3.此时删除动作已经加入到暂存区，再执行提交commit动作，那就真正意义上删除了文件

```
git commit -m "delete file"
```



##### 	4.提交操作，本地的 master 分支推送到 origin 主机的你的分支

```
git push -u origin dd
```

