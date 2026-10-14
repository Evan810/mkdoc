---
title: Spring Boot 技术整合
tags:
  - 后端
  - Spring Boot
  - 微服务
---

## 介绍 / Introduction

记录 Spring Boot 与常见中间件（Redis、Nacos、Sentinel、Seata 等）的整合实战。

!!! note
    所有整合示例基于 Spring Boot 3.x 与 Spring Cloud 2023.x。

## Redis 缓存示例

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 使用

```java
@Service
public class UserService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    public void cacheUser(Long id, String name) {
        redisTemplate.opsForValue().set("user:" + id, name, 1, TimeUnit.HOURS);
    }
}
```

```bash
# Docker 启动 Redis
docker run -p 6379:6379 redis:7
```
