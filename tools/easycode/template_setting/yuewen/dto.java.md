```
##引入宏定义
$!define
$!variable

##设置回调
$!callback.setFileName($tool.append($tableInfo.name, "ListDTO.java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/entity/dto"))

####使用宏定义设置回调（保存位置与文件后缀）
###save("/entity", ".java")

##使用宏定义设置包后缀
#setPackageSuffix("entity.dto")

##使用全局变量实现默认包导入
$!autoImport
import $!{tableInfo.savePackageName}.entity.$!{entityName};
import java.io.Serializable;
import lombok.*;
import java.util.List;

##使用宏定义实现类注释信息
#tableComment("传输类")
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@ToString
public class $!{tableInfo.name}ListDTO implements Serializable {
##    private static final long serialVersionUID = $!tool.serial();
    private static final long serialVersionUID = 1L;
    
    private Integer count;
    
    private List<$!{entityName}> $!{tableInfo.name}List;

}
```