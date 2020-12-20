
## springboot测试类
springboot版本是2.2.3
base测试类，其他测试类只要继承这个类就行
junit5是这几个注解，和junit4不一样
@ExtendWith(SpringExtension.class) 在spring boot2.1之前才需要
（在Spring boot 2.1.x之后，我们查看@SpringBootTest 的代码会发现，其中已经组合了@ExtendWith(SpringExtension.class)，因此，无需在进行该注解的使用了。）
```
package com.example.utils;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.context.web.WebAppConfiguration;

@SpringBootTest
public class TestApplication {

    @BeforeEach
    public void init() {
        System.out.println("开始测试-----------------");
    }

    @AfterEach
    public void after() {
        System.out.println("测试结束-----------------");
    }
}
```
