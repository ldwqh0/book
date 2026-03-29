# RocketMQ 如何保证消息的顺序

发送端有序，使用MessageQueueSelector，使用 MessageQueueSelector，根据业务键（如 orderId）进行哈希取模，强制将同一业务的数据路由到固定队列。

```java
new MessageQueueSelector() {
    @Override
    public MessageQueue select (List < MessageQueue > mqs, Message msg, Object arg){
// 1. 获取业务唯一标识 (如订单ID)
        Long orderId = (Long) arg;

        // 2. 计算哈希值，映射到具体的队列索引
        // 确保同一个 orderId 永远算出同一个 index
        int index = (int) (orderId % mqs.size());

        // 3. 返回选定的队列
        return mqs.get(index);
    }

}
```

消费端有序，注册MessageListenerOrderly

```java

public void dd() {
    consumer.registerMessageListener((MessageListenerOrderly) (msgs, context) -> {
        // 1. 自动提交偏移量（默认配置下）
        context.setAutoCommit(true);
        // 1. 这里会自动加锁！
        // RocketMQ 会保证：对于同一个 MessageQueue，同一时刻只有一个线程能执行到这里

        for (MessageExt msg : msgs) {
            try {
                // 2. 执行你的业务逻辑(必须是幂等的)
                String body = new String(msg.getBody(), StandardCharsets.UTF_8);
                System.out.printf("线程 %s 接收到顺序消息: %s %n", Thread.currentThread().getName(), body);
            } catch (Exception e) {
                // 3. 业务逻辑异常处理
                e.printStackTrace();
                // 如果返回 SUSPEND_CURRENT_QUEUE_A_MOMENT，该队列会暂停消费并在稍后重试
                return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
            }
        }
        // 4. 返回消费成功状态
        return ConsumeOrderlyStatus.SUCCESS;
    });
}
```

```java

@Component
@RocketMQMessageListener(
        consumerGroup = "order-group",
        topic = "order-topic",
        consumeMode = ConsumeMode.ORDERLY
)
public class OrderConsumer implements RocketMQListener<MessageExt> {

    @Override
    public ConsumeOrderlyStatus consumeMessage(List<MessageExt> messages, ConsumeOrderlyContext context) {
        for (MessageExt message : messages) {
            try {
                // 处理消息逻辑
                System.out.println("顺序消息: " + new String(message.getBody()));

                // 消费成功
                return ConsumeOrderlyStatus.SUCCESS;
            } catch (Exception e) {
                // 消费失败，稍后重试
                return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
            }
        }
        return ConsumeOrderlyStatus.SUCCESS;
    }

}

```
