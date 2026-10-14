---
title: Express
tags:
  - 后端
  - Node.js
  - Express
---

## 介绍 / Introduction

Express 是 Node.js 生态中最流行的轻量 Web 框架，路由、中间件机制简洁灵活。

!!! note
    Express 5 已进入 Beta，正式发布后将带来异步路由处理等改进。

## 最小示例

```bash
npm install express
```

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.get('/', (req, res) => {
  res.json({ message: 'Hello Express!' });
});

app.listen(3000, () => console.log('Server running on :3000'));
```

## 路由与中间件

```javascript
// 中间件
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// 参数路由
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  res.json({ id, name: `User-${id}` });
});
```
