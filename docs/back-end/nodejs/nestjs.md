---
title: NestJS
tags:
  - 后端
  - Node.js
  - NestJS
  - TypeScript
---

## 介绍 / Introduction

NestJS 是基于 TypeScript 的企业级 Node.js 框架，借鉴 Angular 风格的装饰器与依赖注入体系。

!!! note
    NestJS 底层默认使用 Express，可切换为 Fastify 以获得更高性能。

## 快速创建

```bash
npm i -g @nestjs/cli
nest new my-nest-app
cd my-nest-app && npm run start:dev
```

## 控制器示例

```typescript
import { Controller, Get, Param, Post, Body } from '@nestjs/common';

@Controller('users')
export class UsersController {

  @Get()
  findAll() {
    return [{ id: 1, name: 'Alice' }];
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return { id: +id, name: 'Alice' };
  }

  @Post()
  create(@Body() body: { name: string }) {
    return { id: Date.now(), ...body };
  }
}
```
