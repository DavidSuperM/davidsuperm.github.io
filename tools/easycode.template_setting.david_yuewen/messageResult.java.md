```
##引入宏定义
$!define
$!variable

##设置回调
$!callback.setFileName("MessageResult.java")
$!callback.setSavePath($tool.append($tableInfo.savePath, "/component"))

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}component;

import lombok.Data;

/**
 * @author qiandawei
 */
@Data
public class MessageResult<T> {
    private int status;
    private String msg;
    private T data;

    public MessageResult(){}

    private MessageResult(int status, String msg) {
        this(status, msg, null);
    }

    private MessageResult(int status, String msg, T data) {
        this.status = status;
        this.msg = msg;
        this.data = data;
    }

    public static <I> MessageResult<I> createSuccessResult() {
        return create(0, "成功", null);
    }

    public static <I> MessageResult<I> createSuccessResult(I data) {
        return create(0, "成功", data);
    }

    public static <I> MessageResult<I> createFailResult() {
        return create(-1, "失败");
    }

    public static <I> MessageResult<I> createFailResult(String msg) {
        return create(-1, msg);
    }

    public static <I> MessageResult<I> create(int status, String msg) {
        return new MessageResult<>(status, msg);
    }

    public static <I> MessageResult<I> create(int status, String msg, I data) {
        return new MessageResult<>(status, msg, data);
    }
}

```