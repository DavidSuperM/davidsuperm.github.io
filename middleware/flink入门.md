# flink入门

因为时间关系，本篇写的有点乱，不详细。

1. 示例代码
参考官网 <https://ci.apache.org/projects/flink/flink-docs-release-1.11/try-flink/datastream_api.html>

1.0 flink程序有3种运行方式。
第一种是idea本地运行，方便调试。

第二种是maven package 生成jar包 提交到flinkui界面，然后在ui界面操作运行，项目启动日志在 Job Manager->Log list->什么什么.log
项目代码里打的日志在 Tasks Managers->Log list->什么什么.log
默认启动的main类在pom里指定了
```
<plugin>
...
<transformers>
    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
        <mainClass>com.david.job.UserScoreJob</mainClass>
    </transformer>
</transformers>
...
</plugin>
```
或者也可以在submit job时 手动指定main类

第三种是 hadoop jarn模式，同样是提交jar包的形式，具体没深入了解了



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


## mac安装flink
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


### checkpoint savepoint配置位置
```
which flink
cd ~/flink
cd conf
cat flink-conf.yaml
state.savepoints.dir: hdfs://YWBDOwnerHDFS/YWBDFlink/savepoints
```

### flink架构整理
yarn flink模式
一个clinet端，用于提交任务，提交后，会有一个jobmanage启动flink集群，分发任务，
jobmanage启动若干个taskmanage来执行任务


### 自定义 testjob
TestJob.java
```
package com.david.job;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.yuewen.entity.UserBehaviorInfo;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

/**
 * @author david
 */
@Slf4j
public class TestJob {
    public static void main(String[] args) throws Exception {
        log.debug("---user_score info in TestJob main");
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<String> s1 = env.socketTextStream("localhost", 9000, "\n");
        // 使用Flink算子对输入流的文本进行操作
        // 按空格切词、计数、分组、设置时间窗口、聚合
        DataStream windowCounts =
                s1
                .flatMap(new FlatMapFunction<String, Tuple3<String, String, Integer>>() {
                    @Override
                    public void flatMap(String value, Collector<Tuple3<String, String, Integer>> out) {
                        log.debug("---user_score flatmap = {}", value);
                        UserBehaviorInfo userBehaviorInfo = null;
                        ObjectMapper objectMapper = new ObjectMapper();
                        try {
                            userBehaviorInfo = objectMapper.readValue(value, UserBehaviorInfo.class);
                        } catch (JsonProcessingException e) {
                            e.printStackTrace();
                        }
                        int bookCount = 1;
                        int chapterCount = userBehaviorInfo.getChapterInfoList().size();
                        Long id = userBehaviorInfo.getYwguid();
                        out.collect(Tuple3.of(id.toString(), "bookCount", 1));
                        out.collect(Tuple3.of(id.toString(), "chapterCount", userBehaviorInfo.getChapterInfoList().size()));
                    }
                })
                        .keyBy(0, 1)
                        .timeWindow(Time.seconds(10))
                        .sum(2)
                        .process(new ProcessFunction<Tuple3<String, String, Integer>, Object>() {
                            @Override
                            public void processElement(Tuple3<String, String, Integer> tuple3, Context context, Collector<Object> collector) throws Exception {
                                log.debug("---user_score ");
                                log.debug("---user_score stringStringIntegerTuple3 = "+tuple3);

                            }
                            });
        windowCounts.print().setParallelism(1);
        env.execute("flink learning connectors kafka2");
    }

    @Data
    public static class SubCount{
        private Long id;
        private Integer bookCount;
        private Integer chapterCount;
    }


}

```

### 本地运行
本地先起一个端口：
nc -l 9000
然后启动TestJob
在本地端口输入数据，手动制造数据源
{"appId":1,"areaId":2,"userIp":"127.0.0.1","ywguid":10301,"cbid":777,"chapterInfoList":[{"ccid":45957379233659101,"chapterCreateTime":1603696853801}],"behaviorType":5,"behaviorTime":1603696853805}
本地启动需要在 configurations勾选  include dependencies with provided scope
