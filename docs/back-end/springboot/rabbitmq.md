---
title: Spring Boot 整合 RabbitMQ
tags:
  - 后端
  - Spring Boot
  - 消息队列
  - RabbitMQ
---

## 介绍 / Introduction

RabbitMQ 是基于 AMQP 协议的消息队列，广泛用于异步解耦、流量削峰、日志收集等场景。本文覆盖核心概念、声明式配置、生产者 / 消费者、消息可靠性、死信与延迟队列等速查与实战内容。

!!! note
    生产环境建议使用「手动 ACK + 死信队列 + 持久化」组合保证消息可靠性。
    本文示例基于 Spring Boot 3.x + `spring-boot-starter-amqp`。

## 核心概念 / Concepts

| 组件 | 说明 |
| --- | --- |
| Producer | 生产者，发送消息的一方 |
| Consumer | 消费者，接收并处理消息的一方 |
| Queue | 队列，存储消息，多条消费者按轮询消费 |
| Exchange | 交换机，接收生产者消息并按规则路由到队列 |
| Binding | 绑定，描述交换机与队列的路由关系 |
| Routing Key | 路由键，Binding 与消息投递时携带的匹配字符串 |
| Virtual Host | 虚拟主机，逻辑隔离单元，权限以 vhost 为粒度 |

消息流转：`Producer → Exchange → (Binding + Routing Key) → Queue → Consumer`。

## 四种交换机类型 / Exchange Types

| 类型 | 路由规则 | 典型场景 |
| --- | --- | --- |
| direct | Routing Key 精确匹配 | 点对点任务分发 |
| topic | Routing Key 通配符匹配（`*` 单层、`#` 多层） | 业务分类订阅 |
| fanout | 忽略 Routing Key，广播到所有绑定队列 | 通知、广播 |
| headers | 按消息 header 键值匹配（忽略 Routing Key） | 复杂条件路由 |

```text
# topic 示例
routing key: order.create.success
pattern order.*.success   -> 匹配（* 匹配一段）
pattern order.#           -> 匹配（# 匹配多段）
pattern order.#.fail      -> 不匹配
```

## 依赖与配置 / Dependencies & Config

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<!-- JSON 消息转换（Jackson 已在 web starter 中，单独引入亦可） -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual      # none / auto / manual
        prefetch: 10                  # 每个消费者最大未确认消息数（限流）
        concurrency: 3                # 最小消费者并发数
        max-concurrency: 10           # 最大消费者并发数
        retry:
          enabled: true               # 消费失败自动重试
          max-attempts: 3
    publisher-confirm-type: correlated # 生产者确认（开启 confirm 回调）
    publisher-returns: true            # 路由失败回退给生产者
    template:
      mandatory: true                  # 无法路由时触发 ReturnCallback
```

!!! tip
    `publisher-confirm-type` 可选值：`none`（关闭）、`simple`（同步等待）、`correlated`（异步回调，推荐）。

## 声明队列与交换机 / Declare Queue & Exchange

```java
@Configuration
public class RabbitMQConfig {

    public static final String EXCHANGE = "exchange.demo";
    public static final String QUEUE = "queue.demo";
    public static final String ROUTING_KEY = "routing.key";
    public static final String DLX_EXCHANGE = "exchange.dlx";
    public static final String DLX_QUEUE = "queue.dlx";

    /** 业务交换机（topic） */
    @Bean
    public TopicExchange demoExchange() {
        return ExchangeBuilder.topicExchange(EXCHANGE).durable(true).build();
    }

    /** 业务队列：持久化 + 绑定死信交换机 */
    @Bean
    public Queue demoQueue() {
        return QueueBuilder.durable(QUEUE)
                .deadLetterExchange(DLX_EXCHANGE)   // 进入死信的交换机
                .deadLetterRoutingKey("dlx.key")    // 死信路由键
                .ttl(60_000)                        // 队列级 TTL（毫秒）
                .maxLength(10_000)                  // 队列最大长度
                .build();
    }

    /** 绑定：交换机 + 队列 + 路由键 */
    @Bean
    public Binding demoBinding(Queue demoQueue, TopicExchange demoExchange) {
        return BindingBuilder.bind(demoQueue).to(demoExchange).with(ROUTING_KEY);
    }

    /** 死信交换机与队列 */
    @Bean
    public DirectExchange dlxExchange() {
        return ExchangeBuilder.directExchange(DLX_EXCHANGE).durable(true).build();
    }

    @Bean
    public Queue dlxQueue() {
        return QueueBuilder.durable(DLX_QUEUE).build();
    }

    @Bean
    public Binding dlxBinding(Queue dlxQueue, DirectExchange dlxExchange) {
        return BindingBuilder.bind(dlxQueue).to(dlxExchange).with("dlx.key");
    }
}
```

!!! note
    队列队列名、交换机名建议统一定义为常量，避免硬编码字符串漂移。

## 生产者 / Producer

```java
@Service
public class Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /** 发送字符串消息 */
    public void send(String msg) {
        rabbitTemplate.convertAndSend("exchange.demo", "routing.key", msg);
    }

    /** 发送对象（依赖 JSON 消息转换器自动序列化） */
    public void sendOrder(OrderDTO order) {
        rabbitTemplate.convertAndSend("exchange.demo", "order.create", order);
    }
}
```

### 消息转换器（JSON）

```java
@Configuration
public class RabbitMQMessageConfig {

    @Bean
    public MessageConverter jacksonMessageConverter() {
        // 使用 Jackson 将对象序列化为 JSON，避免默认 JDK 序列化（可读性差、跨语言不友好）
        return new Jackson2JsonMessageConverter();
    }
}
```

```java
// 自定义 RabbitTemplate 时一并注入转换器
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory factory,
                                     MessageConverter converter) {
    RabbitTemplate template = new RabbitTemplate(factory);
    template.setMessageConverter(converter);
    template.setMandatory(true);   // 路由失败触发 ReturnCallback
    return template;
}
```

## 消费者 / Consumer

```java
@Component
public class Consumer {

    /** 手动 ACK 消费 */
    @RabbitListener(queues = "queue.demo")
    public void handle(OrderDTO order, Channel channel,
                       @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag) throws IOException {
        try {
            // 业务处理
            System.out.println("Received: " + order);
            // 手动确认：deliveryTag, multiple
            channel.basicAck(deliveryTag, false);
        } catch (Exception e) {
            // requeue=false 丢弃或进入死信队列；true 重新入队
            channel.basicNack(deliveryTag, false, false);
        }
    }

    /** 监听死信队列 */
    @RabbitListener(queues = "queue.dlx")
    public void handleDlx(Message message) {
        System.out.println("DLX Received: " + new String(message.getBody()));
    }
}
```

### ACK 模式说明

| acknowledge-mode | 行为 | 适用场景 |
| --- | --- | --- |
| none | 投递即认为成功，不做确认 | 允许丢消息的日志场景 |
| auto | 方法正常返回自动 ACK，抛异常 NACK 重入队 | 简单业务 |
| manual | 由代码显式 `basicAck` / `basicNack` | 需要可靠投递的生产环境 |

!!! warning
    `manual` 模式下若忘记 `basicAck`，消息会一直处于 unacked 状态并占用 prefetch 配额，最终导致消费者卡死。

## 消息可靠性 / Reliability

### 生产者确认

```java
@Component
public class ProducerConfirmCallback implements RabbitTemplate.ConfirmCallback,
                                                RabbitTemplate.ReturnsCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }

    /** 交换机是否收到消息 */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (!ack) {
            System.err.println("消息未到达交换机: " + cause);
        }
    }

    /** 交换机路由到队列失败（mandatory=true 时触发） */
    @Override
    public void returnedMessage(ReturnedMessage message) {
        System.err.println("路由失败: " + message.getReplyText());
    }
}
```

### 可靠性三要素

| 环节 | 措施 |
| --- | --- |
| 生产者 → 交换机 | publisher-confirm（确认到达交换机） |
| 交换机 → 队列 | publisher-returns + mandatory（路由失败回退） |
| 队列 → 磁盘 | 队列 durable + 消息 persistent（持久化） |
| 队列 → 消费者 | 手动 ACK，处理成功后再确认 |

```java
// 消息级持久化：默认 convertAndSend 已是持久化消息
// 若使用 MessageBuilder 手动构建，需设置 deliveryMode
Message msg = MessageBuilder.withBody("payload".getBytes(StandardCharsets.UTF_8))
        .setDeliveryMode(MessageDeliveryMode.PERSISTENT)
        .build();
rabbitTemplate.send("exchange.demo", "routing.key", msg);
```

## 死信队列与延迟队列 / DLX & Delay

### 消息变成死信的三种情况

- 消费者 `basicNack` / `basicReject` 且 `requeue=false`
- 消息 TTL 过期
- 队列达到最大长度被丢弃（队列设置 `maxLength` + `overflow=reject-publish`）

!!! tip
    TTL 队列 + DLX 可实现「延迟队列」：消息在业务队列过期后进入死信队列被消费，即延迟触发。

### 延迟队列方案对比

| 方案 | 实现 | 优点 | 缺点 |
| --- | --- | --- | --- |
| TTL + DLX | 队列/消息设置 TTL，过期入 DLX | 无需插件 | 队列级 TTL 会「队头阻塞」，不适合精确延迟 |
| 延迟插件 | `rabbitmq_delayed_message_exchange` | 单条消息任意延迟、精确 | 需安装插件 |

```bash
# 安装延迟插件（Docker）
docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

```java
/** 延迟交换机（需插件） */
@Bean
public CustomExchange delayExchange() {
    Map<String, Object> args = new HashMap<>();
    args.put("x-delayed-type", "direct");
    return new CustomExchange("exchange.delay", "x-delayed-message", true, false, args);
}

/** 发送延迟消息：header 中指定延迟毫秒数 */
public void sendDelay(String msg, int delayMs) {
    rabbitTemplate.convertAndSend("exchange.delay", "delay.key", msg, m -> {
        m.getMessageProperties().setDelayLong((long) delayMs);
        return m;
    });
}
```

## 消息幂等与重复消费 / Idempotency

RabbitMQ 只能保证「至少一次」投递，消费端可能重复收到同一消息，需做幂等。

| 方案 | 说明 |
| --- | --- |
| 唯一 ID + Redis | 消费前 `SETNX msgId`，已存在则跳过 |
| 数据库唯一索引 | 用业务唯一键建唯一索引，重复插入抛异常忽略 |
| 状态机 / 乐观锁 | 更新前校验状态与版本号 |

```java
@Component
public class IdempotentConsumer {

    @Autowired
    private StringRedisTemplate redisTemplate;

    @RabbitListener(queues = "queue.demo")
    public void handle(Message message, Channel channel,
                       @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws IOException {
        String msgId = message.getMessageProperties().getMessageId();
        // setIfAbsent 成功返回 true，表示首次处理
        Boolean first = redisTemplate.opsForValue()
                .setIfAbsent("mq:idempotent:" + msgId, "1", Duration.ofHours(24));
        if (Boolean.TRUE.equals(first)) {
            doBusiness(message);
        }
        channel.basicAck(tag, false);
    }

    private void doBusiness(Message message) { /* 幂等处理业务 */ }
}
```

## 常见问题 / FAQ

### 消息堆积

- 原因：消费者处理慢、消费线程不足、下游阻塞。
- 处理：提高 `concurrency` / `max-concurrency`，增加消费者实例，优化业务耗时，必要时临时扩容队列消费。
- 监控：管理界面 Queue 的 `Ready / Unacked / Total` 指标。

### 消费者限流（prefetch）

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 10   # 每个消费者一次最多取 10 条未确认消息
```

!!! note
    `prefetch=1` 可实现公平分发（能者多劳），但吞吐较低；批量处理场景可适当调大。

### 其他速查

| 问题 | 排查方向 |
| --- | --- |
| 消息发不出去 | Exchange 名 / Routing Key 是否匹配，confirm 回调 |
| 消息进不了队列 | 绑定是否声明、是否开启了 mandatory + returns |
| 消费者 3 次重试后丢弃 | 检查 retry 配置与死信队列 |
| 消费者卡死 | 是否忘记手动 ACK，prefetch 是否过小 |

## Docker 启动与管理界面 / Docker

```bash
# 启动带管理界面的 RabbitMQ
docker run -d --name rabbitmq \
  -p 5672:5672 -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin123 \
  rabbitmq:3-management

# 管理界面：http://localhost:15672  账号 admin / admin123
# 查看插件 / 启用延迟插件
docker exec -it rabbitmq rabbitmq-plugins list
docker exec -it rabbitmq rabbitmq-plugins enable rabbitmq_delayed_message_exchange

# 常用运维命令
docker exec -it rabbitmq rabbitmqctl list_queues name messages consumers
docker exec -it rabbitmq rabbitmqctl list_exchanges
docker exec -it rabbitmq rabbitmqctl list_bindings
```

!!! tip
    `guest` 账号默认仅允许本机连接。容器或远程连接请通过 `RABBITMQ_DEFAULT_USER/PASS` 创建新用户。
