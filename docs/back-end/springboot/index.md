---
title: Spring Boot 分类
---

## Spring Boot

Spring Boot 让 Spring 应用变得开箱即用。本分类聚焦 Spring Boot 在实际项目中的集成与最佳实践。

!!! note
    示例代码基于 Spring Boot 3.x + Java 17 编写。

### 主要主题

- 技术整合（Redis、Nacos、Sentinel 等）
- 消息队列 RabbitMQ
- 全文检索 Elasticsearch
- ORM 框架 MyBatis-Plus
- 项目模板与脚手架

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```
