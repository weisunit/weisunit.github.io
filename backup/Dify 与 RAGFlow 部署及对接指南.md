# Dify 与 RAGFlow 部署及对接指南

## 1. 产品概述

😀 **RAGFlow** 是一款基于深度文档理解构建的开源 RAG（Retrieval-Augmented Generation）引擎。RAGFlow 可以为各种规模的企业及个人提供一套精简的 RAG 工作流程，结合大语言模型（LLM）针对用户各类不同的复杂格式数据提供可靠的问答以及有理有据的引用。

**Dify** 是一个开源的 LLM 应用开发平台。其直观的界面结合了 AI 工作流、RAG 管道、Agent、模型管理、可观测性功能等，让您可以快速从原型到生产。

**产品优势对比**:

- Dify 主要优势在强大的工作流编排和 Agent 能力构建复杂应用
- RAGFlow 主要优势在文件精细解析能力强，在处理 PDF、扫描件、表格等复杂文档方面表现出色
- 两向结合，优势互补

## 2. Dify 部署

### 系统要求

- CPU >= 2 Core
- RAM >= 4 GiB

### 启动步骤

1. 进入 Dify 源代码的 Docker 目录

```bash
cd dify/docker
```

2. 复制环境配置文件

```bash
cp .env.example .env
```

3. 启动 Docker 容器

```bash
docker compose -p dify_docker up -d
```

### 访问 Dify

- 本地环境: `http://localhost/install`
- 服务器环境: `http://your_server_ip/install`

Dify 主页面:

- 本地环境: `http://localhost`
- 服务器环境: `http://your_server_ip`

### 更新 Dify

```bash
cd dify/docker
docker compose down
git pull origin main
docker compose pull
docker compose -p dify_docker up -d
```

## 3. RAGFlow 部署

### 系统要求

- CPU >= 4 核
- RAM >= 16 GB
- Disk >= 50 GB
- Docker >= 24.0.0 & Docker Compose >= v2.26.1

> 如果没有安装 Docker（Windows、Mac，或者 Linux）, 可以参考 [Install Docker Engine](https://docs.docker.com/engine/install/) 自行安装。

### 启动服务器

1. 确保 `vm.max_map_count` 不小于 262144：

```bash
# 检查当前值
sysctl vm.max_map_count

# 临时设置
sudo sysctl -w vm.max_map_count=262144

# 永久设置（编辑 /etc/sysctl.conf 文件）
vm.max_map_count=262144
```

2. 克隆仓库：

```bash
git clone https://github.com/infiniflow/ragflow.git
```

3. 修改配置文件：
   - 编辑 `ragflow/docker/.evn`:

```env
# The type of doc engine to use.
# Available options:
# - `elasticsearch` (default)
# - `infinity` (https://github.com/infiniflow/infinity)
DOC_ENGINE=${DOC_ENGINE:-infinity}

# The RAGFlow Docker image to download.
# Defaults to the v0.17.2-slim edition, which is the RAGFlow Docker image without embedding models.
RAGFLOW_IMAGE=infiniflow/ragflow:v0.17.2
```

- 编辑 `ragflow/docker/docker-compose.yml` 修改端口

  ![QQ20250428-152121.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/bb771e25-d9f9-4c1a-85fe-e1ec02aca409.png)

4. 启动服务器：

```bash
cd ragflow/docker
# Use CPU for embedding and DeepDoc tasks:
docker compose -f docker-compose.yml up -d
```

5. 浏览器访问 `http://IP:8080` 验证

## 4. 知识库对接

1. 在 Dify 中配置 RAGFlow 的知识库时，需要在 RAGFlow 的基础 Base url 后增加 `api/v1/dify`，这是 Dify 特定的 API 路径。
2. 在 Dify 中点击"知识库-外部知识库API"

   ![QQ20250428-162942.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/10b214c3-f9b1-4ba2-91be-7e2788492deb.png)
3. 完成 Dify 和 RAGFlow 的 API 连接之后:

   - 点击"连接外部知识库"按钮
   - 输入外部知识库 ID（从 RAGFlow 知识库页面的浏览器地址后缀获取）

     ![QQ20250428-163259.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/9f85e104-67c6-47db-8014-661b9aa596f0.png)

     ![QQ20250428-163535.png](https://cdn.jsdelivr.net/gh/weisunit/note-gen-image-sync@main/d8e234b4-c420-4154-96c0-bcb6e30a108d.png)

> Dify 和 RAGFlow 还有许多强大能力，后续可以进一步探索。

## 参考链接

1. [Dify 官方仓库](https://github.com/langgenius/dify)
2. [RAGFlow 官方仓库](https://github.com/infiniflow/ragflow)
3. [Docker 安装文档](https://docs.docker.com/engine/install/)