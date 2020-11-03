# mac安装kafka
```
brew install kafka
```
安装过程将依赖安装 zookeeper

软件位置:
/usr/local/Cellar/zookeeper  
/usr/local/Cellar/kafka

配置文件位置:
/usr/local/etc/kafka/zookeeper.properties
/usr/local/etc/kafka/server.properties

启动kafka
```
cd /usr/local/Cellar/kafka/2.6.0/bin
// 启动zookeeper服务
zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties 
// 启动kafka服务
kafka-server-start /usr/local/etc/kafka/server.properties 
// 创建test1 topic
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test1
// 查看创建的topic
kafka-topics --list --zookeeper localhost:2181  
// 生产数据 
kafka-console-producer --broker-list localhost:9092 --topic test1
// 消费数据
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic test1 --from-beginning
```

