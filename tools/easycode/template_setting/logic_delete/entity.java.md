```
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