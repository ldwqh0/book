# Kafka 如何保证消息不丢

## 保证不丢

acks = all (或 -1)：  
要求 Leader 和所有 ISR（同步副本列表）中的 Follower 都写入成功后，才返回成功响应。这是防止 Leader 宕机丢数据的关键。

retries = Integer.MAX_VALUE：  
遇到网络抖动或 Broker 暂时不可用时，无限次重试，直到成功。

enable.idempotence = true (幂等性)：  
关键：开启重试后，网络超时可能导致 Broker 收到了但 Producer 没收到 ACK，从而重复发送。开启幂等性可以保证“Exactly
Once”语义（单分区内），即重试不会导致消息重复，也不会丢消息。

max.in.flight.requests.per.connection = 5 (配合幂等性)：  
在较新版本（2.5+）开启幂等性后，该值可大于1；但在旧版本或追求严格顺序时，建议设为 1，防止重试时消息乱序（虽然幂等性保不丢，但不一定保序，见下文）

## Kafka 如何开启手动确认

enable.auto.commit 设置为 false。

```java

@KafkaListener(topics = "my-topic2", groupId = "myGroup")
public void receiveMessage2(String message, Acknowledgment ack) {
    log.info("消费消息：" + message);
//ack确认
    ack.acknowledge();
}

```

Kafka事务消息。是指一批消息全部成功，或者一批消息全部失败
