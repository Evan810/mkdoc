---
title: 图标资源
tags:
  - 工具
  - 图标
  - 设计
---

## 介绍 / Introduction

常用免费图标资源与使用方式汇总，包含在线库、React/Vue 组件库与 CDN 引入。

!!! note
    使用前请确认各图标库的 License（通常为 MIT / Apache 2.0 / CC0）。

## 推荐图标库

| 名称 | 风格 | 地址 |
|------|------|------|
| Material Symbols | Google 风格 | fonts.google.com/icons |
| Tabler Icons | 简约线性 | tabler-icons.io |
| Lucide | 轻量现代 | lucide.dev |
| Font Awesome | 经典全面 | fontawesome.com |

## 使用示例

### CDN 引入 Font Awesome

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
<i class="fa-solid fa-rocket"></i>
```

### React 使用 Lucide

```bash
npm install lucide-react
```

```jsx
import { Rocket } from 'lucide-react';
<Rocket size={24} color="#6366f1" />
```
