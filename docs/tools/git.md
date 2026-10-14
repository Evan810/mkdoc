---
title: Git 速查
tags:
  - 工具
  - Git
  - 版本控制
---

## 介绍 / Introduction

Git 常用命令速查，覆盖日常提交、分支管理、远程同步、撤销操作等高频场景。

!!! note
    建议配置 Git alias 简化长命令，例如 `git st` 代替 `git status`。

## 高频命令

### 基础操作

```bash
git init                          # 初始化仓库
git clone <url>                   # 克隆远程仓库
git add .                         # 暂存全部更改
git commit -m "feat: add xxx"     # 提交
git push origin main              # 推送
git pull --rebase                 # 拉取并变基
```

### 分支管理

```bash
git branch                        # 查看分支
git checkout -b feature/new       # 创建并切换
git merge feature/new             # 合并分支
git branch -d feature/new         # 删除已合并分支
```

### 撤销与重置

```bash
git checkout -- file.txt          # 丢弃工作区修改
git reset HEAD file.txt           # 取消暂存
git reset --hard HEAD~1           # 回退一个提交（谨慎！）
git reflog                        # 查看 HEAD 历史，用于找回丢失提交
```
