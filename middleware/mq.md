# mq.md

用的rabbitmq
ErmqMetricUtil.java:
```
import java.util.UUID;
/** 省略若干引用 */
import org.springframework.amqp.AmqpException;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.core.MessagePropertiesBuilder;
import org.springframework.util.StringUtils;

/**
 * 将数据结构格式化并发送到 MQ 中
 */
@Slf4j
public final class ErmqMetricUtil {

    @Resource(name = "districtTemplate")
    private AmqpTemplate syncIdTemplate;

    public static final void convertAndSendWithMetric(final String routingKey,
                                                      final Object object) {
        convertAndSendWithMetric(syncIdTemplate, routingKey, object);
    }

    public static final void convertAndSendWithMetric(final AmqpTemplate amqpTemplate, final String routingKey,
                                                      final Object object) {
        if (null == amqpTemplate) {
            log.error("AmqpTemplate is null");
            return;
        }
        if (StringUtils.isEmpty(routingKey) || object == null) {
            log.error("routingKey or object is null");
            return;
        }
        Transaction transaction = Trace.newTransaction("AMQP_SEND", routingKey);
        try {
            MessageProperties property = MessagePropertiesBuilder.newInstance()
                    .setMessageId(UUID.randomUUID().toString())
                    .build();
            FastJsonMessageConverter messageConverter = new FastJsonMessageConverter();
//FastJsonMessageConverter extends AbstractMessageConverter
            Message message = messageConverter.toMessage(object, property);
            amqpTemplate.send(routingKey, message);
            transaction.setStatus("success");
        } catch (AmqpException e) {
            log.error("amqpTemplate.convertAndSend[{},{}] fail", routingKey, object);
            transaction.setStatus(e);
            throw e;
        } finally {
            transaction.complete();
        }
    }

}
```
rabbitmq.xml:
```
<bean name="districtTemplate" class="org.springframework.amqp.rabbit.core.RabbitTemplate" id="districtTemplate">
        <property name="connectionFactory" ref="eCachingConnectionFactory" />
        <property name="brokerKey" value="maxq.lpd_ai_eye"/>
        <property name="messageConverter" ref="jsonMessageConverter"/>
        <property name="queue" value="eye.district.queue" />
        <property name="exchange" value="district_topic"/>
</bean>
```
这块的配置不全，需要从网上查资料再完善下