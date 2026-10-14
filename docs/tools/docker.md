---
title: Docker
tags:
  - 工具
  - Docker
  - 容器化
---

## 介绍 / Introduction

Docker 容器化工具速查，覆盖镜像构建、容器运行、数据卷与 Docker Compose 编排。

!!! note
    生产环境建议使用多阶段构建减小镜像体积，并搭配 .dockerignore 排除无关文件。

## 基础命令

```bash
docker pull nginx:alpine          # 拉取镜像
docker run -d -p 80:80 nginx      # 后台运行容器
docker ps                         # 查看运行中容器
docker logs -f <container_id>     # 查看日志
docker exec -it <id> bash         # 进入容器
docker stop <id> && docker rm <id>
```

## Dockerfile 示例（多阶段）

```dockerfile
# 构建阶段
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 运行阶段
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```

```bash
docker build -t my-app:latest .
docker run -d -p 8080:80 my-app
```
