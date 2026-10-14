---
title: MkDocs 搭建教程
tags:
  - 教程
  - MkDocs
  - Material
---

## 介绍 / Introduction

使用 MkDocs + Material for MkDocs 搭建个人技术文档站，支持搜索、代码高亮、暗色主题等。

!!! note
    Material for MkDocs 是 MkDocs 最流行的第三方主题，功能全面。

## 安装

```bash
pip install mkdocs-material
mkdocs new my-docs
cd my-docs
```

### mkdocs.yml 关键配置

```yaml
site_name: 我的技术笔记
theme:
  name: material
  features:
    - navigation.tabs
    - search.highlight
    - content.code.copy
  palette:
    - scheme: default
      primary: indigo
      toggle:
        icon: material/weather-sunny
    - scheme: slate
      toggle:
        icon: material/weather-night
markdown_extensions:
  - admonition
  - pymdownx.highlight
  - pymdownx.superfences
```

### 本地预览与部署

```bash
mkdocs serve          # http://127.0.0.1:8000
mkdocs build          # 生成 site/ 静态文件
mkdocs gh-deploy      # 部署到 GitHub Pages
```
