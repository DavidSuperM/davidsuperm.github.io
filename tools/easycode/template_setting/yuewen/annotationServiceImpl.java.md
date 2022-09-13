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
import $!{tableInfo.savePackageName}.entity.dto.$!{tableInfo.name}ListDTO;
import $!{tableInfo.savePackageName}.mapper.$!{tableInfo.name}Mapper;
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import $!{tableInfo.savePackageName}.component.ServiceResult;
import $!{tableInfo.savePackageName}.component.MessageResult;
import $!{tableInfo.savePackageName}.component.LogServiceAspect;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表服务实现类
 *
 * @author $!author
 * @since $!time.currTime()
 */
@Service("$!tool.firstLowerCase($!{tableInfo.name})Service")
public class $!{tableName} implements $!{tableInfo.name}Service {
    
    private final $!{tableInfo.name}Mapper $!tool.firstLowerCase($!{tableInfo.name})Mapper;
    
    @Autowired
    public $!{tableName}($!{tableInfo.name}Mapper $!tool.firstLowerCase($tableInfo.name)Mapper){
        this.$!tool.firstLowerCase($tableInfo.name)Mapper = $!tool.firstLowerCase($tableInfo.name)Mapper;
    }

    /**
     * 通过ID查询单条数据
     *
     * @param $!pk.name 主键
     * @return 实例对象
     */
    @Override
    public ServiceResult<$!{entityName}> getById($!pk.shortType $!pk.name) {
        $!{entityName} $!{tool.firstLowerCase($!{tableInfo.name})} = $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.getById($!pk.name);
        return ServiceResult.createSuccessResult($!{tool.firstLowerCase($!{tableInfo.name})});
    }

    /**
     * 查询多条数据
     *
     * @param currentPage 当前页
     * @param pageSize 一页大小
     * @return 对象列表
     */
    @Override
    public ServiceResult<$!{tableInfo.name}ListDTO> getAllByPage(int currentPage, int pageSize) {
        int count = $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.getTotalNumber();
        List<$!{entityName}> $!{entityName}List =  $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.getAllByLimit((currentPage-1)*pageSize, pageSize);
        $!{tableInfo.name}ListDTO $!{tool.firstLowerCase($!{entityName})}ListDTO = $!{tableInfo.name}ListDTO.builder()
            .count(count)
            .$!{tableInfo.name}List($!{entityName}List)
            .build();
        return ServiceResult.createSuccessResult($!{tool.firstLowerCase($!{entityName})}ListDTO);
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
    public ServiceResult<$!{tableInfo.name}ListDTO> getAllByPageCondition($!{entityName} $!tool.firstLowerCase($!{entityName}), int currentPage, int pageSize) {
        int count = $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.getTotalNumber();
        List<$!{entityName}> $!{entityName}List = $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.getAllByConditionLimit($!tool.firstLowerCase($!{entityName}), (currentPage-1)*pageSize, pageSize);
        $!{tableInfo.name}ListDTO $!{tool.firstLowerCase($!{tableInfo.name})}ListDTO = $!{tableInfo.name}ListDTO.builder()
            .count(count)
            .$!{tableInfo.name}List($!{entityName}List)
            .build();
        return ServiceResult.createSuccessResult($!{tool.firstLowerCase($!{tableInfo.name})}ListDTO);
    }

    /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 
     */
    @Override
    public ServiceResult<Object> insert($!{entityName} $!tool.firstLowerCase($!{entityName})) {
        $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.insert($!tool.firstLowerCase($!{entityName}));
        return ServiceResult.createSuccessResult();
    }
    
    /**
     * 新增数据
     *
     * @param $!tool.firstLowerCase($!{entityName})List 实例对象
     * @return 
     */
    @Override
    public ServiceResult<Object> insertList(List<$!{entityName}> $!tool.firstLowerCase($!{entityName})List) {
        $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.insertList($!tool.firstLowerCase($!{entityName})List);
        return ServiceResult.createSuccessResult();
    }

    /**
     * 修改数据
     *
     * @param $!tool.firstLowerCase($!{entityName}) 实例对象
     * @return 
     */
    @Override
    public ServiceResult<Object> update($!{entityName} $!tool.firstLowerCase($!{entityName})) {
        $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.update($!tool.firstLowerCase($!{entityName}));
        return ServiceResult.createSuccessResult(); 
    }

    /**
     * 通过主键删除数据
     *
     * @param $!pk.name 主键
     * @return 
     */
    @Override
    public ServiceResult<Object> deleteById($!pk.shortType $!pk.name) {
        $!{tool.firstLowerCase($!{tableInfo.name})}Mapper.deleteById($!pk.name);
        return ServiceResult.createSuccessResult(); 
    }
}
```