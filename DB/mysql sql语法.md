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
```

- 索引
//todo 添加唯一性约束和唯一索引
```
alter table tableName add index indexName(name)    // 增加普通索引
alter table tableName add unique key [indexName] (username(length))   // 增加唯一约束
alter table tableName drop index indexName;     //删除索引,删除唯一索引，删除唯一约束
```







