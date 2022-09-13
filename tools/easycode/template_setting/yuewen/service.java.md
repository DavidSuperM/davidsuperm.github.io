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
import $!{tableInfo.savePackageName}.component.ServiceResult;
import $!{tableInfo.savePackageName}.component.MessageResult;
import $!{tableInfo.savePackageName}.component.LogServiceAspect;
import $!{tableInfo.savePackageName}.entity.dto.$!{tableInfo.name}ListDTO;
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
    ServiceResult<$!{entityName}> getById($!pk.shortType $!pk.name);
    
    /**
     * 查询多条数据
     *
     * @param currentPage 当前页
     * @param pageSize 一页大小
     * @return 
     */
    ServiceResult<$!{tableInfo.name}ListDTO> getAllByPage(int currentPage, int pageSize);
    
    /**
     *  条件+分页查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 条件
     * @param currentPage 当前页
     * @param pageSize 一页大小
     * @return 
     */
    ServiceResult<$!{tableInfo.name}ListDTO> getAllByPageCondition($!{entityName} $!tool.firstLowerCase($!{entityName}), int currentPage, int pageSize);

    /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 
     */
    ServiceResult<Object> insert($!{entityName} $!tool.firstLowerCase($!{entityName}));
    
     /**
     * 批量新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName})List 实例对象
     * @return 
     */
    ServiceResult<Object> insertList(List<$!{entityName}> $!tool.firstLowerCase($!{entityName})List);

    /**
     * 修改数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 
     */
    ServiceResult<Object> update($!{entityName} $!tool.firstLowerCase($!{entityName}));

    /**
     * 通过主键删除数据
     *
     * @param $!pk.name 主键
     * @return 
     */
    ServiceResult<Object> deleteById($!pk.shortType $!pk.name);

}
```