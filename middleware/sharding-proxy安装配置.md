# sharding-proxy安装配置


### 分库分表几种方案比较
待完成


### sharding-proxy安装配置
1. 下载地址<https://shardingsphere.apache.org/document/current/cn/downloads/>
选择ShardingSphere-Proxy 二进制包: [ TAR ] 下载 .tar.gz格式
解压 tar -zxvf filename.tar.gz

如果连接的是mysql，还要下mysql驱动包,放在sharding-proxy解压后的lib包下
下载地址: <https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.47/mysql-connector-java-5.1.47.jar>
官方文档说明: <https://shardingsphere.apache.org/document/current/cn/quick-start/shardingsphere-proxy-quick-start/>

2. 编辑 conf/server.yaml
```
authentication:
  users:
    root:
      password: root
    sharding:
      password: sharding
      authorizedSchemas: sharding_db
```

3. 编辑conf/config-sharding.yaml 
原表名
以下是单库4表的配置，原表名pirate_book_chapter_update，现拆分成pirate_book_chapter_update_1,pirate_book_chapter_update_2,pirate_book_chapter_update_3,pirate_book_chapter_update_4 四张表
多库的话 ds_0 后面加个ds_1，然后配置defaultDatabaseStrategy和shardingAlgorithms
分片规则时以book_name这个字段的字符串hash后取余，id是雪花算法
```
schemaName: sharding_db
dataSources:
  ds_0:
      url: jdbc:mysql://127.0.0.1:13306/db_name?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&autoReconnect=true
      username: root
      password: root
      connectionTimeoutMilliseconds: 30000
      idleTimeoutMilliseconds: 60000
      maxLifetimeMilliseconds: 1800000
      maxPoolSize: 50
rules:
- !SHARDING
 tables:
   pirate_book_chapter_update:
     actualDataNodes: ds_0.pirate_book_chapter_update_${1..4}
     tableStrategy:
       standard:
         shardingColumn: book_name
         shardingAlgorithmName: t_order_inline
     keyGenerateStrategy:
       column: id
       keyGeneratorName: snowflake
 bindingTables:
   - pirate_book_chapter_update
 defaultDatabaseStrategy:
   none:
 defaultTableStrategy:
   none:
 
 shardingAlgorithms:
   t_order_inline:
     type: INLINE
     props:
       algorithm-expression: pirate_book_chapter_update_${Math.abs(book_name.hashCode())%4+1}
   
 keyGenerators:
   snowflake:
     type: SNOWFLAKE
     props:
       worker-id: 123
```

4. 启动和停止
```
sh bin/start.sh
sh bin/stop.sh
```

5. 连接测试
-h即安装了sharding-proxy的机器的地址，端口默认3307,不加-A可能报错
```
mysql -uroot -proot -h127.0.0.1 -P3307 -A
use sharding_db
show tables;
```
如果是代码的话，业务逻辑不需要动，只要连接数据库的配置的ip，port 用户名密码，改成sharding-proxy的ip port 用户名 密码


5. 问题
如果出现中文乱码问题
```
vim bin/start.sh
将
JAVA_MEM_OPTS=" -server -Xmx2g -Xms2g -Xmn1g -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70"
修改为
JAVA_MEM_OPTS=" -server -Xmx2g -Xms2g -Xmn1g -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 -Dfile.encoding=UTF-8"
```


### maven手动下载依赖 
1. 下载jar包   假设位置在  /Users/david/Downloads/unite-framework-spring-test-0.0.12.jar
2. 查看依赖的信息例如
```
<dependency>
  <groupId>com.qq.dcache</groupId>
  <artifactId>taf.dcache.proxy</artifactId>
  <version>1.0</version>
</dependency>
```
3. 打开idea的terminal窗口
```
mvn install:install-file -DgroupId=com.qq.dcache -DartifactId=taf.dcache.proxy -Dversion=1.0 -Dpackaging=jar -Dfile=/Users/david/Downloads/unite-framework-spring-test-0.0.12.jar
```
4. maven reimport，关闭idea重新打开




