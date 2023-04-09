# maven repository仓库远程请求依赖流程
1. pom文件定义了远程仓库位置

```
<repositories>
    <repository>
      <id>maven-net-cn</id>
      <name>Maven China Mirror</name>
      <url>http://maven.net.cn/content/groups/public/</url>
      <releases>
        <enabled>true</enabled>
      </releases>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>
  <pluginRepositories>
    <pluginRepository>
      <id>maven-net-cn</id>
      <name>Maven China Mirror</name>
      <url>http://maven.net.cn/content/groups/public/</url>
      <releases>
        <enabled>true</enabled>
      </releases>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>    
    </pluginRepository>
  </pluginRepositories>
```
则maven会去pom中定义的url拉去依赖，正常的依赖去repository中拉取，插件依赖去pluginRepository拉取，如果没有定义pluginRepository，则插件依赖去repository中拉取。这两种，去对应仓库找不到的情况下，会去maven的中心仓库找
pom中的定义只针对该项目。

2. settings.xml文件定义了远程仓库位置
```
<profiles>
    <profile>
      <id>test</id>
      <repositories>
      <repositories />
      <pluginRepositories>
      <pluginRepositories />
    </profile>
</profiles>
```
则maven会去对应repositories的位置拉取依赖，如果同时配置了settings.xml和pom.xml，则先去pom找，找不到再去settings.xml配置的url找？，再找不到去maven的中心仓库找

settings.xml定义镜像
```
<mirrors>
    <mirror>
      <id>maven-net-cn</id>
      <name>Maven China Mirror</name>
      <url>http://maven.net.cn/content/groups/public/</url>
      <mirrorOf>maven-net-cn</mirrorOf>
    </mirror>
  </mirrors>
```
```
<mirrorOf>maven-net-cn</mirrorOf> 
```
会过滤上面pom或settings配置id为maven-net-cn的远程仓库， maven不访问 id为maven-net-cn配置的仓库，转为访问mirror中配置的url，找不到再去maven中心仓库找？
如果
```
<mirrorOf>central</mirrorOf> 
```
过滤中心仓库
```
<mirrorOf>*</mirrorOf> 
```
过滤所有仓库

如果有多个mirror配置
```
<mirrors>
    <mirror>
      <id>maven-net-cn1</id>
      <name>Maven China Mirror1</name>
      <url>http://maven.net.cn/content/groups/public2/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
    <mirror>
      <id>maven-net-c2</id>
      <name>Maven China Mirror2</name>
      <url>http://maven.net.cn/content/groups/public2/</url>
      <mirrorOf>maven-net-cn</mirrorOf>
    </mirror>
  </mirrors>
```
则按顺序匹配第一个，匹配到后但是找不到依赖，不再往下匹配，类似switch case
如上，所有的仓库访问都会被<mirrorOf>*</mirrorOf>过滤，<mirrorOf>maven-net-cn</mirrorOf>实际上不起作用

