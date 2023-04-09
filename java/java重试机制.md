# java重置机制

### 背景
在调用第三方接口或者使用mq时，会出现网络抖动，连接超时等网络异常，所以需要重试。为了使处理更加健壮并且不太容易出现故障，后续的尝试操作，有时候会帮助失败的操作最后执行成功。

### 原理
本质是在一个while循环里，业务execute成功则break,若失败，则sleep几秒，重试次数++,再次execute,当达到重试次数阈值时break

### spring-retry 

1. pom添加依赖
```
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2. 启动类添加注解 @EnableRetry
```
@SpringBootApplication
@MapperScan("com.example.mapper")
@EnableRetry
public class Application {
    ...
}
```

3. 需要重试的方法上添加 @Retryable,默认重试3次，每次间隔1秒
```
// @Retryable(value = RemoteAccessException.class, maxAttempts = 3, backoff = @Backoff(delay = 2000L, multiplier = 1.5))
@Retryable(value = RemoteAccessException.class)
public ServiceResult<UserInfoDO> getById(Long id) throws Exception{
    UserInfoDO userInfo = userInfoMapper.getById(id);
    if(userInfo==null){
        throw new RemoteAccessException("111");
    }
    return ServiceResult.createSuccessResult(userInfo);
}

// 可以不写。作用是用于处理重试结束后仍然抛异常时，调用这个方法，需要保证该方法的异常和 @Retryable 的异常一样
@Recover
public int recover(RemoteAccessException e) {
    log.warn("减库存失败！！！" + LocalTime.now());
    return totalNum;
}
```
> @Retryable的参数说明：
  value：抛出指定异常才会重试
  include：和value一样，默认为空，当exclude也为空时，默认所以异常
  exclude：指定不处理的异常
  maxAttempts：最大重试次数，默认3次
  backoff：重试等待策略，默认使用@Backoff，@Backoff的value默认为1000L，我们设置为2000L；multiplier（指定延迟倍数）默认为0，表示固定暂停1秒后进行重试，如果把multiplier设置为1.5，则第一次重试为2秒，第二次为3秒，第三次为4.5秒。
  
### 配置打印重试日志 
application.properties添加以下内容
```
logging.level.org.springframework.retry.support=debug
```

> 注意调用 @Retryable注解的方法，必须是另外一个spring的bean，因为是通过代理的方式来实现重试的


参考
<https://juejin.cn/post/6844904102468517895>
<https://juejin.cn/post/6844903603862257677>
