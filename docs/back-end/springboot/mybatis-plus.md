---
title: MyBatis-Plus
tags:
  - 后端
  - Spring Boot
  - ORM
---

## 介绍 / Introduction

MyBatis-Plus 是 MyBatis 的增强工具，提供 CRUD 封装、条件构造器、代码生成等开箱即用功能。

!!! note
    合理使用 MP 的 Wrapper 可以减少大量 XML 映射文件。

## 快速上手

### 依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.5</version>
</dependency>
```

### Mapper 与 Service

```java
// 实体
@TableName("user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    private Integer age;
}

// Mapper
@Mapper
public interface UserMapper extends BaseMapper<User> {}

// 使用 LambdaQueryWrapper 查询
List<User> users = userMapper.selectList(
    Wrappers.<User>lambdaQuery()
        .eq(User::getAge, 18)
        .like(User::getName, "张")
);
```
