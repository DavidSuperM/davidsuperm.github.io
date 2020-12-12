```
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
    $!{entityName} getById(@Param("$!pk.name")$!pk.shortType $!pk.name);
    
    /**
     * 查询总数
     *
     * @return 总数
     */
    int getTotalNumber();

    /**
     * 查询指定行数据
     *
     * @param from 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    List<$!{entityName}> getAllByLimit(@Param("from") int from, @Param("limit") int limit);


    /**
     * 通过实体作为筛选条件查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 对象列表
     */
    List<$!{entityName}> getAllByCondition($!{entityName} $!tool.firstLowerCase($!{entityName}));
    
    /**
     * 通过实体作为筛选条件查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @param from 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    List<$!{entityName}> getAllByConditionLimit(@Param("$!tool.firstLowerCase($!{entityName})") $!{entityName} $!tool.firstLowerCase($!{entityName}), @Param("from")int from, @Param("limit")int limit);

    /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 影响行数
     */
    int insert($!{entityName} $!tool.firstLowerCase($!{entityName}));
    
    /**
     * 批量新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName})List 实例对象
     * @return 影响行数
     */
    int insertList(@Param("$!tool.firstLowerCase($!{entityName})List") List<$!{entityName}> $!tool.firstLowerCase($!{entityName})List);

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
    int deleteById(@Param("$!pk.name") $!pk.shortType $!pk.name);

}
```