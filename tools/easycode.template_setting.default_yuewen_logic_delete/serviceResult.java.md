##引入宏定义
$!define
$!variable

##设置回调
$!callback.setFileName("ServiceResult.java")
$!callback.setSavePath($tool.append($tableInfo.savePath, "/component"))

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}component;

import com.fasterxml.jackson.annotation.JsonIgnore;
import lombok.Data;

/**
 * @author qiandawei
 */
@Data
public class ServiceResult<T> {
    private int status;
    private String msg;
    private T data;

    public ServiceResult(){}

    private ServiceResult(int status, String msg) {
        this(status, msg, null);
    }

    private ServiceResult(int status, String msg, T data) {
        this.status = status;
        this.msg = msg;
        this.data = data;
    }

    public static <I> ServiceResult<I> createSuccessResult() {
        return create(0, "成功", null);
    }

    public static <I> ServiceResult<I> createSuccessResult(I data) {
        return create(0, "成功", data);
    }

    public static <I> ServiceResult<I> createFailResult() {
        return create(-1, "失败");
    }

    public static <I> ServiceResult<I> createFailResult(String msg) {
        return create(-1, msg);
    }

    public static <I> ServiceResult<I> create(int status, String msg) {
        return new ServiceResult<>(status, msg);
    }

    public static <I> ServiceResult<I> create(int status, String msg, I data) {
        return new ServiceResult<>(status, msg, data);
    }

    @JsonIgnore
    public boolean isSuccess() {
        return status == 0;
    }
}
