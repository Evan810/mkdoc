---
title: Express
tags:
  - 后端
  - Node.js
  - Express
  - 中间件
  - RESTful
---

## 介绍 / Introduction

Express 是 Node.js 生态中最流行的轻量 Web 框架，路由与中间件机制简洁灵活，
生态成熟、上手成本低，适合快速构建 RESTful API 与中小型服务。

!!! note
    Express 4 为当前主流稳定版；Express 5 已进入 Beta，将原生支持异步路由错误的自动捕获。

## 安装与最小示例

```bash
# 初始化项目
npm init -y
# 安装 express
npm install express
```

```javascript
const express = require('express');

const app = express();

// 解析 JSON 请求体
app.use(express.json());

app.get('/', (req, res) => {
  res.json({ message: 'Hello Express!' });
});

app.listen(3000, () => console.log('Server running on :3000'));
```

!!! tip
    使用 ES Module 时，在 `package.json` 中加入 `"type": "module"`，改用 `import express from 'express'`。

## 路由 / Routing

### 路径参数与查询参数

```javascript
// 路径参数 :id
app.get('/users/:id', (req, res) => {
  const { id } = req.params;          // 字符串类型
  res.json({ id: Number(id), name: `User-${id}` });
});

// 多个路径参数
app.get('/users/:userId/books/:bookId', (req, res) => {
  const { userId, bookId } = req.params;
  res.json({ userId, bookId });
});

// 查询参数 ?page=1&size=10
app.get('/search', (req, res) => {
  const { q, page = 1, size = 10 } = req.query;  // 均为字符串
  res.json({ q, page: Number(page), size: Number(size) });
});
```

### 路由方法

```javascript
app.get('/items', handler);      // 查询
app.post('/items', handler);     // 新增
app.put('/items/:id', handler);  // 全量更新
app.patch('/items/:id', handler);// 局部更新
app.delete('/items/:id', handler); // 删除
app.all('/secret', handler);     // 任意方法都匹配
```

### 路由分组：express.Router

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

// 中间件挂载到该 router，仅对 /users 生效
router.use((req, res, next) => {
  console.log('users router:', req.method);
  next();
});

router.get('/', (req, res) => res.json([]));
router.get('/:id', (req, res) => res.json({ id: req.params.id }));
router.post('/', (req, res) => res.status(201).json({ ok: true }));

module.exports = router;
```

```javascript
// app.js
const usersRouter = require('./routes/users');
app.use('/users', usersRouter);   // 统一前缀 /users
```

## 中间件机制与执行顺序 / Middleware

Express 是「洋葱模型」，请求依次经过中间件，`next()` 继续向下，出错则跳到错误处理中间件。

| 类型 | 挂载方式 | 作用范围 |
| --- | --- | --- |
| 应用级 | `app.use(fn)` / `app.use('/path', fn)` | 全局或指定前缀 |
| 路由级 | `router.use(fn)` | 仅该 router |
| 错误处理 | `app.use((err, req, res, next) => {})` | 四参数签名，兜底异常 |
| 内置 | `express.json()` / `express.static()` | 解析与静态资源 |
| 第三方 | `morgan` / `cors` / `cookie-parser` | 功能扩展 |

```javascript
// 应用级中间件：执行顺序按注册先后
app.use((req, res, next) => {
  console.log('1. 全局中间件');
  next();                          // 必须调用，否则请求挂起
});

// 路由级中间件
app.get('/admin', (req, res, next) => {
  console.log('2. /admin 前置校验');
  next();
}, (req, res) => {
  res.json({ ok: true });
});

// 错误处理中间件（必须 4 个参数，放在所有路由之后）
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.status || 500).json({ message: err.message });
});
```

!!! warning
    错误处理中间件必须声明为 `(err, req, res, next)` 四个参数，否则 Express 会当作普通中间件。

## 请求体解析与静态资源

```javascript
// 解析 JSON 与表单
app.use(express.json({ limit: '1mb' }));
app.use(express.urlencoded({ extended: true }));

// 静态资源：访问 /static/logo.png -> public/logo.png
app.use('/static', express.static('public'));

// 带缓存选项
app.use('/files', express.static('uploads', { maxAge: '1d' }));
```

## RESTful 接口实战

```javascript
const express = require('express');
const router = express.Router();

// 内存数据（真实项目替换为数据库）
let users = [
  { id: 1, name: 'Alice', age: 20 },
  { id: 2, name: 'Bob', age: 25 },
];
let seq = 3;

// 列表
router.get('/', (req, res) => {
  res.json({ code: 0, data: users });
});

// 详情
router.get('/:id', (req, res) => {
  const user = users.find((u) => u.id === Number(req.params.id));
  if (!user) return res.status(404).json({ code: 404, message: '用户不存在' });
  res.json({ code: 0, data: user });
});

// 新增
router.post('/', (req, res) => {
  const { name, age } = req.body;
  const user = { id: seq++, name, age };
  users.push(user);
  res.status(201).json({ code: 0, data: user });
});

// 更新
router.put('/:id', (req, res) => {
  const user = users.find((u) => u.id === Number(req.params.id));
  if (!user) return res.status(404).json({ code: 404, message: '用户不存在' });
  Object.assign(user, req.body);
  res.json({ code: 0, data: user });
});

// 删除
router.delete('/:id', (req, res) => {
  users = users.filter((u) => u.id !== Number(req.params.id));
  res.status(204).end();
});

module.exports = router;
```

## 错误处理与统一响应封装

```javascript
// utils/response.js
// 统一响应结构
function ok(res, data = null, message = 'success') {
  res.json({ code: 0, message, data });
}

function fail(res, message = 'error', code = 1, status = 400) {
  res.status(status).json({ code, message, data: null });
}

module.exports = { ok, fail };

// 业务异常类
class HttpError extends Error {
  constructor(message, status = 400) {
    super(message);
    this.status = status;
  }
}
module.exports.HttpError = HttpError;
```

```javascript
// 全局错误处理（放在最后）
app.use((err, req, res, next) => {
  const status = err.status || 500;
  res.status(status).json({
    code: status,
    message: err.message || '服务器内部错误',
    data: null,
  });
});

// 未匹配路由 404
app.use((req, res) => {
  res.status(404).json({ code: 404, message: '接口不存在' });
});
```

!!! tip
    使用 `express-async-errors`（或 Express 5）让 `async` 路由抛出的异常也能进入错误处理中间件。

## 参数校验 / Validation

### express-validator

```bash
npm install express-validator
```

```javascript
const { body, param, validationResult } = require('express-validator');

const rules = [
  body('name').isString().trim().notEmpty().withMessage('name 必填'),
  body('age').isInt({ min: 0, max: 150 }).withMessage('age 需为 0-150 整数'),
  param('id').isInt().withMessage('id 必须为整数'),
];

// 校验并统一返回错误
function validate(req, res, next) {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ code: 422, errors: errors.array() });
  }
  next();
}

router.post('/', rules, validate, (req, res) => { /* ... */ });
```

### zod（TypeScript 友好）

```javascript
const { z } = require('zod');

const userSchema = z.object({
  name: z.string().min(1, 'name 必填'),
  age: z.number().int().min(0).max(150),
});

function validateBody(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({ code: 422, errors: result.error.issues });
    }
    req.body = result.data;   // 使用解析后的数据
    next();
  };
}

router.post('/', validateBody(userSchema), (req, res) => { /* ... */ });
```

## 连接数据库 / Database

### MySQL：mysql2

```bash
npm install mysql2
```

```javascript
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
  host: '127.0.0.1',
  user: 'root',
  password: 'secret',
  database: 'demo',
  waitForConnections: true,
  connectionLimit: 10,
});

// 参数化查询，防止 SQL 注入
const [rows] = await pool.query('SELECT * FROM users WHERE id = ?', [1]);
// 事务
const conn = await pool.getConnection();
try {
  await conn.beginTransaction();
  await conn.query('UPDATE accounts SET balance = balance - ? WHERE id = ?', [100, 1]);
  await conn.commit();
} catch (e) {
  await conn.rollback();
  throw e;
} finally {
  conn.release();
}
```

### MySQL：Sequelize（ORM）

```bash
npm install sequelize mysql2
```

```javascript
const { Sequelize, DataTypes, Model } = require('sequelize');

const sequelize = new Sequelize('demo', 'root', 'secret', {
  host: '127.0.0.1',
  dialect: 'mysql',
  logging: false,
});

class User extends Model {}
User.init(
  {
    name: { type: DataTypes.STRING, allowNull: false },
    age: DataTypes.INTEGER,
  },
  { sequelize, modelName: 'User', tableName: 'users' }
);

await sequelize.sync();
await User.create({ name: 'Alice', age: 20 });
const users = await User.findAll({ where: { age: { [Sequelize.Op.gt]: 18 } } });
```

### MongoDB：mongoose

```bash
npm install mongoose
```

```javascript
const mongoose = require('mongoose');

await mongoose.connect('mongodb://127.0.0.1:27017/demo');

const userSchema = new mongoose.Schema(
  {
    name: { type: String, required: true },
    age: Number,
  },
  { timestamps: true }
);

const User = mongoose.model('User', userSchema);

await User.create({ name: 'Alice', age: 20 });
const users = await User.find({ age: { $gt: 18 } }).lean();
const user = await User.findById('64f0...');
await User.findByIdAndUpdate(id, { age: 21 }, { new: true });
```

| 方案 | 类型 | 特点 | 适用场景 |
| --- | --- | --- | --- |
| mysql2 | 驱动 | 原生 SQL、轻量、需手写 SQL | 追求性能与可控 SQL |
| Sequelize | ORM | 模型映射、迁移、关联查询 | 关系型、团队协作 |
| mongoose | ODM | Schema 校验、中间件钩子 | 文档型、灵活结构 |

## Redis、Session 与 JWT 鉴权

### 连接 Redis

```bash
npm install ioredis
```

```javascript
const Redis = require('ioredis');
const redis = new Redis({ host: '127.0.0.1', port: 6379 });

await redis.set('key', 'value', 'EX', 60);   // 60 秒过期
const val = await redis.get('key');
await redis.del('key');
```

### Session

```bash
npm install express-session connect-redis
```

```javascript
const session = require('express-session');
const RedisStore = require('connect-redis').default;

app.use(
  session({
    store: new RedisStore({ client: redis }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: { httpOnly: true, maxAge: 24 * 3600 * 1000 },
  })
);

app.post('/login', (req, res) => {
  req.session.user = { id: 1, name: 'Alice' };   // 写入会话
  res.json({ ok: true });
});
```

### JWT 鉴权

```bash
npm install jsonwebtoken
```

```javascript
const jwt = require('jsonwebtoken');

// 签发
const token = jwt.sign({ id: 1, name: 'Alice' }, process.env.JWT_SECRET, {
  expiresIn: '2h',
});

// 校验中间件
function auth(req, res, next) {
  const header = req.headers.authorization || '';
  const token = header.startsWith('Bearer ') ? header.slice(7) : null;
  if (!token) return res.status(401).json({ code: 401, message: '未登录' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (e) {
    res.status(401).json({ code: 401, message: '登录已过期' });
  }
}

// 保护接口
app.get('/profile', auth, (req, res) => res.json({ user: req.user }));
```

!!! warning
    JWT 无法主动失效，需配合 Redis 维护黑名单或使用短期 Token + Refresh Token 方案。

| 方案 | 存储位置 | 优点 | 缺点 |
| --- | --- | --- | --- |
| Session | 服务端（Redis） | 可主动注销 | 依赖存储、分布式需共享 |
| JWT | 客户端 | 无状态、易扩展 | 无法主动失效、体积较大 |

## 环境变量（dotenv）与配置

```bash
npm install dotenv
```

```bash
# .env
PORT=3000
NODE_ENV=development
DB_HOST=127.0.0.1
JWT_SECRET=change-me
SESSION_SECRET=change-me-too
```

```javascript
// 项目最顶部加载
require('dotenv').config();

const config = {
  port: Number(process.env.PORT) || 3000,
  env: process.env.NODE_ENV || 'development',
  db: { host: process.env.DB_HOST },
  jwtSecret: process.env.JWT_SECRET,
};

module.exports = config;

// 使用
app.listen(config.port);
```

!!! warning
    将 `.env` 加入 `.gitignore`，并在仓库中提供 `.env.example` 作为模板，切勿提交真实密钥。

## 日志（morgan）

```bash
npm install morgan
```

```javascript
const morgan = require('morgan');

// 常用格式：combined / common / dev / short / tiny
app.use(morgan('dev'));

// 自定义格式：方法 路径 状态 耗时
morgan.token('body', (req) => JSON.stringify(req.body));
app.use(
  morgan(':method :url :status :response-time ms - body=:body')
);

// 写入文件（追加）
const fs = require('fs');
const accessLog = fs.createWriteStream('access.log', { flags: 'a' });
app.use(morgan('combined', { stream: accessLog }));
```

| 格式 | 说明 |
| --- | --- |
| dev | 彩色、开发调试用 |
| combined | Apache 通用格式，生产常用 |
| common | 精简版 combined |
| tiny | 极简：方法、URL、状态、耗时 |

## 推荐项目目录结构与 npm scripts

```text
my-express-app/
├── src/
│   ├── app.js              # 应用装配（中间件、路由）
│   ├── server.js           # 启动入口
│   ├── config/             # 配置（dotenv 读取）
│   ├── routes/             # 路由定义
│   ├── controllers/        # 业务处理
│   ├── services/           # 业务逻辑
│   ├── models/             # 数据模型
│   ├── middlewares/        # 自定义中间件（auth、error）
│   └── utils/              # 工具（response、logger）
├── public/                 # 静态资源
├── .env
├── .env.example
├── ecosystem.config.js     # pm2 配置
└── package.json
```

```json
{
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "pm2": "pm2 start ecosystem.config.js"
  }
}
```

!!! tip
    分层职责：`routes` 只做参数转发，`controllers` 做参数处理，`services` 承载业务逻辑，`models` 负责数据访问。

## 部署（pm2）与 Nginx 反向代理

```bash
npm install -g pm2
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'my-app',
      script: 'src/server.js',
      instances: 'max',        // 按 CPU 核数启动（需无状态）
      exec_mode: 'cluster',
      env: { NODE_ENV: 'production', PORT: 3000 },
      max_memory_restart: '500M',
    },
  ],
};
```

```bash
pm2 start ecosystem.config.js
pm2 logs my-app        # 查看日志
pm2 reload my-app      # 平滑重启
pm2 save               # 保存进程列表
pm2 startup            # 开机自启
pm2 monit              # 实时监控
```

```nginx
# Nginx 反向代理要点
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;          # 真实 IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;           # WebSocket
        proxy_set_header Connection "upgrade";
    }
}
```

!!! tip
    Cluster 模式下应用需无状态：Session 存 Redis、文件传对象存储，避免依赖单机内存。
