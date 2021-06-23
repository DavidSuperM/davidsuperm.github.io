# 为什么要对表数据进行拆分
答：单表数据量过大时，索引文件会过大，导致B+树高度过高（3层b+树可以索引上百万数据，每多一层就多一次磁盘io）导致查询等操作性能显著降低。（甚至内存并不能放下整个索引文件，导致需要做磁盘io操作，则性能更低)
如果数据只用作备份存储，不查询，则单表容量没有上限。

# 单表数据量到多大时进行拆分
答： mysql单表容量在1000万左右较为合适，即查询效率高，数据再大查询时间会显著下降，具体到多少视数据结构，索引而定，
有些表可能500万就出现性能下降，有些表可能到3000万才会性能下降。

# 数据拆分方案
## 表分区
### 概念介绍
MySQL 数据库在 5.1 版本时添加了对分区（partitioning）的支持。分区的过程是将一个表或索引分解成多个更小、更可管理的部分。就访问数据库的应用而言，从逻辑上来讲，只有一个表或一个索引，但是在物理上这个表或索引可能由数十个物理分区组成。

MySQL 分区功能并不是在存储引擎层完成的，因此不是只有 InnoDB 存储引擎支持分区，常见的存储引擎 MyISAM、NDB 等都支持。

MySQL 数据库支持的分库类型为水平分区（指将同一表中不同行的记录分配到不同的物理文件中），并不支持垂直分区（指将同一表中不同列的记录分配到不同的物理文件中）。

MySQL 数据库的分区是局部分区索引，一个分区中既存放了数据又存放了索引。而全局分区是指，数据存放在各个分区中，但是所有数据的索引放在一个对象中。MySQL 数据库目前不支持全局分区。

MySQL 数据库支持以下几种类型的分区。1 如果表中存在主键/唯一索引时，分区列必须是主键/唯一索引的一个组成部分。2 对于 RANGE、LIST、HASH 和 KEY 这四种分区中，分区的条件是：数据必须是整型，如果不是整型，那应该需要通过函数将其转化为整型，如 YEAR(),TO_DAYS(),MONTH() 等函数。

### 表分区具体操作
1. RANGE分区（常用）
```
create table employees (
    id int not null,
    fname varchar(30),
    lname varchar(30),
    hired date not null default '1970-01-01',
    separated date not null default '9999-12-31',
    job_code int not null,
    store_id int not null
) partition by range (store_id) (
    partition p0 values less than (6),
    partition p1 values less than (11),
    partition p2 values less than (16),
    partition p3 values less than (21)，
    partition p3 values less than maxvalue
);
```

2. LIST分区
```
create table employees (
    id int not null,
    fname varchar(30),
    lname varchar(30),
    hired date not null default '1970-01-01',
    separated date not null default '9999-12-31',
    job_code int not null,
    store_id int not null
) partition by list(store_id)
    partition pNorth values in (3,5,6,9,17),
    partition pEast values in (1,2,10,11,19,20),
    partition pWest values in (4,12,13,14,18),
    partition pCentral values in (7,8,15,16)
)；
```

3. HASH分区
```
create table employees (
    id int not null,
    fname varchar(30),
    lname varchar(30),
    hired date not null default '1970-01-01',
    separated date not null default '9999-12-31',
    job_code int not null,
    store_id int not null
) partition by hash(store_id)
partitions 4；
```

4. KEY分区
5. COLUMNS分区(5.5开始支持,可以直接使用非整形的数据进行分区)

### 分区适用场景
1. 表非常大，或者只是在表的最后部分有热点数据，其他均是历史数据。
2. 分区表的数据更容易维护。例如，想批量删除大量数据可以使用清除整个分区的方式。另外，还可以对一个独立分区进行优化、检查、修复等操作。
3. 分区表的数据分布在不同的物理设备，从而高效的利用多个硬件设备。
4. 使用分区表避免某些特殊的瓶颈，例如InnoDB的单个索引的互斥访问、ext3文件系统的inode锁竞争等、
5. 备份和恢复独立的分区，这在非常大的数据集的场景下更高效。

### 分区优缺点
#### 优点
1. 大表数据分区，查询时优化器根据分区定义，无须扫描所有分区，只查找需要数据的分区
2. 批量删除整个分区十分方便
#### 缺点
1. 一个表最多只能有1024个分区
2. 在分区表上，用于分区表达式里的每一个字段都必须是唯一性索引的一部分(包括主键)
3. 分区表中无法使用外键
4. 分区键不可以为NULL
5. 分区列和索引列不匹配导致无法进行分区过滤
6. 分区仍然是在一个数据库中，受到单库的瓶颈

#### 备注
如果只是数量大（同时1024个分区完全够），但是并发不高，可以用表分区。大公司好像一般都直接分库分表，业务可控


## 分表
### 水平分表
#### 概念
按照某种策略将一个表的数据拆分成多个表。单所有的表仍然在同一个库中
#### 策略
1. range
2. hash
3. 按照地理区域划分
4. 按照时间划分

#### 全局id生成策略
水平分表需要有一个全局唯一的id，推荐用雪花算法，使用分库分表中间件都可以配置的
1. 自动增长列 （设置偏移量和步长，扩容的时候不好解决）
2. UUID （简单唯一，太长无序）
3. Snowflake(雪花) 算法 （64bit  41bit时间+10bit机器id+12bit流水号）
参考<https://github.com/Snailclimb/JavaGuide/blob/master/docs/system-design/micro-service/%E5%88%86%E5%B8%83%E5%BC%8Fid%E7%94%9F%E6%88%90%E6%96%B9%E6%A1%88%E6%80%BB%E7%BB%93.md>

### 垂直分表
按照字段进行拆分，即可以将一个大字段列单独拆分成一个表
优点：数据维护简单
缺点：仍然存在单表数据量过大问题，主键冗余，会引起表join操作

## 分库
mysql单库有性能瓶颈，承受的操作峰值应该在2000左右，超过的话应该分库，将数据拆分到不同的库中。拆分方案和分表类似

## 分库分表后面临的问题
1. 事务支持
2. 多库结果集合并
3. 跨库join

## 分库分表方案
### proxy方案
#### 概念
起一个proxy服务，做一层中间代理，项目上连接mysql的配置改为连接proxy服务
优点：和项目语言无关
缺点：增加了系统的复杂性，需要保证proxy层的稳定性。

#### 方案
1. mycat （阿里出品，由cobar发展而来，支持单库多表或多库单表,支持多个数据库）
2. sharding-sphere的sharding-proxy (apache项目，支持多库多表，只支持mysql)


### client方案
#### 概念
在项目中引入jar包，配置分库分表方案。
优点：操作，维护简单
缺点：和语言强相关，现在一般只有java项目可以用

#### 方案
1. sharding-sphere的sharding-jdbc （当当开源，现在属于apache项目）


### mycat和shading-sphere的对比图
直接看博文：<https://my.oschina.net/u/4318872/blog/4281049>
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/20210526_1_mysql.png)

### 分库分表算法
1. 固定hash算法
2. 一致性哈希算法
3. 虚拟节点 
4. 自定义路由

## 分库分表及平滑扩容详解
<https://juejin.cn/post/6844903648670007310>
<https://www.cnblogs.com/barrywxx/p/11532122.html>
<https://crossoverjie.top/2019/07/24/framework-design/sharding-db-03/>
<https://blog.csdn.net/kefengwang/article/details/81213050>
<https://blog.csdn.net/kefengwang/article/details/81213050>
<https://maxwell.gitbook.io/way-to-architect/shu-ju-ku/mysql/mysqlzhong-de-sharding/fen-ku-fen-biao-hou-de-ping-hua-kuo-rong>


Q:
### 什么时候用单库多库，多库多表
当数据量大，但是并发不大，单词查询时间长，可以用单库多表，因为并发没到单库的极限。
当数据量大并且并发大，用多库多表

### 为什么互联网公司不用表分区
主要是分库分表业务可控
参考<https://www.w3cschool.cn/architectroad/architectroad-mysql-partition-table.html>






