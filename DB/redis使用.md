## redis数据类型：

## redis-cli客户端操作
```
进入客户端
redis-cli 
keys *
del 1
flushdb  // 删除当前库的所有key

字符串操作
set 1 qqq
get 1

hash操作

list操作

set操作
smembers key

zset有序集合操作
zrange key 0 1    //查询索引0-1的数据，前闭后闭
```

## springboot使用redis
1. 加入依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. application.properties加入redis配置
```
# REDIS (RedisProperties)
# Redis数据库索引（默认为0）
spring.redis.database=0
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接超时时间（毫秒）
spring.redis.timeout=0
```

2. 新建配置文件RedisConfig.java，否则 
- 用redisTemplate存入的key会显示乱码，字符串前面会多出 \xac\xed\x00\x05t\x00 这种字符串。但是用代码读没问题，就是用redis-cli客户端查询时显示的像乱码
原因：spring-data-redis的RedisTemplate<K, V>模板类在操作redis时默认使用JdkSerializationRedisSerializer来进行序列化
- 不能保存对象

```
package com.example.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;


/**
 * @author david
 */
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory redisConnectionFactory) {
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // 使用jsckson的序列化
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        //  template.setKeySerializer(new StringRedisSerializer()); 这样配置了，用stringRedisTemplate.set("1","qqq"),再用redisTemplate.get("1")会抛异常
        //  template.setKeySerializer(jackson2JsonRedisSerializer); 这样配置了，redis-cli客户端查询时所有key和value字符串显示为 "\"key\""  且查询方使用new         、、StringRedisSerializer()来序列化key的话是查不到这个key的
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

    @Bean
    @ConditionalOnMissingBean(StringRedisTemplate.class)
    public StringRedisTemplate stringRedisTemplate(
            RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }



//    /**
//     * 使用自定义的序列化方式
//     * @param connectionFactory
//     * @return
//     */
//    @Bean
//    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
//        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
//        redisTemplate.setConnectionFactory(connectionFactory);
//        redisTemplate.setKeySerializer(new StringRedisSerializer());
//        redisTemplate.setValueSerializer(new StringRedisSerializer());
//        // 为了让redis的value可以存入对象
//        redisTemplate.setValueSerializer(new RedisObjectSerializer());
//        return redisTemplate;
//    }

}

```

也可以在对应的serviceImpl构造函数单独指定序列化方式
```
@Autowired
    public IpWhiteListServiceImpl(IpWhiteListMapper ipWhiteListMapper, RedisTemplate redisTemplate){
        this.ipWhiteListMapper = ipWhiteListMapper;
        this.redisTemplate = redisTemplate;
        this.redisTemplate.setKeySerializer(new StringRedisSerializer());
        this.redisTemplate.setValueSerializer(new StringRedisSerializer());
    }
```

3. 使用
```
package com.example.service.impl;

import com.example.component.ServiceResult;
import com.example.entity.UserInfoDO;
import com.example.service.RedisService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

/**
 * @author david
 */
@Service
public class RedisServiceImpl implements RedisService {

    private StringRedisTemplate stringRedisTemplate;
    private RedisTemplate redisTemplate;

    @Autowired
    public RedisServiceImpl(StringRedisTemplate stringRedisTemplate, RedisTemplate redisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
        this.redisTemplate = redisTemplate;
    }

    @Override
    public ServiceResult<UserInfoDO> getNameById(Long id) {
        stringRedisTemplate.opsForValue().set("1","qdw1");
        String res1 = stringRedisTemplate.opsForValue().get("1");

        redisTemplate.opsForValue().set("2","qqq");
        String res2 = (String)redisTemplate.opsForValue().get("2");
        
        // zset可用于分页，参数是key，value，得分
        redisTemplate.opsForZSet().add("2","z1", 10);
        redisTemplate.opsForZSet().add("2","z2", 11);
        (Set<String>) list = (Set<String>)redisTemplate.opsForZSet().range("2",0,1);
        
        // 保存对象
        UserInfoDO infoDO = UserInfoDO.builder()
                .age(18)
                .name("eee")
                .build();
        redisTemplate.opsForValue().set("3",infoDO);
        UserInfoDO userInfoDO = (UserInfoDO) redisTemplate.opsForValue().get("3");
        return ServiceResult.createSuccessResult();
    }
}

```

4. 为什么有 RedisTemplate StringRedisTemplate 两个类操作

RedisTemplate 对五种数据结构分别定义了操作，如下所示：
操作字符串：redisTemplate.opsForValue()
操作 Hash：redisTemplate.opsForHash()
操作 List：redisTemplate.opsForList()
操作 Set：redisTemplate.opsForSet()
操作 ZSet：redisTemplate.opsForZSet()
但是对于 string 类型的数据，Spring Boot 还专门提供了 StringRedisTemplate 类，而且官方也建议使用该类来操作 String 类型的数据。那么它和 RedisTemplate 又有啥区别呢？
RedisTemplate 是一个泛型类，而 StringRedisTemplate 不是，后者只能对键和值都为 String 类型的数据进行操作，而前者则可以操作任何类型。
两者的数据是不共通的，StringRedisTemplate 只能管理 StringRedisTemplate 里面的数据，RedisTemplate 只能管理 RedisTemplate 中 的数据。


