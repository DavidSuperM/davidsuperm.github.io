# maven编译打包过滤文件
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
        <excludes>
            <exclude>**/Application.java</exclude>
            <exclude>com/kedacom/kapsdatacollection/scene/*</exclude>
        </excludes>
    </configuration>
</plugin>
```

注意这种方式只适j合maven编译打包，idea直接运行是无效的，猜测idea运行不是通过maven的方式来编译打包