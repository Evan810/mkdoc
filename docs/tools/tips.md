---
title: 开发小贴士
tags:
  - 工具
  - 技巧
  - 效率
---

## 介绍 / Introduction

零散但高频使用的开发小贴士合集，提升日常编码效率与代码质量。

!!! note
    持续更新中，欢迎贡献你觉得好用的小技巧。

## VS Code 配置片段

### settings.json 常用项

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "files.autoSave": "afterDelay",
  "editor.fontSize": 14,
  "workbench.iconTheme": "material-icon-theme"
}
```

## PowerShell 美化（Windows）

```powershell
# 安装 Oh My Posh
winget install JanDeDobbeleer.OhMyPosh
oh-my-posh init pwsh | Invoke-Expression

# 安装 PSReadLine（命令高亮 + 历史搜索）
Install-Module PSReadLine -Force
```

## Git Bash Alias

```bash
# ~/.bashrc
alias gs='git status'
alias gl='git log --oneline --graph --all -20'
alias gp='git pull --rebase'
```
