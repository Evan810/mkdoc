---
title: Vite
tags:
  - 前端
  - Vite
  - 构建工具
---

## 介绍 / Introduction

Vite 是基于原生 ES Modules 的新一代前端构建工具，开发环境启动极快。

!!! note
    Vite 4.x 起需要 Node.js 14.18+ 或 16+。

## 核心特性

### 创建项目

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm run dev
```

### 自定义配置

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    open: true,
  },
});
```
