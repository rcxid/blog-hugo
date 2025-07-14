+++
title = "paimon学习笔记"
date = "2025-07-11T19:28:51+08:00"
summary = "flink1.20集成paimon1.0.1"
categories = ["数据开发"]
tags = ["flink", "paimon"]
draft = false
+++

## paimon架构

官方文档：https://paimon.apache.org/docs/1.0/concepts/overview/

![1752237205404.png](https://img.617495.xyz/2025/07/11/687104966e345.png)

## 1、创建paimon catalog

paimon catalog目前支持三种元数据存储，fs catalog：元数据和表数据存储在文件系统、hive catalog：元数据额外保存在hive metastore，支持hive访问表、jdbc catalog：元数据额外存储在mysql、postgresql等关系型数据库，前两者比较常用。paimon底层存储支持多种文件系统，例如本地文件系统，hadoop hdfs，阿里云oss，亚马逊s3等。
### 1.1、fs catalog

使用S3存储paimon表数据的话，需要下载依赖jar包放到flink下的lib目录，下载地址可以参考官方文档：https://paimon.apache.org/docs/1.0/maintenance/filesystems

测试中，我使用minio docker镜像搭建了S3对象存储服务，minio 与 Amazon S3 云存储服务 API 兼容。这里遇到2个问题，第一个是容器内部的时区问题，需要把时区调整到上海，可以通过添加环境变量TZ: Asia/Shanghai解决，第二个是s3.endpoint必须写ip，写hostname会超时连接不上。

```sql
-- 存储在hdfs
create catalog paimon_hdfs with (
  'type' = 'paimon',
  'warehouse' = 'hdfs://cluster1:8020/data/paimon/fs'
);

-- 存储在s3对象存储
create catalog paimon_s3 with (
  'type' = 'paimon',
  'warehouse' = 's3://data/paimon',
  's3.endpoint' = 'http://192.168.8.200:9000',
  's3.access-key' = 'xxx',
  's3.secret-key' = 'xxx'
);
```

### 1.2、hive catalog

使用hive catalog，需要下载依赖jar包放到flink下的lib目录，可以通过下面的链接下载，下面链接对应的hive版本是3.1.3，flink版本是1.20.2。

https://mvnrepository.com/artifact/org.apache.flink/flink-sql-connector-hive-3.1.3_2.12/1.20.2

遇到的问题：hive paimon的zstd-jni依赖冲突问题，导致使用hive查询paimon表时报错。paimon使用的zstd-jni版本为：1.5.5-11，hive3.1.3使用的zstd-jni版本为：1.3.2-2。使用下面第二个链接中的jar替换一下，查询报错问题就解决了。

https://github.com/apache/paimon/blob/release-1.0/pom.xml

https://mvnrepository.com/artifact/com.github.luben/zstd-jni/1.5.5-11

```sql
create catalog paimon_hive with (
  'type' = 'paimon',
  -- 指定metastore为hive
  'metastore' = 'hive',
  -- 不设置的话，默认使用hive配置: hive.metastore.warehouse.dir目录
  'warehouse' = 'hdfs://cluster1:8020/data/paimon/hive'
);
```
### 1.3、jdbc catalog


```sql
create catalog paimon_jdbc with (
  'type' = 'paimon',
  'metastore' = 'jdbc',
  'uri' = 'jdbc:postgresql://<host>:<port>/<databaseName>',
  'jdbc.user' = '...',
  'jdbc.password' = '...',
  'catalog-key' = 'jdbc',
  'warehouse' = 'hdfs://cluster1:8020/data/paimon/jdbc'
);
```

## 2、创建paimon表

paimon支持下面的表类型：
- 主键表
- 非主键表/追加表
- 视图表
- 格式表：与hive兼容
- 对象表：（暂时不知道啥样的）
- 物化表

### 2.1、主键表

```sql
use catalog paimon_hdfs;
create database if not exists paimon_tutorial;
use paimon_tutorial;

create table user_table (
  user_id string primary key not enforced,
  name string,
  age int
) with (
  'bucket' = '1'
);
```
### 2.2、非主键表

在batch模式下支持：DELETE、UPDATE、MERGE-INTO操作。

```sql
create table my_table (
  a int,
  b string
);
```

### 2.3、视图表

需要catalog metastore支持。测试hive catalog支持，fs catalog和jdbc catalog不支持。不支持的话，只能使用临时视图，只在当前session有效。

```sql
create view if not exists my_table_view
  (a, b)
as select a, b from my_table;

drop view if exists my_table_view;

show views;

show create view my_table_view;
```

### 2.4、格式表

需要catalog metastore支持，hive catalog支持，hdfs catalog不支持，jdbc catalog没去测试。hdfs上表中的数据和hive表一致。没有paimon表中的schema、snapshot等数据。支持csv、orc、parquet格式数据。

```sql
use catalog paimon_hive;
create database if not exists paimon_test;
use paimon_test;

create table my_csv_table (
  a int,
  b string
) with (
  'type' = 'format-table',
  'file.format' = 'csv',
  'field-delimiter' = ','
);

create table my_orc_table (
  a int,
  b string
) with (
  'type' = 'format-table',
  'file.format' = 'orc'
);

create table my_parquet_table (
  a int,
  b string
) with (
  'type' = 'format-table',
  'file.format' = 'parquet'
);
```

### 2.5、对象表

用于存储非结构化数据，例如，图片、视频、音乐等文件。

```sql
create table `my_object_table` with (
  'type' = 'object-table',
  'object-location' = 'hdfs://cluster1:8020/data/object'
);

-- 刷新对象表
call sys.refresh_object_table('default.my_object_table');

-- 查询对象表
select * from my_object_table;
select * from my_object_table /*+ OPTIONS('scan.snapshot-id' = '1') */;
```

对象表格式如下：

|           name |                type |  null | key | extras | watermark |
|----------------|---------------------|-------|-----|--------|-----------|
|           path |              STRING | FALSE |     |        |           |
|    parent_path |              STRING | FALSE |     |        |           |
|           name |              STRING | FALSE |     |        |           |
|         length |              BIGINT | FALSE |     |        |           |
|          mtime |    TIMESTAMP_LTZ(3) |  TRUE |     |        |           |
|          atime |    TIMESTAMP_LTZ(3) |  TRUE |     |        |           |
|          owner |              STRING |  TRUE |     |        |           |
|     generation |                 INT |  TRUE |     |        |           |
|   content_type |              STRING |  TRUE |     |        |           |
|  storage_class |              STRING |  TRUE |     |        |           |
|       md5_hash |              STRING |  TRUE |     |        |           |
| metadata_mtime |    TIMESTAMP_LTZ(3) |  TRUE |     |        |           |
|       metadata | MAP<STRING, STRING> |  TRUE |     |        |           |

### 2.6、物化表

```sql
create materialized table continuous_users_shops
partitioned by (ds)
freshness = interval '30' second
as select
  user_id,
  ds,
  sum (payment_amount_cents) as payed_buy_fee_sum,
  sum (1) as pv
from (
  select user_id, order_created_at as ds, payment_amount_cents
  from json_source
) as tmp
group by user_id, ds;
```
