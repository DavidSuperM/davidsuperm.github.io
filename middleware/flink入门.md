# mac安装flink
### 安装命令
```
1. 需要先安装java
2. brew install apache-flink
```

### 查看flink版本和安装位置
```
3. flink -v     // flink --version
4. brew info apache-flink
/usr/local/Cellar/apache-flink/1.5.0
```
 
### 启动关闭flink
```
cd /usr/local/Cellar/apache-flink/1.11.2/libexec/bin/
./start-cluster.sh
./stop-cluster.sh
```

### webui界面
<http://localhost:8081/>

# linux安装flink
下载安装包解压：<https://flink.apache.org/downloads.html>

# 待做的
1. flink样例程序
2. flink idea运行
3. linux运行jar命令

# checkpoint savepoint配置位置
```
which flink
cd ~/flink
cd conf
cat flink-conf.yaml
state.savepoints.dir: hdfs://YWBDOwnerHDFS/YWBDFlink/savepoints
```