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

##### EasyCode配置 
使用教程：<https://zhuanlan.zhihu.com/p/106849549?utm_source=qq&utm_medium=social&utm_oi=753897324492632064&utm_content=first>
用完后 maven reload generate 才会显示出生成的文件
自定义配置：
1. easycode-MybatisCodeHelper->作者->david
2. Type Mapper ->  添加： 

|  columnType   | javaType |
|----|----|
| bigint(\(\d+\))? unsigned  | java.lang.Long |
| tinyint  | java.lang.Integer |
| tinyint(\(\d+\))?  | java.lang.Integer |

3. Global Config -> 添加：
variable         

#set($entityName = $tool.append($tableInfo.name, "DO"))


4. Template Setting -> 
4.1 entity.java
```$xslt
##引入宏定义
$!define
$!variable

##设置回调
$!callback.setFileName($tool.append($entityName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/entity"))

####使用宏定义设置回调（保存位置与文件后缀）
###save("/entity", ".java")

##使用宏定义设置包后缀
#setPackageSuffix("entity")

##使用全局变量实现默认包导入
$!autoImport
import java.io.Serializable;
import lombok.*;

##使用宏定义实现类注释信息
#tableComment("实体类")
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@ToString
public class $!{entityName} implements Serializable {
##    private static final long serialVersionUID = $!tool.serial();
    private static final long serialVersionUID = 1L;
#foreach($column in $tableInfo.fullColumn)
    #if(${column.comment})/**
    * ${column.comment}
    */#end

    private $!{tool.getClsNameByFullName($column.type)} $!{column.name};
#end

###foreach($column in $tableInfo.fullColumn)
####使用宏定义实现get,set方法
###getSetMethod($column)
###end

}
```
4.2 dao.java
```$xslt
$!variable
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Dao"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/dao"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}dao;

import $!{tableInfo.savePackageName}.entity.$!{entityName};
import org.apache.ibatis.annotations.Param;
import org.springframework.stereotype.Repository;
import java.util.List;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表数据库访问层
 *
 * @author $!author
 * @since $!time.currTime()
 */
@Repository
public interface $!{tableName} {

    /**
     * 通过ID查询单条数据
     *
     * @param $!pk.name 主键
     * @return 实例对象
     */
    $!{entityName} queryById($!pk.shortType $!pk.name);

    /**
     * 查询指定行数据
     *
     * @param offset 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    List<$!{entityName}> queryAllByLimit(@Param("offset") int offset, @Param("limit") int limit);


    /**
     * 通过实体作为筛选条件查询
     *
     * @param $!tool.firstLowerCase($!{tableInfo.name}) 实例对象
     * @return 对象列表
     */
    List<$!{entityName}> queryAll($!{entityName} $!tool.firstLowerCase($!{entityName}));

    /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 影响行数
     */
    int insert($!{entityName} $!tool.firstLowerCase($!{entityName}));

    /**
     * 修改数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 影响行数
     */
    int update($!{entityName} $!tool.firstLowerCase($!{entityName}));

    /**
     * 通过主键删除数据
     *
     * @param $!pk.name 主键
     * @return 影响行数
     */
    int deleteById($!pk.shortType $!pk.name);

}
```
4.3 service.java
```$xslt
$!variable

##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Service"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/service"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}service;

import $!{tableInfo.savePackageName}.entity.$!{entityName};
import java.util.List;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表服务接口
 *
 * @author $!author
 * @since $!time.currTime()
 */
public interface $!{tableName} {

    /**
     * 通过ID查询单条数据
     *
     * @param $!pk.name 主键
     * @return 实例对象
     */
    $!{entityName} queryById($!pk.shortType $!pk.name);

    /**
     * 查询多条数据
     *
     * @param offset 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    List<$!{entityName}> queryAllByLimit(int offset, int limit);

    /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 实例对象
     */
    $!{entityName} insert($!{entityName} $!tool.firstLowerCase($!{entityName}));

    /**
     * 修改数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 实例对象
     */
    $!{entityName} update($!{entityName} $!tool.firstLowerCase($!{entityName}));

    /**
     * 通过主键删除数据
     *
     * @param $!pk.name 主键
     * @return 是否成功
     */
    boolean deleteById($!pk.shortType $!pk.name);

}
```
4.4 serviceImpl.java
```$xslt
$!variable

##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "ServiceImpl"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/service/impl"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}service.impl;

import $!{tableInfo.savePackageName}.entity.$!{entityName};
import $!{tableInfo.savePackageName}.dao.$!{tableInfo.name}Dao;
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表服务实现类
 *
 * @author $!author
 * @since $!time.currTime()
 */
@Service("$!tool.firstLowerCase($!{tableInfo.name})Service")
public class $!{tableName} implements $!{tableInfo.name}Service {
    
    private $!{tableInfo.name}Dao $!tool.firstLowerCase($!{tableInfo.name})Dao;
    
    @Autowired
    public $!{tableName}($!{tableInfo.name}Dao $!tool.firstLowerCase($tableInfo.name)Dao){
        this.$!tool.firstLowerCase($tableInfo.name)Dao = $!tool.firstLowerCase($tableInfo.name)Dao;
    }

    /**
     * 通过ID查询单条数据
     *
     * @param $!pk.name 主键
     * @return 实例对象
     */
    @Override
    public $!{entityName} queryById($!pk.shortType $!pk.name) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.queryById($!pk.name);
    }

    /**
     * 查询多条数据
     *
     * @param offset 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    @Override
    public List<$!{entityName}> queryAllByLimit(int offset, int limit) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.queryAllByLimit(offset, limit);
    }

    /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 实例对象
     */
    @Override
    public $!{entityName} insert($!{entityName} $!tool.firstLowerCase($!{entityName})) {
        $!{tool.firstLowerCase($!{tableInfo.name})}Dao.insert($!tool.firstLowerCase($!{entityName}));
        return $!tool.firstLowerCase($!{entityName});
    }

    /**
     * 修改数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 实例对象
     */
    @Override
    public $!{entityName} update($!{entityName} $!tool.firstLowerCase($!{entityName})) {
        $!{tool.firstLowerCase($!{tableInfo.name})}Dao.update($!tool.firstLowerCase($!{entityName}));
        return this.queryById($!{tool.firstLowerCase($!{entityName})}.get$!tool.firstUpperCase($pk.name)());
    }

    /**
     * 通过主键删除数据
     *
     * @param $!pk.name 主键
     * @return 是否成功
     */
    @Override
    public boolean deleteById($!pk.shortType $!pk.name) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.deleteById($!pk.name) > 0;
    }
}
```
4.5 controller.java
```$xslt
$!variable

##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Controller"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/controller"))
##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}controller;

import $!{tableInfo.savePackageName}.entity.$!{entityName};
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表控制层
 *
 * @author $!author
 * @since $!time.currTime()
 */
@RestController
@RequestMapping("$!tool.firstLowerCase($tableInfo.name)")
public class $!{tableName} {
    
    private $!{tableInfo.name}Service $!tool.firstLowerCase($tableInfo.name)Service;
    
    @Autowired
    public $!{tableName}($!{tableInfo.name}Service $!tool.firstLowerCase($tableInfo.name)Service){
        this.$!tool.firstLowerCase($tableInfo.name)Service = $!tool.firstLowerCase($tableInfo.name)Service;
    }

    /**
     * 通过主键查询单条数据
     *
     * @param id 主键
     * @return 单条数据
     */
    @GetMapping("selectOne")
    public $!{entityName} selectOne($!pk.shortType id) {
        return $!{tool.firstLowerCase($tableInfo.name)}Service.queryById(id);
    }

}
```
4.6 mapper.xml
```$xslt
$!variable

##引入mybatis支持
$!mybatisSupport

##设置保存名称与保存位置
$!callback.setFileName($tool.append($!{tableInfo.name}, "Dao.xml"))
$!callback.setSavePath($tool.append($modulePath, "/src/main/resources/mapper"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="$!{tableInfo.savePackageName}.dao.$!{tableInfo.name}Dao">

    <resultMap id="BaseResultMap" type="$!{tableInfo.savePackageName}.entity.$!{entityName}">
        <!--@Table $!{tableInfo.originTableName}-->
#foreach($column in $tableInfo.fullColumn)
        <result property="$!column.name" column="$!column.obj.name" jdbcType="$!column.ext.jdbcType"/>
#end
    </resultMap>
    
    <sql id="BaseColumnList">
        #allSqlColumn()
    </sql>

    <!--查询单个-->
    <select id="queryById" resultMap="BaseResultMap">
        select
          <include refid="BaseColumnList"/>
        from $!{tableInfo.obj.parent.name}.$!tableInfo.obj.name
        where $!pk.obj.name = #{$!pk.name}
    </select>

    <!--查询指定行数据-->
    <select id="queryAllByLimit" resultMap="BaseResultMap">
        select
          <include refid="BaseColumnList"/>
        from $!{tableInfo.obj.parent.name}.$!tableInfo.obj.name
        limit #{offset}, #{limit}
    </select>

    <!--通过实体作为筛选条件查询-->
    <select id="queryAll" resultMap="BaseResultMap">
        select
          <include refid="BaseColumnList"/>
        from $!{tableInfo.obj.parent.name}.$!tableInfo.obj.name
        <where>
#foreach($column in $tableInfo.fullColumn)
            <if test="$!column.name != null#if($column.type.equals("java.lang.String")) and $!column.name != ''#end">
                and $!column.obj.name = #{$!column.name}
            </if>
#end
        </where>
    </select>

    <!--新增所有列-->
    <insert id="insert" keyProperty="$!pk.name" useGeneratedKeys="true">
        insert into $!{tableInfo.obj.parent.name}.$!{tableInfo.obj.name}(#foreach($column in $tableInfo.otherColumn)$!column.obj.name#if($velocityHasNext), #end#end)
        values (#foreach($column in $tableInfo.otherColumn)#{$!{column.name}}#if($velocityHasNext), #end#end)
    </insert>

    <!--通过主键修改数据-->
    <update id="update">
        update $!{tableInfo.obj.parent.name}.$!{tableInfo.obj.name}
        <set>
#foreach($column in $tableInfo.otherColumn)
            <if test="$!column.name != null#if($column.type.equals("java.lang.String")) and $!column.name != ''#end">
                $!column.obj.name = #{$!column.name},
            </if>
#end
        </set>
        where $!pk.obj.name = #{$!pk.name}
    </update>

    <!--通过主键删除-->
    <delete id="deleteById">
        delete from $!{tableInfo.obj.parent.name}.$!{tableInfo.obj.name} where $!pk.obj.name = #{$!pk.name}
    </delete>

</mapper>

```
4.7 debug.json   无改动
```$xslt
//调试表原始对象
$!tool.debug($tableInfo.obj)

//调试列原始对象
$!tool.debug($tableInfo.fullColumn.get(0).obj)

//调试列原始列类型
$!tool.debug($tableInfo.fullColumn.get(0).obj.dataType)

//获取原始列类型中的字段
sqlType = $!tool.getField($tableInfo.fullColumn.get(0).obj.dataType, "typeName")

//执行原始列类型中的方法
sqlTypeLen = $!tableInfo.fullColumn.get(0).obj.dataType.getLength()

```
4.8 annotationMapper.java
```$xslt
$!variable
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Mapper"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/dao"))

#set($i = 0)
#set($size = $tableInfo.fullColumn.size())
#foreach($column in $tableInfo.fullColumn)
    #set($columnList = $tool.append($columnList, $!column.obj.name))
    #set($i=$i+1)
    #if($i<$size)
        #set($columnList = $tool.append($columnList, ","))
    #end
#end

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}dao;

import $!{tableInfo.savePackageName}.entity.$!{entityName};
import org.apache.ibatis.annotations.*;
import org.springframework.stereotype.Repository;
import java.util.List;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表数据库访问层
 *
 * @author $!author
 * @since $!time.currTime()
 */
@Repository
public interface $!{tableName} {

    @Results(id="resultMap", value = {
#foreach($column in $tableInfo.fullColumn)
        @Result(column = "$!column.obj.name", property = "$!column.name"),
#end
    })
    
    /**
     * 通过ID查询单条数据
     *
     * @param $!pk.name 主键
     * @return 实例对象
     */
    @Select("select $!{columnList} from $!{tableInfo.obj.parent.name}.$!tableInfo.obj.name where $!pk.obj.name = #{$!pk.name} ")
    $!{entityName} queryById($!pk.shortType $!pk.name);

    
}
```




#### 另外几种自动生成增删改查的方法
使用脚本生成注解sql：<https://blog.csdn.net/wang845252276/article/details/103292071>
使用xml maven生成：<https://zhuanlan.zhihu.com/p/37636458?utm_source=qq&utm_medium=social&utm_oi=753897324492632064&utm_content=first>
<https://www.jianshu.com/p/a247c6f5cebf>
另两个自动生成的项目:<https://springboot.plus/guide/generator.html#_2-%E4%BD%BF%E7%94%A8%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%E5%99%A8%E7%94%9F%E6%88%90curd%E4%BB%A3%E7%A0%81>
<https://gitee.com/flying-cattle/mybatis-dsc-generator>
