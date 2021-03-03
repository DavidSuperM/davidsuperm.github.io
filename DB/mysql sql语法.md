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
所以要保持有序，手动加上order by 

## 
select 大字段会导致慢，因为数据大，可能站多个数据快，io次数就多



