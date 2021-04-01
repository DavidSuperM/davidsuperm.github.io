```
$!variable 
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Mapper"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/mapper"))

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

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}mapper;

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
    @ResultMap(value="resultMap")
    @Select("select $!{columnList} from $!tableInfo.obj.name where $!pk.obj.name = #{$!pk.name} and status!=-1 ")
    $!{entityName} getById(@Param("$!pk.name") $!pk.shortType $!pk.name);
    
     /**
     * 得到总条数
     *
     * @return 总条数
     */
    @Select("select count(*) from $!tableInfo.obj.name where status!=-1")
    int getTotalNumber();

    /**
     * 查询指定行数据
     *
     * @param from 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    @ResultMap(value="resultMap")
    @Select("select $!{columnList} from $!tableInfo.obj.name where status!=-1 limit #{from},#{limit}  ")
    List<$!{entityName}> getAllByLimit(@Param("from") int from, @Param("limit") int limit);
    
     /**
     * 通过实体作为筛选条件查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 对象列表
     */
    @ResultMap(value="resultMap")
    @Select("<script>" + 
        "select $!{columnList} from $!tableInfo.obj.name where 1=1 and status!=-1" + 
        #foreach($column in $tableInfo.fullColumn)
        "<if test='$!column.name != null #if($column.type.equals("java.lang.String")) and $!column.name != \"\" #end '> and $!column.obj.name = #{$!column.name} </if>" +
        #end
        "</script>")
    List<$!{entityName}> getAllByCondition($!{entityName} $!tool.firstLowerCase($!{entityName}));

    /**
     * 分页+筛选查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @param from 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    @ResultMap(value="resultMap")
    @Select("<script>" + 
        "select $!{columnList} from $!tableInfo.obj.name where 1=1 and status!=-1" + 
        #foreach($column in $tableInfo.fullColumn)
        "<if test='$!tool.firstLowerCase($!{entityName}).$!column.name != null and $!tool.firstLowerCase($!{entityName}).$!column.name != \"\"'> and $!column.obj.name = #{$!tool.firstLowerCase($!{entityName}).$!column.name} </if>" +
        #end
        " limit #{from},#{limit}"+
        "</script>")
    List<$!{entityName}> getAllByConditionLimit(@Param("$!tool.firstLowerCase($!{entityName})")$!{entityName} $!tool.firstLowerCase($!{entityName}), @Param("from")int from, @Param("limit")int limit);

    
    /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 影响行数
     */
    @Insert("insert into $!{tableInfo.obj.name}(#foreach($column in $tableInfo.otherColumn)$!column.obj.name#if($velocityHasNext), #end#end) "
    + "values (#foreach($column in $tableInfo.otherColumn)#{$!{column.name}}#if($velocityHasNext), #end#end)")
    int insert($!{entityName} $!tool.firstLowerCase($!{entityName}));
    
    /**
     * 批量新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName})List 实例对象
     * @return 影响行数
     */
    @Insert("<script> " + 
            "insert into $!{tableInfo.obj.name}(#foreach($column in $tableInfo.otherColumn)$!column.obj.name#if($velocityHasNext), #end#end) " + 
            "values " + 
            "<foreach collection='$!tool.firstLowerCase($!{entityName})List' item='item' index='index' separator=','>" +
            "(#foreach($column in $tableInfo.otherColumn)#{item.$!{column.name}}#if($velocityHasNext), #end#end)" +
            "</foreach>" +
            "</script>")
    int insertList(@Param("$!tool.firstLowerCase($!{entityName})List") List<$!{entityName}> $!tool.firstLowerCase($!{entityName})List);

    /**
     * 修改数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 影响行数
     */
    @Update("<script> "+
    "update $!{tableInfo.obj.name} set " +
    #foreach($column in $tableInfo.otherColumn)
    "<if test='$!column.name != null #if($column.type.equals("java.lang.String")) and $!column.name != \"\" #end '> $!column.obj.name = #{$!column.name},</if>"+
    #end
    "$!pk.obj.name = #{$!pk.name} where $!pk.obj.name = #{$!pk.name}" +
    "</script>")
    int update($!{entityName} $!tool.firstLowerCase($!{entityName}));

    /**
     * 通过主键删除数据
     *
     * @param $!pk.name 主键
     * @return 影响行数
     */
    @Delete("update $!{tableInfo.obj.name} set status=-1 where $!pk.obj.name = #{$!pk.name}")
    int deleteById(@Param("$!pk.name") $!pk.shortType $!pk.name);
    
}
```