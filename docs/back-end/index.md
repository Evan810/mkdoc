---
title: 后端开发分类
tags:
  - 后端
  - Spring Boot
  - Node.js
---

## 后端开发 / Backend Development

后端开发分类整理了服务端开发的技术栈，从框架选型到中间件集成，覆盖常见业务场景。

!!! note
    本站偏向 Java (Spring Boot) 与 Node.js 两大技术栈。

## 技术栈概览

| 领域 | Java 生态 | Node.js 生态 |
| --- | --- | --- |
| Web 框架 | Spring Boot | Express / NestJS |
| ORM | MyBatis-Plus / JPA | TypeORM / Prisma / Sequelize |
| 缓存 | Redis | Redis (ioredis) |
| 消息队列 | RabbitMQ / Kafka | RabbitMQ / BullMQ |
| 全文检索 | Elasticsearch | Elasticsearch |
| 注册/配置中心 | Nacos | Consul |
| 网关 | Spring Cloud Gateway | Nginx / Kong |

## 子分类

<div class="grid cards" markdown>

- :fontawesome-brands-java: **Spring Boot**
  { .card-heading }
  Java 企业级开发，覆盖技术整合、消息队列、全文检索、ORM
  [进入 Spring Boot &rarr;](springboot/index.md)

- :fontawesome-brands-node-js: **Node.js**
  { .card-heading }
  JavaScript 后端运行时，覆盖 Express 轻量框架与 NestJS 企业级框架
  [进入 Node.js &rarr;](nodejs/index.md)

</div>

## 学习路线

```text
基础语法 / Java 基础
        ↓
Web 框架（Spring Boot / Express）
        ↓
数据持久化（MyBatis-Plus / TypeORM）
        ↓
缓存与中间件（Redis / RabbitMQ / ES）
        ↓
微服务与分布式（Nacos / Sentinel / Seata）
        ↓
部署运维（Docker / Nginx / CI-CD）
```

```bash
# 快速启动一个 Spring Boot 项目
curl https://start.spring.io/starter.zip -d dependencies=web -o demo.zip
```

```bash
# 快速创建 Express 项目
npm init -y && npm install express
```

!!! tip
    建议先掌握一门框架的完整开发流程，再横向对比另一技术栈，避免同时铺开导致基础不牢。
