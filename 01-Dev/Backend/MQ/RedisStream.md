# 一.RedisStream基础
RedisStream是一个可持久化的追加日志队列：
- XADD -> 写入消息
- XREAD -> 读取消息
- XGROUP -> 创建消费者组
- XREADGROUP -> 消费者组读取
- XACK -> 消费确认
每条消息都有唯一的消息ID。

## 1.相比List的优势是什么？
List的问题是：
- 不支持**消费者组**。
- 消息被弹出后，如果**消费者宕机会导致消息丢失**。
- 不方便**记录消费进度**。
- 不方便处理**未确认的消息**。
Stream的优点：
- 支持消费者组。
- 支持消息ID。
- 支持`ACK`确认。
- 支持`PendingList`，可以处理**已投递但未确认**的消息。
- 可以通过`RDB/AOF`进行持久化。
所以RedisStream更像轻量级的MQ。

## 2.消费者组
消费者组就是一组消费者共同消费一个Stream。
- 同一个消费者组：一条消息只会被一个消费者消费。
- 不同消费者组之间：多个消费者组可以消费同一批消息。
比如
- `stream: order_stream`
- `group1`: 订单服务消费者组
- `group2`: 数据统计消费者组
`group1`可以消费一次，`group2`也可以消费一次。

## 3.PendingList是什么？
已经投递给消费者但是没有被`ack`的消息。
所以RedisStream至少可以做到：消息不轻易丢失，但可能重复消费。

## 4.RedisStream如何保证消息可靠性？
Redis Stream 通过**消费者组、ACK 确认机制和 Pending List** 来提高消息可靠性。消费者读取消息后，如果处理成功，需要执行 XACK 确认；如果没有确认，消息会留在 **Pending Lis**t 中，后续可以被重新投递给其他消费者处理。同时 Stream 数据可以依赖 Redis 的 **RDB 或 AOF 进行持久化**。不过它一般**只能保证至少一次投递，所以业务侧要做好幂等处理**。

## 5.RedisStream和RabbitMQ的区别
Redis Stream 是 Redis 内置的**轻量级消息队列**，适合已有 Redis 的项目中做**简单异步消费**。它支持消费者组、ACK 和 Pending List，但**整体可靠性、路由能力、死信、延迟消息、消息堆积处理能力**不如 RabbitMQ。RabbitMQ 是专业 MQ，支持交换机、路由、生产者确认、消费者 ACK、死信队列、延迟队列和高可用机制，更适合复杂业务解耦。