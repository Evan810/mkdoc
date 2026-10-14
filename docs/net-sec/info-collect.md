---
title: 信息收集
tags:
  - 网络安全
  - 渗透测试
  - 信息收集
---

## 介绍 / Introduction

信息收集是渗透测试的第一步，涵盖子域名枚举、端口扫描、指纹识别、社工情报等。

!!! warning
    仅对已授权目标执行信息收集，禁止对未授权系统进行扫描。

## 子域名枚举

```bash
# subfinder
subfinder -d example.com -silent

# amass
amass enum -passive -d example.com
```

## 端口扫描

```bash
# 全端口扫描 + 版本探测
nmap -sV -p- -T4 192.168.1.1

# 常见漏洞脚本
nmap --script=vuln -p 80,443,8080 target
```
