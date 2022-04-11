[TOC]
# Hive外部表挂载数据以及数据删除注意事项

## 挂载数据
准备表格
```sql
-- 不含分区的外部表
create external table if not exists student (
	name string comment '姓名',
	age int comment '年龄',
	score int comment '分数'
) row format delimited fields terminated by '\t'
stored as textfile
location '/tmp/delectAfterTest/user';

-- 含分区的外部表
create external table if not exists student_partition(
	name string comment '姓名',
	age int comment '年龄',
	score int comment '分数'
) row format delimited fields terminated by '\t'
partitioned by(`create_date` string)
stored as textfile
location '/tmp/delectAfterTest/user';
```

准备数据
```sql
[root@nd2 test]# hadoop fs -cat /user/wh/test/a.txt
lucy,20,100
jack,21,90
Rose,22,90
[root@nd2 test]# hadoop fs -cat /user/wh/test/b.txt
Tom,16,100
[root@nd2 test]# hadoop fs -cat /user/wh/test/c.txt
[root@nd2 test]# 
```
### 通过 load data
外部表使用 load data 命令`也会`将数据移动到 表格的 hive,metastore.warehouse.dir 路径下 (不配置, 默认值为/user/hive/warehouse) 
**不含分区的 student 表**
```sql
```

**含分区的 student 表**
```sql
```

### 通过 alert table  location
**不含分区的 student 表**
```sql
--指向某一个文件
alter table test_01 set location '/user/wh/test/a.txt';
hive> select * from test_01;
OK
lucy	20	100
jack	21	90
Rose	22	90

--指向文件夹
alter table test_01 set location '/user/wh/test';
hive> select * from test_01;
OK
lucy	20	100
jack	21	90
Rose	22	90
Tom		16	100
```

**包含分区的 student 表**
```sql
-- 这里指向文件会报错
alter table test_02 add if not exists partition (stat_date='20190101') 
location '/user/wh/test/a.txt';
FAILED: Execution Error, return code 1 
from org.apache.hadoop.hive.ql.exec.DDLTask.
MetaException(message:java.io.IOException:
hdfs:/user/wh/test/a.txt is not a directory or unable to create one)

--指向文件夹
alter table test_02 add if not exists partition (stat_date='20190101') 
location '/user/wh/test/';

hive> select * from test_02;
OK
lucy	20	100	20190101
jack	21	90	20190101
Rose	22	90	20190101
Tom		16	100	20190101
```


## 删除数据
```sql
-- 外部表不能直接使用truncate删除表, 会报错
hive> truncate table test_01;
FAILED: SemanticException [Error 10146]: 
Cannot truncate non-managed table test_01.
```

**不含分区的 student 表**
```sql
--这里我们可以将location指向一个空文件
alter table test_01 set location '/user/wh/test/c.txt';
hive> select * from test_01;
OK
Time taken: 0.079 seconds
```

**含分区的 student 表**
```sql
--直接删除指定分区即可
alter table test_02 drop if exists partition (stat_date='20190101');
hive> select * from test_02;
OK
Time taken: 0.107 seconds
```

## stored as textfile VS stored as sequence
Hive本身支持的文件格式只有：Text File，Sequence File。

如果文件数据是`纯文本`，可以使用 [STORED AS TEXTFILE]。
如果数据`需要压缩`，使用 [STORED AS SEQUENCE] ；

## 参考
[Hive外部表挂载数据以及数据删除注意事项](https://blog.csdn.net/henrrywan/article/details/100115264)  

