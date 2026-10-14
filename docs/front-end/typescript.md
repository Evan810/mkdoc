---
title: TypeScript
tags:
  - 前端
  - TypeScript
  - 类型系统
---

## 介绍 / Introduction

TypeScript 是 JavaScript 的超集，引入静态类型系统，在编译期发现潜在错误。

!!! note
    推荐配合 ESLint + Prettier 使用，保持类型与代码风格一致。

## 类型基础

### 基础类型与接口

```typescript
interface User {
  id: number;
  name: string;
  email?: string;
}

function greet(user: User): string {
  return `Hello, ${user.name}!`;
}

const u: User = { id: 1, name: 'Alice' };
console.log(greet(u));
```

### 泛型示例

```typescript
function identity<T>(value: T): T {
  return value;
}
```

```bash
# 编译 TypeScript
tsc --init
npx tsc
```
