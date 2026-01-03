# Docker 学习笔记

> 本文档记录了 Docker 学习的重点问题和知识点，便于复习和查阅。

---

## 📚 第1章：Docker 介绍

### 🎯 核心问题

#### 1. 内核是什么？
**问题**：Docker 容器"共享内核"，内核是什么？

**答案**：
```
内核（Kernel）= 操作系统的心脏 ⭐
├─ 管理硬件资源（CPU、内存、硬盘、网络）
├─ 调度进程（决定哪个程序什么时候运行）
├─ 提供系统调用接口
└─ 实现隔离与安全

类比：内核 = 大楼的"中央控制系统"
```

**关键点**：
- 内核是操作系统的核心
- Docker 需要 Linux 内核的特性（Namespaces、cgroups）
- macOS 内核是 Darwin（BSD），Windows 内核是 Windows Kernel，都不是 Linux

#### 2. Docker 为什么是 Client-Server 架构？
**问题**：为什么 Docker 不把客户端和守护进程合并成一个程序？

**答案**：
```
架构优势：
┌─────────────────────────────────────┐
│  Docker Client（客户端）             │
│  └─ 轻量级，发送命令                  │
│         │                            │
│         │ REST API 请求               │
│         ▼                            │
│  Docker Daemon（守护进程/服务器）     │
│  └─ 持续运行，执行所有操作             │
│  └─ 维护状态，管理容器和镜像           │
└─────────────────────────────────────┘

优势：
├─ ✅ 性能：客户端轻量快速
├─ ✅ 状态管理：守护进程保持状态
├─ ✅ 远程管理：可以管理远程服务器
└─ ✅ API 标准化：统一的 REST API
```

#### 3. Docker 和虚拟机的区别
**问题**：为什么 Docker 比虚拟机更轻量？虚拟机需要完整 OS，Docker 共享内核

**答案**：
```
虚拟机架构：
┌─────────────────────────────────────┐
│  应用1 │ 应用2 │ 应用3               │
│  ─────────────────────────────────  │
│ Guest OS 1 │ Guest OS 2 │ Guest OS 3│  ← 每个都要完整 OS
│  ─────────────────────────────────  │
│      Hypervisor（虚拟化管理层）      │
│  ─────────────────────────────────  │
│         Host OS                      │
└─────────────────────────────────────┘
资源：每个 VM 需要几 GB 内存

Docker 容器架构：
┌─────────────────────────────────────┐
│  容器1 │ 容器2 │ 容器3               │
│  ─────────────────────────────────  │
│      Docker Engine                  │
│  ─────────────────────────────────  │
│     共享 Host OS 内核 ⭐              │
│  ─────────────────────────────────  │
│         Host OS                      │
└─────────────────────────────────────┘
资源：每个容器只需要几十到几百 MB
```

#### 4. macOS/Windows 为什么需要虚拟机？
**问题**：为什么 macOS 上需要 Docker Desktop（包含虚拟机）？

**答案**：
```
原因链条：
macOS 内核 = Darwin（BSD）❌
    │
    │ Docker 需要 Linux 内核特性
    │（Namespaces、cgroups 等）
    │
    ▼
需要 Linux 内核环境 ✅
    │
    ▼
解决方案：运行 Linux 虚拟机
    │
    ├─ macOS: Docker Desktop（包含轻量级 Linux VM）
    │   └─ 使用 HyperKit 或 Apple Virtualization Framework
    │
    └─ Windows: Docker Desktop（使用 WSL 2）
        └─ Windows Subsystem for Linux 2
```

**关键点**：
- Docker 依赖 Linux 内核特性
- macOS/Windows 没有 Linux 内核
- 需要在虚拟机中运行 Linux 来提供 Linux 内核环境
- Linux 系统：直接运行（有 Linux 内核）✅
- macOS/Windows：通过虚拟机运行 Linux 内核 ✅

### 📝 重要知识点

```
Docker 核心概念：
┌─────────────────────────────────────┐
│  镜像（Image）                       │
│  └─ 只读模板，包含应用和依赖          │
│         │                            │
│         │ docker run                 │
│         ▼                            │
│  容器（Container）                   │
│  └─ 镜像的运行实例                    │
│  └─ 可读写的容器层                    │
│         │                            │
│         │ docker commit（不推荐）     │
│         ▼                            │
│  Dockerfile                          │
│  └─ 构建镜像的脚本                    │
│         │                            │
│         │ docker push                │
│         ▼                            │
│  镜像仓库（Registry）                │
│  └─ Docker Hub / 私有仓库            │
└─────────────────────────────────────┘
```

**Docker 四个特点**：
1. **轻量性**：共享内核，不需要完整 OS
2. **可移植性**：一次打包，到处运行
3. **隔离性**：每个容器独立，互不干扰
4. **标准化**：统一的接口和工具链

---

## ⚙️ 第2章：Docker 安装

### 🎯 核心问题

#### 5. Windows 也需要虚拟机吗？
**问题**：Windows 也是需要虚拟机才能运行 Docker 吗？

**答案**：
```
三种系统的 Docker 运行方式：

Linux 系统：
└─ 直接运行 ✅（有 Linux 内核）

macOS 系统：
└─ Docker Desktop + 传统虚拟机 ✅
   └─ HyperKit / Apple Virtualization Framework

Windows 系统：
└─ Docker Desktop + WSL 2 ✅
   └─ Windows Subsystem for Linux 2（特殊的虚拟化技术）
   └─ 比传统虚拟机更轻量、更快
```

**关键点**：
- 所有非 Linux 系统都需要虚拟化技术来提供 Linux 内核
- Windows 使用 WSL 2（比传统 VM 更高效）
- macOS 使用传统虚拟机（但经过优化）

### 📝 重要知识点

```
安装方式：
├─ Linux: 包管理器安装（yum/apt）
├─ macOS: Docker Desktop（图形化工具）
└─ Windows: Docker Desktop（需要 WSL 2）

验证安装：
└─ docker --version
└─ docker info
└─ docker run hello-world
```

---

## 🛠️ 第3章：Docker 使用

### 🎯 核心问题

#### 6. 镜像分层存储是什么？
**问题**：`docker pull nginx` 时显示的多层（Layer）是什么意思？

**答案**：
```
镜像分层存储（Layered Storage）：

nginx:latest 镜像结构：
┌─────────────────────────────────────┐
│  Layer 5（应用层）                    │
│  └─ nginx 程序和配置                  │
├─────────────────────────────────────┤
│  Layer 4（依赖库层）                  │
│  └─ nginx 需要的库                    │
├─────────────────────────────────────┤
│  Layer 3（系统工具层）                │
├─────────────────────────────────────┤
│  Layer 2（基础系统层）                │
├─────────────────────────────────────┤
│  Layer 1（基础 Linux 系统）           │
└─────────────────────────────────────┘

优势：
├─ ✅ 共享基础层，节省空间
├─ ✅ 复用已有层，加速下载
└─ ✅ 只更新变化的层，加速构建
```

#### 7. 容器可写层的作用
**问题**：在容器中安装 Python 包，包会存储在容器层吗？

**答案**：
```
容器运行时的结构：
┌─────────────────────────────────────┐
│  容器可写层（Container Layer）⭐      │
│  └─ 你安装的包、创建的文件            │
│  └─ 每个容器独立                     │
├─────────────────────────────────────┤
│  镜像层（只读，所有容器共享）          │
│  └─ Layer 5, 4, 3, 2, 1...          │
└─────────────────────────────────────┘

关键点：
├─ ✅ 安装的包存储在容器可写层
├─ ✅ 每个容器的可写层独立
├─ ✅ 容器删除后，可写层数据丢失
└─ ✅ 多个容器共享底层镜像层
```

#### 8. docker run 命令参数详解
**问题**：`docker run -it -v /Users/yourname/data:/data python:3.9 /bin/bash` 各参数含义

**答案**：
```
docker run [OPTIONS] IMAGE [COMMAND]

参数解析：
├─ -it: 交互式终端
│   ├─ -i: Interactive（保持 STDIN 打开）
│   └─ -t: TTY（分配伪终端）
│
├─ -v: 数据卷挂载
│   └─ 格式：-v 主机路径:容器路径
│   └─ /Users/yourname/data:/data
│       ├─ 主机：/Users/yourname/data
│       └─ 容器：/data
│
├─ python:3.9: 镜像名称和标签
│
└─ /bin/bash: 要执行的命令

完整含义：
创建容器，使用 python:3.9 镜像
├─ 交互式模式运行
├─ 挂载主机的 /Users/yourname/data 到容器的 /data
└─ 启动 bash shell
```

#### 9. 仓库为什么存储镜像而不是 Dockerfile？
**问题**：用 Dockerfile 写了构建过程，运行 docker build 后，系统会多出什么文件？为什么仓库不是存储 Dockerfile？

**答案**：
```
docker build 执行过程：
┌─────────────────────────────────────┐
│  步骤1：读取 Dockerfile              │
│         │                            │
│         ▼                            │
│  步骤2：逐层构建镜像                 │
│  ├─ Layer 1: FROM python:3.9        │
│  ├─ Layer 2: RUN pip install...     │
│  ├─ Layer 3: COPY script.py         │
│  └─ ...                              │
│         │                            │
│         ▼                            │
│  步骤3：生成镜像文件                 │
│  └─ 存储在 /var/lib/docker/          │
└─────────────────────────────────────┘

为什么仓库存储镜像而不是 Dockerfile？
├─ 镜像可以直接运行（结果）
├─ Dockerfile 需要构建（过程）
├─ 镜像保证一致性（特定版本）
└─ 镜像包含所有依赖（无需网络下载）
```

#### 10. 容器停止后可以再启动吗？
**问题**：容器运行、停止后，下次还可以启动吗？

**答案**：
```
容器生命周期：
创建 → 运行 → 停止 → 可以再次启动 ✅
              │
              ▼
           删除容器 ❌（删除后无法再启动）

命令对比：
├─ docker run: 创建新容器并启动
├─ docker start: 启动已存在的容器
├─ docker stop: 停止容器（保留容器和数据）
└─ docker rm: 删除容器（删除后无法再启动）

数据保留：
├─ 容器停止：数据保留 ✅
├─ 容器重启：数据保留 ✅
└─ 容器删除：容器层数据丢失，数据卷数据保留 ✅
```

### 📝 重要知识点

```
镜像操作流程：
┌─────────────────────────────────────┐
│  docker pull IMAGE                  │
│  └─ 从仓库下载镜像                   │
│         │                            │
│         ▼                            │
│  docker images                      │
│  └─ 查看本地镜像列表                 │
│         │                            │
│         ▼                            │
│  docker run IMAGE                   │
│  └─ 创建并运行容器                   │
│         │                            │
│         ▼                            │
│  docker ps                           │
│  └─ 查看运行中的容器                 │
└─────────────────────────────────────┘

容器操作：
├─ docker run: 创建并运行
├─ docker start/stop: 启动/停止
├─ docker exec: 在运行中的容器执行命令
├─ docker logs: 查看日志
└─ docker rm: 删除容器
```

---

## 📝 第4章：Dockerfile

### 🎯 核心问题

#### 11. CMD 指令的作用
**问题**：`CMD ["python", "data_analysis.py"]` 什么意思？

**答案**：
```
CMD 指令：
├─ 作用：设置容器启动时的默认命令
├─ 格式：CMD ["命令", "参数1", "参数2"]
├─ 时机：运行时执行（不是构建时）
└─ 特性：可以被 docker run 的命令覆盖

CMD ["python", "data_analysis.py"]
└─ 容器启动时默认运行：python data_analysis.py

对比：
├─ RUN: 构建时执行，结果保存在镜像层
└─ CMD: 运行时执行，只是设置默认命令
```

#### 12. WORKDIR vs VOLUME 的区别
**问题**：WORKDIR 和 VOLUME 的区别？VOLUME 只是告知作用吗？

**答案**：
```
WORKDIR /app
├─ 有实际作用 ✅
├─ 创建目录并设置工作目录
├─ 后续命令都在此目录执行
└─ 可以使用相对路径

VOLUME ["/data"]
├─ 只是声明/告知作用 ⚠️
├─ 不实际创建目录或挂载
├─ 运行时才会创建卷/挂载
└─ 必须使用绝对路径

路径简化：
├─ WORKDIR /app 后
├─ COPY script.py .  ✅（相对路径）
├─ RUN python script.py  ✅（相对于 /app）
└─ 避免每次写 /app/ 前缀
```

#### 13. 匿名卷详解
**问题**：匿名卷是什么？有什么用？对应主机的地址是什么？

**答案**：
```
匿名卷（Anonymous Volume）：
┌─────────────────────────────────────┐
│  创建方式：                           │
│  ├─ Dockerfile 中：VOLUME ["/data"]  │
│  └─ 运行时不指定 -v                  │
│         │                            │
│         ▼                            │
│  Docker 自动创建匿名卷               │
│  └─ 卷名：随机 64 字符 ID             │
│         │                            │
│         ▼                            │
│  存储位置：                           │
│  Linux: /var/lib/docker/volumes/    │
│        <卷ID>/_data                  │
│                                      │
│  macOS/Windows: 虚拟机内部相同路径    │
│  └─ 需要通过 Docker 命令访问          │
└─────────────────────────────────────┘

作用：
├─ ✅ 数据持久化（即使容器删除）
├─ ✅ 性能优化（避免写入容器层）
└─ ⚠️ 问题：卷名随机，难以管理

对比：
├─ 匿名卷：随机名称，难以管理
├─ 命名卷：有名称，容易管理 ✅
└─ 绑定挂载：直接使用主机路径 ✅
```

#### 14. Mount 发生在哪个阶段？
**问题**：mount 是发生在 build image 阶段还是 run container 阶段？

**答案**：
```
Mount（挂载）阶段：

构建阶段（docker build）：
┌─────────────────────────────────────┐
│  VOLUME ["/data"]                   │
│  └─ 只是声明，记录在元数据中          │
│  └─ ❌ 没有实际挂载发生               │
└─────────────────────────────────────┘

运行阶段（docker run）：
┌─────────────────────────────────────┐
│  docker run -v /host:/data          │
│  └─ ✅ 实际挂载发生在这里             │
│  └─ 读取 -v 参数，创建挂载点          │
└─────────────────────────────────────┘

关键点：
├─ VOLUME 指令：只是声明（构建时）
├─ -v 参数：实际挂载（运行时）
└─ 挂载发生在运行容器阶段 ✅
```

#### 15. EXPOSE 端口声明
**问题**：端口声明不太熟悉，EXPOSE 的作用是什么？

**答案**：
```
EXPOSE 指令：

EXPOSE 8000
├─ 作用：只是文档说明 ⚠️
├─ 告诉使用者：容器使用 8000 端口
├─ ❌ 不会实际映射端口
└─ ✅ 需要用 -p 参数实际映射

对比：
┌─────────────────────────────────────┐
│  Dockerfile: EXPOSE 8000            │
│  └─ 只是说明                         │
│         │                            │
│         ▼                            │
│  docker run -p 8000:8000            │
│  └─ 实际端口映射 ✅                   │
│         │                            │
│         ▼                            │
│  可以访问 http://localhost:8000      │
└─────────────────────────────────────┘

格式：-p 主机端口:容器端口
示例：-p 8080:8000
      └─ 主机 8080 → 容器 8000
```

### 📝 重要知识点

```
Dockerfile 常用指令：
┌─────────────────────────────────────┐
│  FROM python:3.9                    │
│  └─ 基础镜像（必需，第一行）          │
│         │                            │
│         ▼                            │
│  ENV PYTHONUNBUFFERED=1             │
│  └─ 设置环境变量                     │
│         │                            │
│         ▼                            │
│  WORKDIR /app                       │
│  └─ 设置工作目录（可以使用相对路径）   │
│         │                            │
│         ▼                            │
│  COPY requirements.txt .            │
│  RUN pip install -r requirements.txt│
│  └─ 复制依赖文件并安装                │
│         │                            │
│         ▼                            │
│  COPY . .                           │
│  └─ 复制代码                         │
│         │                            │
│         ▼                            │
│  EXPOSE 8000                        │
│  └─ 声明端口（只是说明）              │
│         │                            │
│         ▼                            │
│  CMD ["python", "app.py"]           │
│  └─ 默认启动命令（运行时执行）         │
└─────────────────────────────────────┘

最佳实践：
├─ ✅ 合并 RUN 命令，减少层数
├─ ✅ 先复制依赖文件，利用缓存
├─ ✅ 使用 .dockerignore 排除文件
└─ ✅ 使用轻量级基础镜像（slim/alpine）
```

---

## 🎼 第5章：Docker Compose

### 🎯 核心问题

（本章没有特别的问题，主要是学习和使用）

### 📝 重要知识点

```
Docker Compose 工作流程：
┌─────────────────────────────────────┐
│  编写 docker-compose.yml            │
│  └─ 定义所有服务配置                 │
│         │                            │
│         ▼                            │
│  docker-compose up -d               │
│  └─ 启动所有服务（一条命令）          │
│         │                            │
│         ├─ 创建网络                  │
│         ├─ 创建数据卷                │
│         ├─ 构建镜像（如果需要）       │
│         └─ 启动所有容器               │
│         │                            │
│         ▼                            │
│  docker-compose ps                  │
│  └─ 查看服务状态                     │
│         │                            │
│         ▼                            │
│  docker-compose logs -f             │
│  └─ 查看日志                         │
│         │                            │
│         ▼                            │
│  docker-compose down                │
│  └─ 停止并删除所有服务                │
└─────────────────────────────────────┘
```

```
docker-compose.yml 结构：
┌─────────────────────────────────────┐
│  version: '3.8'                     │
│         │                            │
│         ▼                            │
│  services:                           │
│    service1:                        │
│      image: ...                     │
│      ports: ...                     │
│      volumes: ...                   │
│      environment: ...               │
│      depends_on: ...                │
│         │                            │
│         ▼                            │
│  networks:                           │
│    my-network:                      │
│         │                            │
│         ▼                            │
│  volumes:                            │
│    my-volume:                       │
└─────────────────────────────────────┘
```

**常用命令**：
```
启动服务：    docker-compose up -d
停止服务：    docker-compose down
查看状态：    docker-compose ps
查看日志：    docker-compose logs -f
构建镜像：    docker-compose build
重启服务：    docker-compose restart
执行命令：    docker-compose exec SERVICE CMD
```

**Data Engineer 典型配置**：
```yaml
services:
  postgres:
    image: postgres:14
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret
  
  jupyter:
    image: jupyter/scipy-notebook
    ports:
      - "8888:8888"
    volumes:
      - ./notebooks:/home/jovyan/work
    depends_on:
      - postgres
```

---

## 📋 第6章：Docker 常用命令

### 📝 重要知识点

```
命令分类：

镜像命令：
├─ docker pull IMAGE          # 拉取镜像
├─ docker images              # 查看镜像
├─ docker search NAME         # 搜索镜像
├─ docker build -t NAME .     # 构建镜像
├─ docker rmi IMAGE           # 删除镜像
├─ docker save -o FILE IMAGE  # 导出镜像
└─ docker load -i FILE        # 导入镜像

容器命令：
├─ docker run [OPTIONS] IMAGE     # 创建并运行
├─ docker ps                      # 查看容器
├─ docker stop CONTAINER          # 停止容器
├─ docker start CONTAINER         # 启动容器
├─ docker rm CONTAINER            # 删除容器
├─ docker exec -it CONTAINER CMD  # 进入容器
├─ docker logs CONTAINER          # 查看日志
└─ docker inspect CONTAINER       # 查看详细信息

Docker Compose：
├─ docker-compose up -d           # 启动所有服务
├─ docker-compose down            # 停止所有服务
├─ docker-compose ps              # 查看状态
├─ docker-compose logs -f         # 查看日志
└─ docker-compose exec SERVICE    # 执行命令
```

---

## 🎯 核心概念总结

### 镜像 vs 容器

```
镜像（Image）         容器（Container）
───────────          ───────────────
只读模板             运行实例
静态的               动态的
类（Class）          对象（Object）
一个镜像             多个容器
```

### 数据存储方式

```
数据存储选择：
┌─────────────────────────────────────┐
│  方式1：容器层                        │
│  └─ 临时数据，容器删除后丢失          │
│         │                            │
│         ▼                            │
│  方式2：匿名卷                        │
│  └─ 持久化，但难以管理                │
│         │                            │
│         ▼                            │
│  方式3：命名卷                        │
│  └─ 持久化，容易管理 ✅（生产环境）    │
│         │                            │
│         ▼                            │
│  方式4：绑定挂载                      │
│  └─ 直接使用主机路径 ✅（开发环境）    │
└─────────────────────────────────────┘
```

### 完整工作流程

```
开发流程：
┌─────────────────────────────────────┐
│  1. 编写 Dockerfile                 │
│         │                            │
│         ▼                            │
│  2. docker build -t IMAGE .         │
│         │                            │
│         ▼                            │
│  3. docker run IMAGE                │
│     或                              │
│     docker-compose up -d            │
│         │                            │
│         ▼                            │
│  4. 使用和管理容器                    │
│         │                            │
│         ▼                            │
│  5. docker push IMAGE（可选）        │
└─────────────────────────────────────┘
```

---

## 💡 快速参考

### 最常用命令

```bash
# 镜像
docker pull IMAGE
docker images
docker build -t NAME .

# 容器
docker run -d -p PORT:PORT IMAGE
docker ps
docker stop CONTAINER
docker exec -it CONTAINER bash

# Compose
docker-compose up -d
docker-compose down
docker-compose ps
docker-compose logs -f
```

### Data Engineer 常用场景

```bash
# 运行数据分析环境
docker-compose up -d

# 运行数据处理脚本
docker run --rm -v $(pwd)/data:/data python:3.9 python script.py

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f postgres
```

---

## 📌 学习要点记忆

1. **镜像 = 模板，容器 = 实例**
2. **Dockerfile = 构建镜像的脚本**
3. **Docker Compose = 管理多容器应用**
4. **数据持久化 = 使用数据卷（volumes）**
5. **端口访问 = 端口映射（-p 参数）**
6. **开发环境 = 绑定挂载（-v 主机路径:容器路径）**
7. **Mount = 运行时发生，不是构建时**
8. **EXPOSE = 只是说明，需要用 -p 实际映射**
9. **WORKDIR = 有实际作用，VOLUME = 只是声明**
10. **容器停止后可以再启动，删除后无法启动**

---

*最后更新：学习完成时*
*学习方式：互动式问答 + 实践操作*

