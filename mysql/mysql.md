## mysql笔记

[TOC]

##### 1.mysql模糊查找

-- 模糊匹配含有“网”字的数据

```
SELECT * from app_info where appName（字段名） like '%网%';
```

-- 模糊匹配以“网”字结尾的数据

```java
SELECT * from app_info where appName like '%网';
```

-- 精准匹配，appName like '网' 等同于：appName = '网'

```java
SELECT * from app_info where appName = '网';
-- 等同于
SELECT * from app_info where appName like '网';
```



##### 2.binlog设置

binlog 有两种格式，一种是 statement，一种是row。可能你在其他资料上还会看到有第三种格式，叫作 mixed，其实它就是前两种格式的混合。

 

查看当前的格式：

```
show VARIABLES like '%binlog_format%'
```

mysql> SET GLOBAL binlog_format = 'ROW';

然而设置如上的方式不能生效，这是因为这个命令是全局的设置，需要重启后才能生效。

 

如果当前生效的话，可以使用如下的方式：

```
mysql> SET SESSION binlog_format = 'ROW';
```

表示对当前的环境生效，但是一旦重启服务，这个就失效了。



##### 3.快速建表语句

```
CREATE TABLE IF NOT EXISTS `runoob_tbl`(
   `runoob_title` VARCHAR(100) NOT NULL,
   `runoob_author` VARCHAR(40) NOT NULL
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

