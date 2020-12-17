## mysql开启慢查询
#### mysql查看慢查询是否开启和日志位置
```
show variables like 'slow_query%';
```

#### 开启慢查询和设置日志位置和慢查询的时间阈值
```
将 slow_query_log 全局变量设置为“ON”状态
mysql> set global slow_query_log='ON'; 

设置慢查询日志存放的位置
mysql> set global slow_query_log_file='/usr/local/mysql/data/slow.log';

查询超过1秒就记录
mysql> set global long_query_time=1;

测试
select sleep(2);
```