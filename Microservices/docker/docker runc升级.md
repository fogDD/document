### docker runc升级

#### 1.下载安全版本的runc到本地电脑

```
https://github.com/opencontainers/runc/releases/download/v1.0.0-rc95/runc.amd64
```

 [runc.amd64](runc.amd64) 



#### 2.将下载好的runc.amd64文件上传到服务器、修改文件名并赋权

```
mv runc.amd64 runc && chmod +x runc
```



#### 3.备份原有的runc

```
mv /usr/bin/runc /home/runcbak
```



#### 4.替换新版本runc

```
cp runc /usr/bin/runc
```



#### 5.检查runc是否升级成功

```
docker verison
runc -v
```

