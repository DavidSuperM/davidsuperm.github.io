```
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
import $!{tableInfo.savePackageName}.component.ServiceResult;
import $!{tableInfo.savePackageName}.component.MessageResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;


/**
 * $!{tableInfo.comment}($!{tableInfo.name})表控制层
 *
 * @author $!author
 * @since $!time.currTime()
 */
@RestController
@RequestMapping("/$!tool.firstLowerCase($tableInfo.name)")
public class $!{tableName} {
    
    private final $!{tableInfo.name}Service $!tool.firstLowerCase($tableInfo.name)Service;
    
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
    @GetMapping("/selectOne")
    public MessageResult<$!{entityName}> getById(
        @RequestParam("id") $!pk.shortType id
) {
        ServiceResult<$!{entityName}> serviceResult = $!{tool.firstLowerCase($tableInfo.name)}Service.getById(id);
        return MessageResult.create(serviceResult.getStatus(), serviceResult.getMsg(), serviceResult.getData());
    }

}
```