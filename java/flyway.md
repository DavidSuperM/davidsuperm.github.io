#flyway

### 作用
记录数据库表结构和数据更新记录，使别人clone代码后，一键运行即可有相同的表结构和数据。

### pom
```
<dependency>
      <groupId>org.flywaydb</groupId>
      <artifactId>flyway-core</artifactId>
 </dependency>    
```

### 配置
```
spring:
  flyway:
    # 启用或禁用 flyway
    enabled: true
    # flyway 的 clean 命令会删除指定 schema 下的所有 table, 生产务必禁掉。这个默认值是 false 理论上作为默认配置是不科学的。
    clean-disabled: true
    # SQL 脚本的目录,多个路径使用逗号分隔 默认值 classpath:db/migration
    locations: classpath:db/migration
    #  metadata 版本控制信息表 默认 flyway_schema_history
    table: flyway_schema_history
    # 如果没有 flyway_schema_history 这个 metadata 表， 在执行 flyway migrate 命令之前, 必须先执行 flyway baseline 命令
    # 设置为 true 后 flyway 将在需要 baseline 的时候, 自动执行一次 baseline。
    baseline-on-migrate: true
    # 指定 baseline 的版本号,默认值为 1, 低于该版本号的 SQL 文件, migrate 时会被忽略
    baseline-version: 1
    # 字符编码 默认 UTF-8
    encoding: UTF-8
    # 是否允许不按顺序迁移 开发建议 true  生产建议 false
    out-of-order: false
    # 需要 flyway 管控的 schema list,这里我们配置为flyway  缺省的话, 使用spring.datasource.url 配置的那个 schema,
    # 可以指定多个schema, 但仅会在第一个schema下建立 metadata 表, 也仅在第一个schema应用migration sql 脚本.
    # 但flyway Clean 命令会依次在这些schema下都执行一遍. 所以 确保生产 spring.flyway.clean-disabled 为 true
    schemas: flyway
    # 执行迁移时是否自动调用验证   当你cu'r的 版本不符合逻辑 比如 你先执行了 DML 而没有 对应的DDL 会抛出异常
    validate-on-migrate: true
```

### 文件位置
在resources资源文件下新建db.migration文件夹，下面放sql文件，里面记录了sql脚本。命名类似
```
V2__V1.0.0_RELEASE_KAWF_INIT.sql
V3__V1.0.0_RELEASE_KAPS_DYNAMIC_CONFIG.sql
```
每次只能添加文件，不能修改之前的文件数据

### sql文件命名规则
待更新

### 运行
#### 1.maven运行
pom配置
```
<plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
    <version>${flyway-maven-plugin.version}</version>
    <configuration>
        <table>schema_version</table>
        <validateOnMigrate>true</validateOnMigrate>
        <baselineOnMigrate>true</baselineOnMigrate>
        <user>${db.username}</user>
        <password>${db.password}</password>
        <driver>${db.driver}</driver>
        <url>${db.url}</url>
        <locations>
            <location>db/migration</location>
        </locations>
    </configuration>
</plugin>
```
然后在idea右侧maven中找到plugin->flyway,点击migrate

#### 2.继承在springboot中，每次启动项目自动运行
增加配置类即可
```
import org.flywaydb.core.Flyway;
import org.springframework.beans.factory.annotation.Autowire;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Flyway 启动配置类
 */
@Configuration
public class KapsFlywayConfig {

    @Value("${flyway.table:schema_version}")
    private String table;

    @Value("${flyway.baselineOnMigrate:true}")
    private Boolean baselineOnMigrate;

    @Value("${flyway.locations:classpath:/db/origin}")
    private String[] locations;
	
    @ConditionalOnProperty(name = "flyway.kaps.enabled", havingValue = "true")
    @Bean(initMethod = "migrate", autowire = Autowire.BY_TYPE)
    public Flyway getFlyway() {
        Flyway flyway = new Flyway();
        flyway.setBaselineOnMigrate(baselineOnMigrate);
        flyway.setTable(table);
        flyway.setLocations(locations);
        return flyway;
    }

}
```

#### 3.代码main运行
没有测试过
```
public class FlywayTest {
	public static void main(String[] args) {
		String url = "jdbc:mysql://127.0.0.1:3306/flyway?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&rewriteBatchedStatements=true&useSSL=false&serverTimezone=GMT%2B8";
		String user = "root";
		String password = "tiger";
		Flyway flyway = Flyway.configure().dataSource(url, user, password).load();
		// 创建 flyway_schema_history 表
//		flyway.baseline();
		// 删除 flyway_schema_history 表中失败的记录
//		flyway.repair();
		// 检查 sql 文件
//		flyway.validate();
		// 执行数据迁移
		flyway.migrate();
		// 删除当前 schema 下所有表
//		flyway.clean();
	}
}
```

