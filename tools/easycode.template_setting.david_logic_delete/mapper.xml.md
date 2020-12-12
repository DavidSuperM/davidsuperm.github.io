```
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
    <select id="getById" resultMap="BaseResultMap">
        select
          <include refid="BaseColumnList"/>
        from $!{tableInfo.obj.parent.name}.$!tableInfo.obj.name
        where $!pk.obj.name = #{$!pk.name} and status!=-1
    </select>
    
     <!--查询总数-->
    <select id="getTotalNumber" resultType="java.lang.Integer">
        select count(*) from $!tableInfo.obj.name where status!=-1
    </select>

    <!--查询指定行数据-->
    <select id="getAllByLimit" resultMap="BaseResultMap">
        select
          <include refid="BaseColumnList"/>
        from $!tableInfo.obj.name
        where status!=-1
        limit #{from}, #{limit}
    </select>

    <!--通过实体作为筛选条件查询-->
    <select id="getAllByCondition" resultMap="BaseResultMap">
        select
          <include refid="BaseColumnList"/>
        from $!{tableInfo.obj.parent.name}.$!tableInfo.obj.name
        <where>
#foreach($column in $tableInfo.fullColumn)
            <if test="$!column.name != null">
                and $!column.obj.name = #{$!column.name}
            </if>
#end
            <if test="1 == 1">
                and status != -1
            </if>
        </where>
    </select>
    
    <!--通过实体作为筛选条件查询-->
    <select id="getAllByConditionLimit" resultMap="BaseResultMap">
        select
          <include refid="BaseColumnList"/>
        from $!{tableInfo.obj.parent.name}.$!tableInfo.obj.name
        <where>
#foreach($column in $tableInfo.fullColumn)
            <if test="$!tool.firstLowerCase($!{entityName}).$!column.name != null">
                and $!column.obj.name = #{$!tool.firstLowerCase($!{entityName}).$!column.name}
            </if>
#end
            <if test="1 == 1">
                and status != -1
            </if>
        </where>
        limit #{from},#{limit}
    </select>

    <!--新增所有列-->
    <insert id="insert" keyProperty="$!pk.name" useGeneratedKeys="true">
        insert into $!{tableInfo.obj.parent.name}.$!{tableInfo.obj.name}(#foreach($column in $tableInfo.otherColumn)$!column.obj.name#if($velocityHasNext), #end#end)
        values (#foreach($column in $tableInfo.otherColumn)#{$!{column.name}}#if($velocityHasNext), #end#end)
    </insert>
    
    <!--批量新增-->
    <insert id="insertList" keyProperty="$!pk.name" useGeneratedKeys="true">
        insert into $!{tableInfo.obj.parent.name}.$!{tableInfo.obj.name}(#foreach($column in $tableInfo.otherColumn)$!column.obj.name#if($velocityHasNext), #end#end)
        values 
        <foreach collection='$!tool.firstLowerCase($!{entityName})List' item='item' index='index' separator=','>
            (#foreach($column in $tableInfo.otherColumn)#{item.$!{column.name}}#if($velocityHasNext), #end#end)
        </foreach>
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
        update $!{tableInfo.obj.parent.name}.$!{tableInfo.obj.name} set status=-1 where $!pk.obj.name = #{$!pk.name}
    </delete>

</mapper>

```