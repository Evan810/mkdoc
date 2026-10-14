---
title: Spring Boot 项目模板
tags:
  - 后端
  - Spring Boot
  - 脚手架
---

## 介绍 / Introduction

整理可直接 fork 使用的 Spring Boot 项目模板，包含通用分层、统一返回、异常处理等脚手架能力。

!!! note
    个人模板仓库地址见文末链接，欢迎 Star / Fork。

## 推荐模板结构

```bash
springboot-template/
├── common/          # 通用工具、枚举、异常
├── controller/      # 对外接口层
├── service/         # 业务逻辑层
├── mapper/          # 数据访问层 (MyBatis-Plus)
├── entity/          # 数据库实体
├── dto/             # 入参 DTO
├── vo/              # 出参 VO
└── config/          # 配置类
```

## 统一返回示例

```java
@Data
public class R<T> {
    private int code;
    private String msg;
    private T data;

    public static <T> R<T> ok(T data) {
        R<T> r = new R<>();
        r.setCode(200);
        r.setMsg("success");
        r.setData(data);
        return r;
    }
}
```
