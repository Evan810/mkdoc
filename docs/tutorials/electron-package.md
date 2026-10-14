---
title: Electron 打包教程
tags:
  - 教程
  - Electron
  - 打包
---

## 介绍 / Introduction

使用 electron-builder 将 Electron 应用打包为 Windows / macOS / Linux 可执行文件或安装包。

!!! note
    Windows 下推荐使用 NSIS 安装包；跨平台打包需在对应系统执行或使用 CI。

## 安装与配置

```bash
npm install electron electron-builder --save-dev
```

### package.json 关键配置

```json
{
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "build": "electron-builder --win --x64"
  },
  "build": {
    "appId": "com.example.myapp",
    "productName": "MyApp",
    "win": {
      "target": ["nsis", "portable"],
      "icon": "build/icon.ico"
    },
    "nsis": {
      "oneClick": false,
      "allowToChangeInstallationDirectory": true
    }
  }
}
```

### 执行打包

```bash
npm run build
```
