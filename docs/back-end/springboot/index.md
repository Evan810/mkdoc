---
title: Spring Boot 分类
tags:
  - 后端
  - Spring Boot
---

## Spring Boot

Spring Boot 让 Spring 应用变得开箱即用。本分类聚焦 Spring Boot 在实际项目中的集成与最佳实践。

!!! note
    示例代码基于 Spring Boot 3.x + Java 17 编写。

## 核心特性

| 特性 | 说明 |
| --- | --- |
| 自动配置 | 根据依赖自动装配 Bean，减少 XML |
| Starter 依赖 | 一站式引入某类功能的依赖集合 |
| 内嵌容器 | 内置 Tomcat，可直接 `java -jar` 运行 |
| Actuator | 健康检查、指标监控、运行状态 |
| 外部化配置 | `application.yml` / 环境变量 / 配置中心 |

## 主要主题

<div class="grid cards" markdown>

- :fontawesome-solid-puzzle-piece: **技术整合**
  { .card-heading }
  Redis、Nacos、Sentinel、Seata 等中间件整合
  [查看 &rarr;](integrate-tech.md)

- :fontawesome-solid-envelope: **RabbitMQ**
  { .card-heading }
  消息队列：异步解耦、死信队列、可靠性投递
  [查看 &rarr;](rabbitmq.md)

- :fontawesome-solid-magnifying-glass: **Elasticsearch**
  { .card-heading }
  分布式全文检索：DSL 查询、聚合、Spring 整合
  [查看 &rarr;](elasticsearch.md)

- :fontawesome-solid-database: **MyBatis-Plus**
  { .card-heading }
  ORM 增强：CRUD 封装、条件构造器、代码生成
  [查看 &rarr;](mybatis-plus.md)

- :fontawesome-solid-cubes: **项目模板**
  { .card-heading }
  可直接 fork 的脚手架：分层、统一返回、异常处理
  [查看 &rarr;](springboot-template.md)

</div>

## 快速开始

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

```yaml
# application.yml 常用配置
server:
  port: 8080
spring:
  application:
    name: demo
  profiles:
    active: dev
```

```bash
# 生成项目 / 启动 / 打包
curl https://start.spring.io/starter.zip -d dependencies=web -o demo.zip
mvn spring-boot:run
mvn clean package -DskipTests
```

!!! tip
    掌握顺序建议：基础 Web 开发 → 数据持久化 → 缓存与消息队列 → 配置中心与微服务。
