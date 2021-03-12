# springboot配置单数据源

application.properties文件配置
```
#将database_name改成自己的数据库的名称
spring.datasource.url=jdbc:mysql://localhost:3306/test_db?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.username=root
spring.datasource.password=qianwei
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

# 输出mysql sql日志，打印sql
logging.level.com.example.dao=debug

## Mybatis 配置  xml-sql的形式就要配置这个 否则会报BindingException: Invalid bound statement (not found) 错误
#mybatis.typeAliasesPackage=com.example.entity
mybatis.mapperLocations=classpath:mapper/*.xml
```

# springboot配置多数据源

1. application.properties文件配置
注意.url变成了.jdbc-url
```
spring.datasource.db1.jdbc-url=jdbc:mysql://localhost:3306/test_db1?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.db1.username=root
spring.datasource.db1.password=qianwei
spring.datasource.db1.driver-class-name=com.mysql.jdbc.Driver

spring.datasource.db2.jdbc-url=jdbc:mysql://localhost:3306/test_db2?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.db2.username=root
spring.datasource.db2.password=qianwei
spring.datasource.db2.driver-class-name=com.mysql.jdbc.Driver
```

2. 启动类Applicaiton.java加上
```
@SpringBootApplication(exclude = {
        DataSourceAutoConfiguration.class
})
```

因为DataSourceAutoConfiguration会读取application.properties文件的spring.datasource.*属性并自动配置单数据源

3 创建数据源配置 文件
DB1Config.java
```
package com.example.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;

/**
 * @author david
 */
@Configuration
@MapperScan(basePackages = "com.example.mapper.db1", sqlSessionTemplateRef = "defaultSqlSessionTemplate")
public class DB1Config {

    @Bean(name = "originalDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.db1")
    @Primary
    public DataSource originalDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "defaultSqlSessionFactory")
    @Primary
    public SqlSessionFactory defaultSqlSessionFactory() throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(originalDataSource());
        org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
        configuration.setMapUnderscoreToCamelCase(true);
        bean.setConfiguration(configuration);
        // 如果是xml配置的mybatis，就需要下面这一行指定xml位置，否则报Invalid bound statement错误
        // factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/db1/*.xml"));
        // 如果是注解mapper就配置下面的
        bean.getObject().getConfiguration().addMappers("com.example.mapper.db1");
        return bean.getObject();
    }

    @Bean(name = "defaultSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate defaultSqlSessionTemplate(@Qualifier("defaultSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}

```


DB2Config.java
```
package com.example.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;

/**
 * @author david
 */
@Configuration
@MapperScan(basePackages = "com.example.mapper.db2", sqlSessionTemplateRef = "defaultSqlSessionTemplate")
public class DB2Config {

    @Bean(name = "originalDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.db2")
    @Primary
    public DataSource originalDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "defaultSqlSessionFactory")
    @Primary
    public SqlSessionFactory defaultSqlSessionFactory() throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(originalDataSource());
        org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
        configuration.setMapUnderscoreToCamelCase(true);
        bean.setConfiguration(configuration);
        bean.getObject().getConfiguration().addMappers("com.example.mapper.db2");
        return bean.getObject();
    }

    @Bean(name = "defaultSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate defaultSqlSessionTemplate(@Qualifier("defaultSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}

```


### 另外一种创建数据源配置类的方式(没有测试)
3. 创建DataSourceConfig.java文件
(手动创建数据源)
```
@Configuration
public class DataSourceConfig {

    @Bean(name = "db1")
    @ConfigurationProperties(prefix = "spring.datasource.db1")
    public DataSource DB1DataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "db2")
    @ConfigurationProperties(prefix = "spring.datasource.db2")
    public DataSource DB2DataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

创建Db1Config.java文件
(分别配置不同数据源的mybatis的SqlSessionFactory
 这样做可以让我们的不同包名底下的mapper自动使用不同的数据源)
```
@Configuration
@MapperScan(basePackages = {"com.example.mapper.db1"}, sqlSessionFactoryRef = "sqlSessionFactoryDb1")
public class Db1Config {

    @Autowired
    @Qualifier("db1")
    private DataSource dataSourceDb1;


    @Bean
    public SqlSessionFactory sqlSessionFactoryDb1() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceDb1);
        // 如果是xml配置的mybatis，就需要下面这一行指定xml位置，否则报Invalid bound statement错误
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/db1/*.xml"));
        // 如果是注解mapper就配置下面的
        // factoryBean.getObject().getConfiguration().addMappers("com.example.mapper.db1");
        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplateDb1() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactoryDb1());
    }

}
```

创建Db2Config.java文件
```
@Configuration
@MapperScan(basePackages = {"com.example.mapper.db2"}, sqlSessionFactoryRef = "sqlSessionFactoryDb2")
public class Db2Config {

    @Autowired
    @Qualifier("db2")
    private DataSource dataSourceDb2;

    @Bean
    public SqlSessionFactory sqlSessionFactoryDb2() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceDb2);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/db2/*.xml"));
        // factoryBean.getObject().getConfiguration().addMappers("com.example.mapper.db2");
        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplateDb2() throws Exception {
        return new SqlSessionTemplate(sqlSessionFactoryDb2());
    }

}

```



