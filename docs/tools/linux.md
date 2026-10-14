---
title: Linux 常用命令
tags:
  - 工具
  - Linux
  - Shell
---

## 介绍 / Introduction

Linux / macOS 下高频 Shell 命令速查，覆盖文件操作、进程管理、网络与权限。

!!! note
    Windows 用户推荐在 WSL2 或 Git Bash 下使用这些命令。

## 文件操作

```bash
ls -lah                    # 详细列出含隐藏文件
cp -r src/ dst/            # 递归复制
mv old.txt new.txt         # 移动 / 重命名
rm -rf temp/               # 强制递归删除（谨慎！）
find . -name "*.log"        # 按名称查找文件
grep -r "TODO" .           # 递归搜索字符串
```

## 进程与网络

```bash
ps aux | grep python       # 过滤进程
kill -9 <pid>              # 强制杀死进程
netstat -tlnp              # 查看监听端口
curl -I http://example.com # 只看响应头
```

## 权限

```bash
chmod +x script.sh         # 加执行权限
chown user:group file      # 修改所有者
sudo -i                    # 切换 root
```
