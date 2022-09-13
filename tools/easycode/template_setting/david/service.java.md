```
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
    $!{entityName} getById($!pk.shortType $!pk.name);
    
     /**
     * 得到总数
     *
     * @return 总数
     */
    int getTotalNumber();

    /**
     * 查询多条数据
     *
     * @param from 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    List<$!{entityName}> getAllByLimit(int from, int limit);
    
    /**
     *  条件查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 条件
     * @return 对象列表
     */
    List<$!{entityName}> getAllByCondition($!{entityName} $!tool.firstLowerCase($!{entityName}));
    
    /**
     *  条件+分页查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 条件
     * @param from 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    List<$!{entityName}> getAllByConditionLimit($!{entityName} $!tool.firstLowerCase($!{entityName}), int from, int limit);

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
     * @return 实例对象
     */
    int insertList(List<$!{entityName}> $!tool.firstLowerCase($!{entityName})List);

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