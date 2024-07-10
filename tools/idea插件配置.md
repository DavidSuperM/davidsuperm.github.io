# idea插件配置

### 安装的插件列表
1. SonarLint     // 检查语法，优化代码
2. python       
3. Lombok
4. Free MyBatis plugin     // dao跳转到xml
5. EasyCode     // 自动生成mapper entity等代码，并可以编辑模板
6. BashSupport
7. Alibaba Java Coding plugin support   
8. .ignore
9. maven helper  // 查看maven依赖树，解决冲突

### EasyCode配置 
使用教程：<https://zhuanlan.zhihu.com/p/106849549?utm_source=qq&utm_medium=social&utm_oi=753897324492632064&utm_content=first>
用完后 maven reload generate 才会显示出生成的文件
自定义配置：
1. easycode-MybatisCodeHelper->作者->david
2. Type Mapper ->  添加： 

|  columnType   | javaType |
|----|----|
| bigint(\\(\\d+\\))? unsigned  | java.lang.Long |
| tinyint  | java.lang.Integer |
| tinyint(\\(\\d+\\))?  | java.lang.Integer |
| date  | java.time.LocalDate |

> 看源码的话注意上面的斜杠 \\ 是加了转义的

Type Mapper ->  修改： 

|  columnType   | javaType |
|----|----|
| datetime  | java.time.LocalDateTime |
| timestamp  | java.time.LocalDateTime |

3. Global Config -> 添加：
variable         

#set($entityName = $tool.append($tableInfo.name, "DO"))


4. Template Setting -> 
参考 easycode 文件夹下


5. generate code时配置package和path
![](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202020-11-25%20%E4%B8%8B%E5%8D%8811.29.57.png)

6. 文件需要手动add 
git add . 或 选中文件  option+command+a


### 另外几种自动生成增删改查的方法
使用脚本生成注解sql：<https://blog.csdn.net/wang845252276/article/details/103292071>

mybatis-generator：<https://zhuanlan.zhihu.com/p/37636458?utm_source=qq&utm_medium=social&utm_oi=753897324492632064&utm_content=first>
<https://www.jianshu.com/p/a247c6f5cebf>
不使用的原因：还要配置xml文件，加dependency依赖，如果要定制化，比如tinyint默认映射成java的boolean，如果想映射成Integer，还要加代码，继承一个类，对原来的业务代码侵入性太高，每个项目都要加xml文件和依赖，太麻烦

mybatis-plus
不使用的原因：还要手动写entity类，mapper是继承BaseMapper，只是省略了mapper的crud方法，其他的entity,service都要自己手写

另两个自动生成的项目:<https://springboot.plus/guide/generator.html#_2-%E4%BD%BF%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%99%A8%E7%94%9F%E6%88%90curd%E4%BB%A3%E7%A0%81>
<https://gitee.com/flying-cattle/mybatis-dsc-generator>

### idea注释变成竖线
file->setting->editor->general->apperance->取消Render documentation comments