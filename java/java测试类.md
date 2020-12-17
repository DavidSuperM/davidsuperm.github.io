
## springboot测试类
base测试类，其他测试类只要继承这个类就行
junit5是这几个注解，和junit4不一样
```
package com.example.utils;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.context.web.WebAppConfiguration;

@ExtendWith(SpringExtension.class)
@SpringBootTest
//由于是Web项目，Junit需要模拟ServletContext，因此我们需要给我们的测试类加上@WebAppConfiguration。
@WebAppConfiguration
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
