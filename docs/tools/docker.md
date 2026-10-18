---
title: Docker
tags:
  - 工具
  - Docker
  - 容器化
---

# Docker 常用命令使用说明

> 按"从拿到镜像到排查容器问题"的实际流程组织，覆盖镜像、容器、网络、数据卷、Compose、日志与排障。带 `< >` 的部分需替换为实际值，`[ ]` 表示可选参数。示例基于 Docker Engine 20.10+ 与 Docker Compose V2（`docker compose` 子命令形式）。

---

## 一、安装状态与基本信息

```bash
docker version                       # 客户端与服务端版本（服务未起会报连接错误）
docker info                          # 总览：镜像/容器数、存储驱动、内核、代理配置
systemctl status docker              # daemon 运行状态
systemctl enable docker              # 开机自启
docker context ls                    # 当前连接的 Docker 端点（本机/远程）
echo $DOCKER_HOST                    # 远程端点环境变量（如 tcp:// 或 ssh://）
docker --help                        # 全部子命令
docker <命令> --help                 # 单个命令的完整参数，例：docker run --help
```

> 免 sudo 使用 docker：`sudo usermod -aG docker $USER`（**必须带 `-a`**，否则会把该用户的其他附加组清空），执行后需重新登录或 `newgrp docker` 生效。这等于给用户 root 级权限，共享机器上谨慎开放。

---

## 二、镜像管理

### 查找与拉取

```bash
docker search nginx                          # 查 Docker Hub（推荐去官网看官方镜像）
docker pull nginx                            # 拉最新标签（等同 nginx:latest）
docker pull nginx:1.27-alpine                # ⚠️ 生产环境务必指定明确标签
docker pull mysql:8.0.36                     # 指定版本
docker pull registry.cn-xxx.aliyuncs.com/ns/app:v1.2   # 私有仓库镜像
```

### 查看、标记、删除

```bash
docker images                            # 本机镜像列表
docker images -q                          # 只输出镜像 ID（脚本用）
docker image ls --filter "dangling=true"  # 列出无标签的悬空镜像
docker inspect nginx:1.27-alpine          # 详细信息：层、环境变量、默认启动命令
docker history nginx:1.27-alpine          # 逐层构建过程（排查镜像体积很有用）
docker tag myapp:latest registry/ns/myapp:v1.0   # 打标签
docker rmi nginx:1.27-alpine              # 删除标签（被容器占用会失败）
docker rmi -f <镜像ID>                     # 强制删除
docker image prune -a                     # 清理所有未被容器使用的镜像（⚠️ 交互确认）
docker image prune                        # 只清理悬空镜像，更安全
```

### 导出与导入

```bash
docker save -o nginx.tar nginx:1.27-alpine    # 打包镜像为 tar（离线分发）
docker load -i nginx.tar                      # 从 tar 载入本机
docker export <容器ID> > rootfs.tar           # 导出容器文件系统（丢失分层与元数据）
docker import rootfs.tar myapp:base           # 从文件系统 tar 创建镜像
```

> `save/load` 保留镜像完整结构与默认启动命令；`export/import` 只是文件系统快照，导入时必须自己指定启动命令，两者不要混用。

---

## 三、容器生命周期

### 创建并启动（run）

```bash
docker run -d --name web -p 8080:80 nginx:1.27-alpine
# -d 后台运行   --name 命名   -p 主机端口:容器端口

docker run -it ubuntu:22.04 bash          # 交互式进入（一次性调试用）
docker run -d --restart=unless-stopped --name redis redis:7-alpine   # 开机/崩溃自动重启
docker run -d -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=StrongPass123 \
  -e MYSQL_DATABASE=appdb \
  -v /data/mysql:/var/lib/mysql \
  --name mysql mysql:8.0.36                # 环境变量 + 数据持久化
docker run -it --rm alpine sh             # --rm 退出即删除，适合临时任务
docker run -d --network mynet --add-host api.local:10.0.0.5 app   # 指定网络与自定义 hosts
docker run -d -m 512m --cpus 1.5 app      # 限制内存与 CPU
docker run -d -v /host/certs:/etc/certs:ro app   # 只读挂载
docker run -d --privileged app            # ⚠️ 特权模式，容器等同宿主机 root，仅调试用
```

### 查看与运行中管理

```bash
docker ps                                # 运行中的容器
docker ps -a                             # 全部（含已退出，重点看 STATUS 与退出码）
docker ps --format '{{.Names}}\t{{.Status}}\t{{.Ports}}'   # 自定义输出
docker stats                             # 实时 CPU/内存/网络（--no-stream 只取一次）
docker top <容器>                         # 容器内进程
docker inspect <容器>                      # JSON 全量信息
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <容器>   # 取 IP
docker port <容器>                        # 端口映射关系
docker diff <容器>                        # 相对镜像被改动了哪些文件
```

### 启停、进入、执行

```bash
docker stop web                          # 优雅停止（发 SIGTERM，默认等 10s）
docker stop -t 30 web                    # 自定义优雅等待秒数
docker kill web                          # 立即 SIGKILL（不给清理机会）
docker start web                         # 启动已存在的容器
docker restart web                       # 重启
docker pause / unpause web               # 冻结/恢复进程（调试用，少见）
docker rename old new                    # 改名
docker exec -it web bash                 # 进入运行中容器（镜像无 bash 时用 sh）
docker exec -u root -it web bash         # 以指定用户进入
docker exec web ls /app                  # 不进入直接执行命令
docker wait web                          # 阻塞直到容器退出，返回退出码（脚本编排用）
```

### 文件复制与删除

```bash
docker cp web:/app/config.yaml ./        # 容器 → 主机
docker cp ./app.jar web:/app/            # 主机 → 容器
docker rm web                            # 删除已停止的容器
docker rm -f web                         # 强制删除运行中的容器
docker container prune                   # 清理所有已停止容器（⚠️ 先 docker ps -a 确认清单）
docker logs web                          # 查看日志
docker logs -f --tail 100 web            # 跟踪最近 100 行起
docker logs --since 30m web              # 最近 30 分钟
docker logs -t web                       # 加时间戳
```

> ⚠️ `docker rm` 会删除容器可写层，容器内直接写下的数据（未挂载卷的部分）会永久丢失。生产数据一律放数据卷或宿主机目录。

---

## 四、数据卷与备份

```bash
docker volume create appdata             # 命名卷（推荐，由 Docker 管理路径）
docker volume ls                         # 列表
docker volume inspect appdata            # 查看实际挂载点（一般在 /var/lib/docker/volumes/）
docker run -d -v appdata:/var/lib/mysql mysql:8.0.36    # 使用命名卷
docker run -d -v /data/pg:/var/lib/postgresql/data pg   # 绑定挂载宿主机目录
docker run -d --volumes-from mysql0 app  # 复用其他容器已挂载的卷（参数是容器名）
docker volume rm appdata                 # 删除卷（⚠️ 数据不可恢复）
docker volume prune                      # 清理未被使用的卷

# 卷备份与恢复（标准套路）
docker run --rm -v appdata:/from -v "$(pwd)":/to alpine tar czf /to/appdata.tar.gz -C /from .
docker run --rm -v appdata:/to -v "$(pwd)":/from alpine tar xzf /from/appdata.tar.gz -C /to
```

---

## 五、网络

```bash
docker network ls                        # 网络列表（bridge/host/none + 自定义）
docker network create --driver bridge mynet    # 自定义网络（容器间可用容器名互访）
docker network inspect mynet             # 成员、IP、驱动详情
docker network connect mynet web         # 把容器接入网络
docker network disconnect mynet web
docker network rm mynet
docker run -d --network mynet --name api app        # 加入网络
# 同一自定义网络内直接用 http://api:8080 访问；默认 bridge 网络需 --link（已废弃）

# 容器内测试连通性
docker exec -it api sh -c "getent hosts db"     # DNS 解析
docker exec -it api sh -c "nc -zv db 5432"      # 端口可达
```

| 驱动 | 用途 |
|---|---|
| bridge | 默认，单机容器互联 |
| host | 直接用宿主机网络栈，性能最好但端口会冲突 |
| none | 无网络，隔离任务 |
| overlay | Swarm 多机通信 |

---

## 六、镜像构建

```bash
docker build -t myapp:1.0 .                    # 用当前目录 Dockerfile 构建
docker build -t myapp:1.0 -f Dockerfile.prod . # 指定文件
docker build --no-cache -t myapp:1.0 .         # 忽略缓存（怀疑缓存污染时）
docker build -t myapp:1.0 --build-arg ENV=prod .   # 传入构建参数
docker build -t myapp:1.0 --target builder .   # 多阶段构建指定目标阶段
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:1.0 .  # 跨架构构建
docker images -f "reference=myapp:1.0"         # 查构建结果与体积
```

### 最小 Dockerfile 骨架

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
CMD ["node", "server.js"]
```

> 关键习惯：基础镜像固定版本；先 `COPY` 依赖清单再 `COPY` 源码（否则改一行代码就重装全部依赖，缓存失效）；构建上下文中无关大文件写进 `.dockerignore`；`latest` 不要用于生产。

---

## 七、Docker Compose（多容器编排）

```bash
docker compose config                     # 校验并输出最终合并后的配置（排错第一步）
docker compose up -d                      # 按 compose.yml 创建并后台启动
docker compose up -d --build              # 启动前重新构建镜像
docker compose up -d web                  # 只启动某个服务
docker compose ps                         # 本项目容器状态
docker compose logs -f --tail 100         # 全部服务日志
docker compose logs -f web                # 单个服务日志
docker compose exec web bash              # 进入某服务容器
docker compose stop                       # 停止（保留容器）
docker compose start
docker compose restart web
docker compose down                       # 停止并删除容器与网络（卷保留）
docker compose down -v                    # ⚠️ 同时删除数据卷，数据库数据会丢
docker compose pull                       # 拉取最新基础镜像
docker compose build --no-cache web
docker compose top
```

```yaml
# compose.yml 最小示例
services:
  web:
    build: .
    ports: ["8080:80"]
    environment:
      - DB_HOST=db
    depends_on: [db]
    restart: unless-stopped
  db:
    image: mysql:8.0.36
    environment:
      MYSQL_ROOT_PASSWORD: StrongPass123
    volumes:
      - dbdata:/var/lib/mysql
    restart: unless-stopped
volumes:
  dbdata:
```

> 老版 `docker-compose`（带连字符、Python 实现）与新版 `docker compose` 命令不通用，但配置文件基本兼容；新装系统一律用 `docker compose`。

---

## 八、清理与磁盘治理

```bash
docker system df                         # 各类资源占用（镜像/容器/卷/构建缓存）
docker system df -v                      # 展开到每项明细，定位大对象
docker system prune                      # 清停止容器 + 悬空镜像 + 未用网络 + 构建缓存
docker system prune -a                   # ⚠️ 追加删除所有未被使用镜像
docker system prune -a --volumes         # ⚠️ 连未使用卷一起删，务必确认无状态服务在跑
docker builder prune                     # 只清构建缓存（构建机磁盘满常因它）
docker image inspect --format '{{.Size}}' <镜像>    # 单个镜像字节数
```

> 磁盘突然变满的排查顺序：`df -h` → `docker system df` → 若是 overlay2 目录大则 `docker builder prune` 与 `docker image prune`；若是容器日志，检查 `/var/lib/docker/containers/*/*-json.log` 体积并配置日志轮转（见第十节）。

---

## 九、私有仓库与推送

```bash
docker login registry.example.com                 # 登录（私有仓库需带域名）
docker tag myapp:1.0 registry.example.com/ns/myapp:1.0
docker push registry.example.com/ns/myapp:1.0
docker logout registry.example.com
docker manifest inspect registry.example.com/ns/myapp:1.0    # 查多架构清单
```

> 凭据保存在 `~/.docker/config.json`（明文 base64）。共享机器上建议改用 `docker-credential-*` 助手，或每次操作后 `docker logout`。

---

## 十、常见故障对照

| 现象 | 起手排查 |
|---|---|
| 容器不断重启 | `docker ps -a` 看退出码 → `docker logs --tail 100 <容器>` → `docker inspect -f '{{.State.ExitCode}} {{.State.OOMKilled}}' <容器>` |
| `docker: Cannot connect to the Docker daemon` | `systemctl status docker`、`journalctl -u docker --since "10 min"`；再确认当前用户是否在 docker 组 |
| 端口映射不生效 | `docker port <容器>` 确认映射存在 → `ss -tlnp \| grep <端口>` 看宿主机是否被其他进程占用 → 云安全组是否放行 |
| 容器内访问不到宿主机服务 | 用 `host.docker.internal`（Docker Desktop）或宿主机局域网 IP；Linux 上可 `--network host` 或 `ip route` 取网关地址 |
| 容器时间/时区不对 | `docker exec <容器> date` → 加 `-e TZ=Asia/Shanghai` 或挂载 `/etc/localtime` |
| 改配置后行为没变 | 镜像未重建：`docker compose up -d --build`；或 `docker build --no-cache` |
| 构建卡在拉取基础镜像 | `docker info` 看 Registry Mirrors；配置 `/etc/docker/daemon.json` 后 `systemctl restart docker` |
| 内存超限被杀（OOMKilled=true） | 加 `-m` 限制前先确认应用真实用量；`docker stats` 观察，必要时调大或分库拆分 |
| 磁盘被日志写满 | 见第八节；长期方案在 `daemon.json` 配 `"log-driver":"json-file","log-opts":{"max-size":"50m","max-file":"3"}` |
| 卷挂载后权限报错 | 容器内用户 uid 与宿主机目录属主不一致：`ls -n` 对比 uid，用 `chown` 或 `--user` 对齐 |
| `docker rm` 提示卷仍被占用 | 先删卷：`docker volume rm <卷名>`；或 `docker rm -v <容器>` 一并删除该容器的匿名卷 |
| 无法进入容器（镜像无 bash） | 用 `docker exec -it <容器> sh`；极简镜像（distroless）无 shell，需另起一个带工具的容器挂载同一文件系统的卷来排查 |

---

## 十一、高频场景速查

| 场景 | 命令 |
|---|---|
| 起一个临时 MySQL 试数据 | `docker run -d --name t-mysql -e MYSQL_ROOT_PASSWORD=root123 -p 3307:3306 mysql:8.0.36` |
| 快速跑一个命令即退出 | `docker run --rm alpine date` |
| 看某容器占用多少资源 | `docker stats --no-stream <容器>` |
| 进容器改文件后重新打包 | `docker exec` 修改 → `docker commit <容器> myapp:debug`（仅临时救急，正式改动应回写 Dockerfile） |
| 查某镜像的默认启动命令 | `docker inspect -f '{{.Config.Cmd}} {{.Config.Entrypoint}}' <镜像>` |
| 列出所有容器及其端口 | `docker ps -a --format '{{.Names}}: {{.Ports}}'` |
| 一次性清理停止的容器 | `docker container prune` |
| 挂载本地代码热调试 | `docker run -d -v "$(pwd)":/app -w /app node:20 npm run dev` |
| 把 Compose 项目迁移到新机 | 复制 `compose.yml` + 镜像（`docker save`）+ 卷备份（第四节 tar 命令） |
| 检查容器健康状态 | `docker inspect -f '{{.State.Health.Status}}' <容器>`；`docker events` 实时看事件 |
| 看 Docker 版本与存储驱动 | `docker info --format '{{.ServerVersion}} / {{.Driver}}'` |

---

## 十二、使用建议

1. **永远指定明确标签**：`nginx:1.27-alpine` 而不是 `nginx`；`latest` 会在某次重建后悄悄换版本，是"昨天还好好的"最常见原因。
2. **数据一律出容器**：命名卷或宿主机目录挂载，并定期按第四节做卷备份；删容器前先确认数据不在可写层。
3. **restart 策略选对**：一般服务用 `unless-stopped`；`always` 会覆盖你手动 `docker stop` 的意图；调试期先不设。
4. **资源限制与日志轮转提前配**：单机多容器时 `--cpus` / `-m` 与 `log-opts` 一次性写进 `daemon.json`，比事后救火省事。
5. **别在生产用 `--privileged`**：需要设备访问时优先精确的 `--device` 或 `cap_add`，最小权限原则。
6. **改完配置记得 `config` 校验**：`docker compose config` 能在启动前抓出缩进、服务名拼错、变量未定义等绝大多数低级问题。
7. **镜像瘦身靠多阶段与 alpine/slim 基础镜像**：`docker history <镜像>` 逐层找体积元凶，常见是构建工具链和缓存没清（`npm ci` 后删缓存、`apt-get install -y --no-install-recommends` + `rm -rf /var/lib/apt/lists/*`）。

---


