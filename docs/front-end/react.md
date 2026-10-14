---
title: React
tags:
  - 前端
  - React
  - JavaScript
---

## 介绍 / Introduction

React 是 Meta 推出的声明式、组件化 JavaScript 库，用于构建用户界面。

!!! note
    本页为占位内容，后续将补充 Hooks、状态管理（Redux/Zustand）、服务端组件等主题。

## 核心概念

### 函数式组件与 Hooks

```jsx
import { useState, useEffect } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

### JSX 语法

JSX 是 JavaScript 的语法扩展，可以直观地描述 UI 结构。

```bash
# 创建 React 项目
npm create vite@latest my-react-app -- --template react
```
