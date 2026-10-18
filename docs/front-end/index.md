---
title: 前端开发分类
tags:
  - 前端
---

## 前端开发 / Frontend Development

本分类汇集前端开发相关的学习笔记与实践经验，涵盖主流框架、工程化工具与语言特性。

!!! note
    目录持续更新中，欢迎通过 GitHub 提交补充内容。

## 技术栈概览

| 领域 | 主流方案 | 说明 |
| --- | --- | --- |
| 框架 | React / Vue | 组件化 UI 开发 |
| 语言 | TypeScript / JavaScript | 静态类型提升可维护性 |
| 构建工具 | Vite / Webpack | 开发服务器与打包 |
| 状态管理 | Redux Toolkit / Zustand / Pinia | 跨组件状态共享 |
| 路由 | React Router / Vue Router | 单页应用导航 |
| 样式方案 | CSS Modules / Sass / Tailwind | 样式组织与复用 |
| 请求库 | axios / fetch / TanStack Query | 数据请求与缓存 |

## 主要内容

<div class="grid cards" markdown>

- :fontawesome-brands-react: **React**
  { .card-heading }
  函数式组件、Hooks、状态管理与性能优化
  [查看 &rarr;](react.md)

- :fontawesome-brands-vuejs: **Vue**
  { .card-heading }
  组合式 API、响应式原理、Pinia 状态管理
  [查看 &rarr;](vue.md)

- :fontawesome-solid-code: **TypeScript**
  { .card-heading }
  类型系统、泛型、工具类型与工程实践
  [查看 &rarr;](typescript.md)

- :fontawesome-solid-screwdriver-wrench: **工程化**
  { .card-heading }
  Vite、Webpack 等构建工具的使用与原理
  [查看 &rarr;](engineering/index.md)

</div>

## 学习路线

```text
HTML / CSS / JavaScript 基础
        ↓
TypeScript 类型系统
        ↓
框架入门（React 或 Vue 二选一）
        ↓
路由 + 状态管理 + 请求封装
        ↓
工程化（Vite / Webpack）
        ↓
性能优化与工程实践
```

```bash
# 创建 Vue 项目
npm create vue@latest

# 创建 React 项目
npm create vite@latest my-react-app -- --template react-ts
```

!!! tip
    建议先深入掌握一门框架，再横向对比学习另一门，重点理解「响应式 / 状态更新」的差异而非 API 细节。
