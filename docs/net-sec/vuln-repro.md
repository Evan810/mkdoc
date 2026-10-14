---
title: 常见漏洞复现清单
tags:
  - 网络安全
  - 漏洞复现
  - CVE
---

## 介绍 / Introduction

整理近期高危 CVE 的复现环境与 PoC 链接，方便快速搭建本地靶场验证。

!!! note
    建议使用 Docker / vulhub 一键拉起漏洞环境。

## 推荐靶场

```bash
# vulhub 启动示例
git clone https://github.com/vulhub/vulhub.git
cd vulhub/spring/CVE-2022-22965
docker-compose up -d
```

## 近期关注

| CVE | 组件 | 类型 |
|-----|------|------|
| CVE-2024-XXXX | XXX | RCE |
| CVE-2024-YYYY | XXX | SQLi |
