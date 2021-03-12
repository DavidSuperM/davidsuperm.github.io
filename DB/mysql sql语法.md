- 建表
mediumtext 类型不能设置not null default
```
CREATE TABLE `test_info` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `description` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '描述',
  `enabled` tinyint(1) NOT NULL DEFAULT 1 COMMENT '启用状态 0禁用 1启用',
  `status` tinyint(4) NOT NULL DEFAULT 0 COMMENT '状态 -1删除 0正常',
  `page_number` int(11) NOT NULL DEFAULT 0  COMMENT '搜索页数',
  `chapter_content` mediumtext comment '章节内容';   
  `version` int(11) NOT NULL DEFAULT 0  COMMENT '版本号',
  `create_time` timestamp NOT NULL default '1980-01-01 00:00:00' COMMENT '创建时间',
  `update_time` timestamp NOT NULL default '1980-01-01 00:00:00' ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  index [indexName](description(length)),
  UNIQUE KEY [keyName] (`page_number`,`description`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='测试表';

// mysql里唯一约束和唯一索引实际是一样的，唯一约束的就是靠唯一索引实现的。
// 建表时创建唯一约束和唯一索引  unique / unique key / unique index 都行，mysql会自动都转换成unique key
```
- 数据操作
```
select * from test_table;    // 查询
insert into test_table(name, type)
values
('daivd', 1),
('zhangsan', 2);     //插入
update test_table set name = 'li_si' where id = 1;   
delete from test_table where id = 1; 
```

- 表操作
```
drop table tableName    //删除表
alter table tableName add column delay int(11) NOT NULL default -1 comment 'test';  //增加列
alter table tableName drop column columnName     // 删除列
alter table tableName modify column columnName varchar(2048) COLLATE utf8mb4_unicode_ci NOT NULL default '' COMMENT '注释';
```

- 索引
//todo 添加唯一性约束和唯一索引
```
alter table tableName add index indexName(列名)    // 增加普通索引
alter table tableName add index indexName(列名1，列名2，列名3)    // 增加联合索引
alter table tableName add unique key [indexName] (列名(length))   // 增加唯一约束
alter table tableName drop index indexName;     //删除索引,删除唯一索引，删除唯一约束
```

## mysql explain
<https://www.jianshu.com/p/ea3fc71fdc45>
<https://www.cnblogs.com/xuanzhi201111/p/4175635.html>

## explain和limit
```
explain select * from user limit 0,10;
// 会发现rows预计扫描有几百万行（假设user表有几百万数据）
```
当explain遇上limit时，rows的值和limit没有关系，加不加limit，rows值不变。
explain分析的rows行数只和索引有关，来采样预估需要扫描的行数，和limit没有关系。


## mysql查看表数据和索引占用空间大小
```
use information_schema;
// table_schema填DB名，table_name填表名
select table_name, table_rows,data_length,index_length from tables where table_schema = 'test_db' and table_name='pirate_book_chapter_update';
```

## mysql select默认不一定按主键id排序
不加order by的话，select查出数据的顺序和用的索引有关。
如果是select * from user_info; 用全表扫描，不用索引，数据是按主键id顺序查出。
如果是select id from user_info limit 10; 查出数据没按主键id顺序，那我们用explain看下会发现这条语句是用了除主键外的其他索引。
如果没有用任何索引，就是按照物理顺序给出结果。物理顺序不一定是按id排序，因为先插入id 3 4 5，再删除id为4的数据，再插入id 6，则可能id 6的数据插在原来4的位置，则物理顺序就是3 6 5
所以要保持有序，手动加上order by 

## 
select 大字段会导致慢，因为数据大，可能站多个数据快，io次数就多

## 查看sql执行过程和时间（sql执行时间）
1 show profiles; 
2 show variables;查看profiling 是否是on状态； 
3 如果是off，则 set profiling = 1； 
4 执行自己的sql语句； 
5 show profiles；就可以查到sql语句的执行时间；
6.show profile for query Query_ID  // 查看具体执行过程，id是上一个命令可以查到

## 慢日志开启
默认情况下slow_query_log的值为OFF，表示慢查询日志是禁用的。
```
// 开启慢日志
show variables  like '%slow_query_log%';
set global slow_query_log=1;

// 设置超时时间
show variables like 'long_query_time'
set global long_query_time=4

// 永久生效，修改 my.cnf
slow_query_log=1
long_query_time=10
slow_query_log_file=/path/mysql_slow.log
```

## 联合索引
创建联合索引(a,b,c)
select * from mytable where a=3 and b>7 and c=3
这条sql只能用到a，b的索引，用不到c
原理请看联合索引在B+树的结构
<https://blog.csdn.net/weixin_30531261/article/details/79329722>

## select count(*) 会用到索引
表结构
```
 CREATE TABLE `pirate_book_chapter_update` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `book_name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '书名',
  `chapter_name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '章节名',
  `pirate_site_url` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '盗版站url',
  `status` int(11) NOT NULL DEFAULT '0' COMMENT '状态',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `book_site_chapter_unique` (`book_name`,`chapter_name`,`pirate_site_url`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='盗版网站更新书章节表'
```
sql语句
```
explain select count(*) from pirate_book_chapter_update;
```
+----+-------------+----------------------------+------------+-------+---------------+-----------------+---------+------+----------+----------+-------------+
| id | select_type | table                      | partitions | type  | possible_keys | key             | key_len | ref  | rows     | filtered | Extra       |
+----+-------------+----------------------------+------------+-------+---------------+-----------------+---------+------+----------+----------+-------------+
|  1 | SIMPLE      | pirate_book_chapter_update | NULL       | index | NULL          | idx_create_time | 4       | NULL | 77234393 |   100.00 | Using index |
+----+-------------+----------------------------+------------+-------+---------------+-----------------+---------+------+----------+----------+-------------+
可以看到使用了idx_create_time索引，因为主键索引是8字节，timestamp是4字节，占用空间小，统计总数时mysql选择索引列的小的统计
用索引快因为B+树叶节点有向右的指针，直接按右向指针数B+树叶节点的个数就可以了。不用索引的话得去物理块数，物理块数据大，分布散，磁盘IO次数多

# 问题
## 子查询有max或min的函数，外层查询用in，没有走索引，而是全表扫描
#### 表结构
```
 CREATE TABLE `pirate_book_chapter_update` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `book_name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '书名',
  `chapter_name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '章节名',
  `pirate_site_url` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '盗版站url',
  `status` int(11) NOT NULL DEFAULT '0' COMMENT '状态',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `book_site_chapter_unique` (`book_name`,`chapter_name`,`pirate_site_url`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='盗版网站更新书章节表'
```
#### 业务需求
查出'临渊行'这本书在每个盗版站的最新章节名

#### sql解析
- 
```
select pirate_site_url,chapter_name from pirate_book_chapter_update  where id in 
 (select max(id) as id from pirate_book_chapter_update where book_name='临渊行' group by pirate_site_url ) ;
```
explain下
| id | select_type | table                      | partitions | type  | possible_keys            | key                      | key_len | ref   | rows     | filtered | Extra                                                     |
+----+-------------+----------------------------+------------+-------+--------------------------+--------------------------+---------+-------+----------+----------+-----------------------------------------------------------+
|  1 | PRIMARY     | pirate_book_chapter_update | NULL       | All   | NULL                     |                          |         | NULL  | 76966720 |   100.00 |                                 |
|  2 | SUBQUERY    | pirate_book_chapter_update | NULL       | ref   | book_site_chapter_unique | book_site_chapter_unique | 202     | const |    83516 |   100.00 | Using where; Using index; Using temporary; Using filesort |
可以看到外部查询是ALL全表扫描

这是mysql的bug，子查询中用max min 会导致父查询全表扫描 详情见博客 <https://www.jianshu.com/p/df805776c15d>

正确做法：
```
select pirate_site_url,chapter_name from pirate_book_chapter_update where id in 
(select id from (select max(id) as id from pirate_book_chapter_update where book_name='临渊行' group by pirate_site_url ) a);
```







