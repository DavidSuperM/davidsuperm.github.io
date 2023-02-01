```
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

import java.util.List;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表服务实现类
 *spring
 * @author $!author
 * @since $!time.currTime()
 */
@Service("$!tool.firstLowerCase($!{tableInfo.name})Service")
public class $!{tableName} implements $!{tableInfo.name}Service {
    
    private final $!{tableInfo.name}Dao $!tool.firstLowerCase($!{tableInfo.name})Dao;
    
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
    public $!{entityName} getById($!pk.shortType $!pk.name) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.getById($!pk.name);
    }
    
    /**
     * 查询总数
     *
     * @return 实例对象
     */
    @Override
    public int getTotalNumber() {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.getTotalNumber();
    }


    /**
     * 查询多条数据
     *
     * @param currentPage 当前页
     * @param pageSize 一页大小
     * @return 对象列表
     */
    @Override
    public List<$!{entityName}> getAllByPage(int currentPage, int pageSize) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.getAllByLimit((currentPage-1)*pageSize, pageSize);
    }
    
    /**
     * 条件查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 条件
     * @return 对象列表
     */
    @Override
    public List<$!{entityName}> getAllByCondition($!{entityName} $!tool.firstLowerCase($!{entityName})) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.getAllByCondition($!tool.firstLowerCase($!{entityName}));
    }
    
    /**
     * 条件+分页查询
     *
     * @param $!tool.firstLowerCase($!{entityName}) 条件
     * @param currentPage 当前页
     * @param pageSize 一页大小
     * @return 对象列表
     */
    @Override
    public List<$!{entityName}> getAllByPageCondition($!{entityName} $!tool.firstLowerCase($!{entityName}), int currentPage, int pageSize) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.getAllByConditionLimit($!tool.firstLowerCase($!{entityName}), (currentPage-1)*pageSize, pageSize);
    }

     /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 影响行数
     */
    @Override
    public int insert($!{entityName} $!tool.firstLowerCase($!{entityName})) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.insert($!tool.firstLowerCase($!{entityName}));
    }
    
    /**
     * 批量新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName})List 实例对象
     * @return 影响行数
     */
    @Override
    public int insertList(List<$!{entityName}> $!tool.firstLowerCase($!{entityName})List) {
        return $!{tool.firstLowerCase($!{tableInfo.name})}Dao.insertList($!tool.firstLowerCase($!{entityName})List);
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
        return this.getById($!{tool.firstLowerCase($!{entityName})}.get$!tool.firstUpperCase($pk.name)());
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