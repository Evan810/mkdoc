---
title: 前端工程化分类
tags:
  - 前端
  - 工程化
---

## 前端工程化 / Frontend Engineering

前端工程化旨在通过工具链提升开发效率、代码质量与构建性能。本分类记录主流工程化工具的使用与原理。

!!! note
    工程化工具更新较快，建议关注官方文档获取最新 API。

## 工具对比

| 对比项 | Vite | Webpack |
| --- | --- | --- |
| 定位 | 新一代构建工具 | 老牌模块打包器 |
| 开发启动 | 基于原生 ESM，极快 | 需先打包，较慢 |
| 生产构建 | Rollup | 自研打包 |
| 配置复杂度 | 低，约定优先 | 高，配置灵活 |
| 生态 | 快速增长 | 成熟、插件丰富 |
| 适用场景 | 新项目、现代浏览器 | 复杂/legacy 项目 |

## 涵盖内容

<div class="grid cards" markdown>

- :fontawesome-solid-bolt: **Vite**
  { .card-heading }
  基于 ESM 的新一代构建工具，开发环境启动极快
  [查看 &rarr;](vite.md)

- :fontawesome-solid-box: **Webpack**
  { .card-heading }
  老牌模块打包器，loader/plugin 生态成熟
  [查看 &rarr;](webpack.md)

</div>

## 工程化关注点

- **开发体验**：热更新（HMR）、启动速度、错误提示
- **构建产物**：分包、压缩、Tree Shaking、缓存策略
- **代码质量**：ESLint、Prettier、提交规范（commitlint + husky）
- **环境管理**：多环境配置、环境变量注入、代理
- **部署**：静态资源 CDN、Nginx 配置、CI/CD

```bash
# 使用 Vite 初始化项目
npm create vite@latest my-app -- --template react-ts
```

```bash
# 使用 Webpack 构建
npx webpack --mode development
npx webpack --mode production
```

!!! tip
    新项目优先选择 Vite；维护存量项目或需要高度定制打包流程时再使用 Webpack。
