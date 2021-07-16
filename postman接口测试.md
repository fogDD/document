### postman接口测试

#### 1.登录区块链平台下载应用列表的应用ID的证书！

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713111727141.png" alt="image-20210713111727141"  />

### 

#### 2.打开postman配置证书，选择settings，创建add证书

![6fd87000c122d8f7ee00a572f3b34ef](C:\Users\ADMINI~1\AppData\Local\Temp\WeChat Files\6fd87000c122d8f7ee00a572f3b34ef.png)

![image-20210713112140846](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713112140846.png)



![image-20210713112425560](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713112425560.png)



#### 3.填写https://service-gateway:30004并导入区块链平台下载id的key、cert证书



![image-20210713113045277](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713113045277.png)

#### 4.打开off，导入ca

![image-20210713114120696](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713114120696.png)

#### 5.哈希上链请求

导入区块链的csg（upload Files），把在区块链平台创建的用户id中的id、key替换postman里面的id、key

![image-20210713115431528](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713115431528.png)



![image-20210713115703776](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713115703776.png)

![image-20210713165927234](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713165927234.png)

send之后输出的accessid将作为后续查询请求的需要的参数

#### 6.哈希查询请求

![image-20210713170213826](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210713170213826.png)

查询请求accessid填写刚刚哈希上链请求输出的accessid
