---
title: Node.js 分类
tags:
  - 后端
  - Node.js
---

## Node.js

Node.js 是基于 V8 的 JavaScript 运行时，广泛用于 I/O 密集型服务端应用。

!!! note
    本分类同时覆盖轻量框架 Express 与企业级框架 NestJS。

## 核心特点

- 事件驱动、非阻塞 I/O，适合高并发 I/O 场景
- 单线程 + 事件循环模型，CPU 密集型任务需借助 Worker / 子进程
- 与前端同栈，全栈开发成本低
- npm 生态庞大，包管理便捷

## 框架对比

| 对比项 | Express | NestJS |
| --- | --- | --- |
| 定位 | 轻量、灵活 | 企业级、约定优先 |
| 语言 | JavaScript | TypeScript |
| 架构 | 自由组织 | 模块化（Module/Controller/Service） |
| 依赖注入 | 无 | 内置 |
| 学习曲线 | 低 | 中 |
| 适用 | 中小项目、快速原型 | 中大型项目、团队协作 |

## 子页面

- :fontawesome-solid-bolt: [Express](express.md) —— 轻量灵活，路由与中间件机制简洁
- :fontawesome-solid-layer-group: [NestJS](nestjs.md) —— TypeScript 企业级，装饰器与依赖注入

## 快速开始

```bash
# 创建 Express 项目
npm install express

# 创建 NestJS 项目
npm i -g @nestjs/cli
nest new my-nest-app
```

```javascript
// 最小 Express 服务
const express = require('express');
const app = express();

app.get('/', (req, res) => res.json({ message: 'Hello Node.js!' }));
app.listen(3000, () => console.log('Server running on :3000'));
```

!!! tip
    追求快速上手选 Express；追求可维护性与工程化选 NestJS。
