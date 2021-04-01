# 1.存储引擎
Innodb

# 2.字符集
utf-8

# 3.设计约定
### 3.1主键
1. 主键为bigint(20)型，防止溢出
2. 主键自增

## 3.2外键
1. 数据库表禁止主外键关联，需要在程序业务逻辑中维护

## 3.3表名
1. 表明字段名小写，形如xx_xx

## 3.4禁用字段
1. timest/**/amp：不同MySQL版本及不同配置参数会表现不一样的行为，同时存在潜在的因时区转换导致的性能问题，可用datetime类型替代
2. bit ：写入、读取及对比bit类型字段存在安全隐患，可用整型tinyint替代
3. float、double ：存在数值精度、数值运算正确性方面的隐患，可用decimal类型替代
4. enum、set ：存在性能、变更方面的隐患、可用整型、字符串类型替

## 3.5必要字段
1. version
原因：数据库字段要加 version  记录更新版本。防止两个人同时在原有基础上新增10块钱导致数据不对，所以正常更新的时候需要set version = version +1，如果需要根据之前数据库的数据来加数据，则需要set money = money+10, version = version +1 where id =#{id} and version = #{version} 保证version和数据库里的版本一致。

# 4.数据库类型，java类型，jdbctype对照
数据库类型|java类型|jdbctype
--|:--:|--:
varchar(64) | String | VARCHAR
tinyint(1) | Boolean | TINYINT
tinyint(4) | Integer | INTEGER
int(11) | Integer | INTEGER
bigint(20) | Long | BIGINT/INTEGER
datetime |java.util.Date | TIMESTAMP

# 小数类型用decimal
【强制】小数类型为 decimal，禁止使用 float 和 double。
说明：在存储的时候，float 和 double 都存在精度损失的问题，很可能在比较值的时候，得到不正确的
结果。如果存储的数据范围超过 decimal 的范围，建议将数据拆成整数和小数并分开存储。

用法：
decimal(a,b)
参数说明
a指定指定小数点左边和右边可以存储的十进制数字的最大个数，最大精度38，默认10
b指定小数点右边可以存储的十进制数字的最大个数。小数位数必须是从 0 到 a之间的值。默认小数位数是 0。

# tinyint
tinyInt(1) 只用来代表Boolean含义的字段，且0代表False，1代表True。（springboot会自动把数据库里tinyint(1)的类型转成代码里boolean类型）
如果要存储多个数值，则定义为tinyInt(N), N>1。例如 tinyInt(2)