# RocketMQ 如何保证消息不丢失

* 生产者使用同步发送增加重试  
  异步可以重试，同步发送可以自动重试（失败指的是发送到broke失败）  
  同步发送+自动重试机制+多个Master节点
* RocketMQ本身  
  多副本，做集群
* 消费者，手动ack，  
  在try中返回 ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
  在catch中返回ConsumeResult.FAILURE


