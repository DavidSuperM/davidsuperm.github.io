# springboot多环境配置
假设有dev，test，prod三个环境，数据库等配置都是不同的
1. application.properties放公共配置
```
server.port=8081
server.servlet.context-path=/springboot-demo
spring.profiles.active=dev
```

2. application-dev.properties,application-test.properties,application-prod.properties分别放对应环境的配置
```
spring.datasource.url=...
...
logging.config=classpath:logback-spring.xml
logging.level.root=info
```

3. 设置启用的配置文件
第一种是设置 spring.profiles.active=dev  （优先级低） 
第二种idea->edit configurations->Active profiles设置为dev (优先级高)
   
4. pom设置profile,目的是打包时只打包指定的配置文件
```
<profiles>
        <profile>
            <id>dev</id>
            <properties>
                <profiles.active>dev</profiles.active>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <profiles.active>test</profiles.active>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <profiles.active>prod</profiles.active>
            </properties>
        </profile>
</profiles>
```
