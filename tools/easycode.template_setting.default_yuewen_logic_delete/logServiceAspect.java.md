```
##引入宏定义
$!define
$!variable

##设置回调
$!callback.setFileName("LogServiceAspect.java")
$!callback.setSavePath($tool.append($tableInfo.savePath, "/component"))

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}component;

import com.qq.cloud.taf.client.exc.NoConnectionException;
import com.qq.cloud.taf.client.rpc.exc.NoInvokerException;
import com.yuewen.web.utils.JsonUtil;
import org.apache.commons.lang3.exception.ExceptionUtils;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import org.springframework.boot.logging.LogLevel;
import org.springframework.stereotype.Component;

import java.util.concurrent.TimeoutException;

/**
 * @author hanbian
 * @description:用于业务层，统一收集日志
 * @date 2020/09/15
 */
@Aspect
@Component
public class LogServiceAspect {


    private static final Logger logger = LoggerFactory.getLogger(LogServiceAspect.class);

    @Pointcut("execution(public com.yuewen.web.component.ServiceResult *(..))")
    private void serviceResult() {
    }

//    @Pointcut("execution(public * com.yuewen.amis.api.service.impl..*.*(..)) ")
//    private void serviceResult() {
//
//    }

    @Around(value = "serviceResult()")
    public Object businessLogger(ProceedingJoinPoint point) {
        ServiceResult resultValue;
        long start = System.currentTimeMillis();
        // 避免线程池线程重用导致的数据混乱
        MDC.clear();
        String methodName = point.getSignature().getName();
        try {
            resultValue = (ServiceResult) point.proceed();
            long end = System.currentTimeMillis();
            String costTime = String.valueOf(end - start);
            MDC.put("methodName", methodName);
            MDC.put("className", point.getTarget().getClass().getName());
            MDC.put("costTime", costTime);


            if (resultValue != null) {
                if (!resultValue.isSuccess()) {
                    logInvokerInfo(point, resultValue, LogLevel.WARN, point.getThis().getClass());
                }
                if (logger.isDebugEnabled())
                    logInvokerInfo(point, resultValue, LogLevel.DEBUG, point.getThis().getClass());
            } else {
                logInvokerInfo(point, resultValue, LogLevel.WARN, point.getThis().getClass());
                //如果返回值等于null，则抛出错误
                resultValue = ServiceResult.createFailResult("Entry返回null");
            }
        } catch (TimeoutException throwable) {
            logger.error("Service method:{} Paramters:{} Timeout Error:{}",
                    point.getSignature(),
                    JsonUtil.getInstance().object2String(point.getArgs()),
                    throwable.getMessage());
            resultValue = ServiceResult.createFailResult("系统超时异常");
        } catch (NoConnectionException throwable) {
            logger.error("Service method:{} Paramters:{} NoConnection Error:{}",
                    point.getSignature(),
                    JsonUtil.getInstance().object2String(point.getArgs()),
                    throwable.getMessage());
            resultValue = ServiceResult.createFailResult("NoConnection异常");
        } catch (NoInvokerException throwable) {
            logger.error("Service method:{} Paramters:{}  Error:{}",
                    point.getSignature(),
                    JsonUtil.getInstance().object2String(point.getArgs()),
                    throwable.getMessage());
            resultValue = ServiceResult.createFailResult("未找到Invoker");
        } catch (Throwable throwable) {
            logger.error("Service method:{} Paramters:{}  Error:{}",
                    point.getSignature(),
                    JsonUtil.getInstance().object2String(point.getArgs()),
                    ExceptionUtils.getStackTrace(throwable));
            resultValue = ServiceResult.createFailResult("系统异常");
        }
        //清除，避免串了
        MDC.clear();

        return resultValue;

    }

    private void logInvokerInfo(JoinPoint joinpoint, Object returnValue, LogLevel level,
                                Class logClass) {
        StringBuilder builder = new StringBuilder();
        builder.append("\n-----------------\n");
        builder.append(">>>>>>method:").append(joinpoint.getSignature());
        builder.append(">>>>>>Parameters:").append(JsonUtil.getInstance().object2String(joinpoint.getArgs()));
        builder.append(">>>>>>Return:").append(JsonUtil.getInstance().object2String(returnValue)).append("\n");
        builder.append("----------------\n");
        switch (level) {
            case DEBUG:
                //LoggerFactory.getLogger(logClass).debug(builder.toString());
                break;
            case INFO:
                //LoggerFactory.getLogger(logClass).info(builder.toString());
                break;
            case WARN:
                //LoggerFactory.getLogger(logClass).warn(builder.toString());
                break;
            case ERROR:
                //LoggerFactory.getLogger(logClass).error(builder.toString());
                break;
        }

    }

}

```