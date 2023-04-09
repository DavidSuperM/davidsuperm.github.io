# mysql唯一约束下的逻辑删除
### 数据库表结构
```
CREATE TABLE `user_info` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `name` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '名字',
  `age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态 -1:删除 0:正常',
  `version` int(11) NOT NULL DEFAULT '0' COMMENT '版本',
  `creator` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '创建人',
  `update_user` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '修改人',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT '1980-01-01 00:00:00' ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `del_uni_key` bigint(20) unsigned NOT NULL default '0' COMMENT '用于唯一约束下的逻辑删除',
  PRIMARY KEY (`id`),
  unique key (name, age, del_uni_key)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户信息表'
```

### 如何逻辑删除带唯一约束的表的数据
1. 表增加一个 del_uni_key 字段，和id类型一样
2. 唯一约束加上 del_uni_key 这个字段
2. 删除时的语句：update user_info set status = -1, del_uni_key = id where id = 2;

