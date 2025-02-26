## mysql常见问题

### 1.ERROR 1071 (42000): Specified key was too long; max key length is 767 bytes

错误是指超出索引字节的限制，并不是指字段长度限制。在官方文档“Limits on InnoDB Tables”有关于这方面的介绍、描述（详情请见参考资料）：

如果启用了系统变量innodb_large_prefix（默认启用，注意实验版本为MySQL 5.6.41,默认是关闭的，MySQL 5.7默认开启），则对于使用DYNAMIC或COMPRESSED行格式的InnoDB表，索引键前缀限制为3072字节。如果禁用innodb_large_prefix，则对于任何行格式的表，索引键前缀限制为767字节。

 

innodb_large_prefix将在以后的版本中删除、弃用。在MySQL 5.5中引入了innodb_large_prefix，用来禁用大型前缀索引，以便与不支持大索引键前缀的早期版本的InnoDB兼容。

对于使用REDUNDANT或COMPACT行格式的InnoDB表，索引键前缀长度限制为767字节。例如，您可能会在TEXT或VARCHAR列上使用超过255个字符的列前缀索引达到此限制，假设为utf8mb3字符集，并且每个字符最多包含3个字节。

 

尝试使用超出限制的索引键前缀长度会返回错误。要避免复制配置中出现此类错误，请避免在主服务器上启用enableinnodb_large_prefix（如果无法在从服务器上启用）。



适用于索引键前缀的限制也适用于全列索引键。

注意：上面是767个字节，而不是字符，具体到字符数量，这就跟字符集有关。GBK是双字节的，UTF-8是三字节的

 **解决方案：**

```
mysql> show variables like '%innodb_large_prefix%';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| innodb_large_prefix | OFF   |
+---------------------+-------+
1 row in set (0.00 sec)
 
mysql> set global innodb_large_prefix=on;
Query OK, 0 rows affected (0.00 sec)

mysql> set global innodb_file_format=Barracuda;
Query OK, 0 rows affected (0.00 sec)

mysql> show table status from MyDB where name='TEST'\G;
*************************** 1. row ***************************
           Name: TEST
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 16384
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2018-09-20 13:53:49
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: 
        Comment: 
 
mysql>  ALTER TABLE TEST ROW_FORMAT=DYNAMIC;
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0
 
mysql> show table status from MyDB where name='TEST'\G;
*************************** 1. row ***************************
           Name: TEST
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 16384
      Data_free: 0
 Auto_increment: NULL
    Create_time: 2018-09-20 14:04:05
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options: row_format=DYNAMIC
        Comment: 
1 row in set (0.00 sec)
 
ERROR: 
No query specified
 
mysql> ALTER TABLE TEST MODIFY CODE_VALUE1 VARCHAR(350);
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

如该表未创建，则在建表语句中加入

```
CREATE TABLE `****` () ENGINE=InnoDB ROW_FORMAT=Dynamic;
```

