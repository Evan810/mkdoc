---
title: Vue
tags:
  - 前端
  - Vue
  - JavaScript
---

## 介绍 / Introduction

Vue.js 是一套渐进式 JavaScript 框架，以响应式数据绑定与组合式 API 为核心特性。

!!! note
    示例代码基于 Vue 3 + `<script setup>` 写法。

## 组合式 API

### 响应式状态

```vue
<script setup>
import { ref, computed } from 'vue';

const count = ref(0);
const doubled = computed(() => count.value * 2);
</script>

<template>
  <button @click="count++">{{ count }} → {{ doubled }}</button>
</template>
```

### 项目创建

```bash
npm create vue@latest
npm install && npm run dev
```
