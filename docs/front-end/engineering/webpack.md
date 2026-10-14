---
title: Webpack
tags:
  - 前端
  - Webpack
  - 构建工具
---

## 介绍 / Introduction

Webpack 是老牌模块打包器，拥有成熟的 loader/plugin 生态，适合复杂工程场景。

!!! note
    新项目推荐优先使用 Vite；Webpack 在 legacy 项目中仍广泛使用。

## 基础配置

### 最小可用配置

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  module: {
    rules: [
      { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ },
      { test: /\.css$/, use: ['style-loader', 'css-loader'] },
    ],
  },
};
```

### 常用命令

```bash
npx webpack --mode development
npx webpack --mode production
```
