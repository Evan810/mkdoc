---
title: Spring Boot 整合 RabbitMQ
tags:
  - 后端
  - Spring Boot
  - 消息队列
---

## 介绍 / Introduction

RabbitMQ 是基于 AMQP 协议的消息队列，广泛用于异步解耦、流量削峰等场景。

!!! note
    生产环境建议使用手动 ACK + 死信队列保证消息可靠性。

## 生产者与消费者

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 代码示例

```java
// 生产者
@Service
public class Producer {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void send(String msg) {
        rabbitTemplate.convertAndSend("exchange.demo", "routing.key", msg);
    }
}

// 消费者
@Component
public class Consumer {
    @RabbitListener(queues = "queue.demo")
    public void handle(String msg) {
        System.out.println("Received: " + msg);
    }
}
```

```bash
# Docker 启动 RabbitMQ（带管理界面）
docker run -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```
