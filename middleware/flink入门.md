# flink入门
1. 示例代码
参考官网 <https://ci.apache.org/projects/flink/flink-docs-release-1.11/try-flink/datastream_api.html>
1.1 新建项目
```
$ mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-walkthrough-datastream-java \
    -DarchetypeVersion=1.11.2 \
    -DgroupId=frauddetection \
    -DartifactId=frauddetection \
    -Dversion=0.1 \
    -Dpackage=spendreport \
    -DinteractiveMode=false
```
1.2 新建文件FraudDetectionJob.java
```
package spendreport;

import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.walkthrough.common.sink.AlertSink;
import org.apache.flink.walkthrough.common.entity.Alert;
import org.apache.flink.walkthrough.common.entity.Transaction;
import org.apache.flink.walkthrough.common.source.TransactionSource;

public class FraudDetectionJob {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Transaction> transactions = env
            .addSource(new TransactionSource())
            .name("transactions");
        
        DataStream<Alert> alerts = transactions
            .keyBy(Transaction::getAccountId)
            .process(new FraudDetector())
            .name("fraud-detector");

        alerts
            .addSink(new AlertSink())
            .name("send-alerts");

        env.execute("Fraud Detection");
    }
}
```
1.3 新建文件FraudDetector.java
```
package spendreport;

import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.walkthrough.common.entity.Alert;
import org.apache.flink.walkthrough.common.entity.Transaction;

public class FraudDetector extends KeyedProcessFunction<Long, Transaction, Alert> {

    private static final long serialVersionUID = 1L;

    private static final double SMALL_AMOUNT = 1.00;
    private static final double LARGE_AMOUNT = 500.00;
    private static final long ONE_MINUTE = 60 * 1000;

    @Override
    public void processElement(
            Transaction transaction,
            Context context,
            Collector<Alert> collector) throws Exception {

        Alert alert = new Alert();
        alert.setId(transaction.getAccountId());

        collector.collect(alert);
    }
}
```


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

flink datastream api 入门示例从参考
<https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/project-configuration.html>


# checkpoint savepoint配置位置
```
which flink
cd ~/flink
cd conf
cat flink-conf.yaml
state.savepoints.dir: hdfs://YWBDOwnerHDFS/YWBDFlink/savepoints
```
