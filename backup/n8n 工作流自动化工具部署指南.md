# n8n 工作流自动化工具部署指南

## 工具简介

😀 n8n 是一个基于 Node.js 开发的开源工作流自动化工具

### **应用场景**

- **业务流程自动化**：连接现有业务工具与应用程序，简化数据输入、客户入职、发票开具等重复性任务。
- **跨平台集成**：无缝集成不同系统，实现基于云和本地软件之间的数据流与流程自动化。
- **数据驱动的见解**：从多个来源收集、汇总与分析数据，转化为可操作的见解，助力决策制定。
- **AI 集成**：通过集成 LangChain 等，将自然语言处理和生成模型等高级人工智能功能融入自动化工作流。

## 部署步骤

### 1. 安装Docker

安装方式网上很多，本文使用[LinuxMirrors](https://github.com/SuperManito/LinuxMirrors)的脚本安装：

```bash
bash <(curl -sSL https://linuxmirrors.cn/main.sh)
bash <(curl -sSL https://linuxmirrors.cn/docker.sh)
```

### 2. 创建docker-compose.yml及.env文件

在`n8n-compose`文件夹中创建以下文件：

#### docker-compose.yml

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_SECURE_COOKIE=${N8N_SECURE_COOKIE}
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=${N8N_PORT}
      - N8N_PROTOCOL=${N8N_PROTOCOL}
      - NODE_ENV=${NODE_ENV}
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
      - N8N_BASIC_AUTH_ACTIVE=${N8N_BASIC_AUTH_ACTIVE}
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
      - N8N_LANGUAGE=zh-CN
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
      - DB_POSTGRESDB_SSL=${POSTGRES_SSL}
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - n8n-network
    depends_on:
      - postgres

  postgres:
    image: postgres:15-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_INITDB_ARGS=--encoding=UTF-8
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - n8n-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  n8n-network:

volumes:
  n8n_data:
  postgres_data:  
```

#### .env 配置文件

```yaml
# n8n基础配置
N8N_SECURE_COOKIE=false
#修改为自己的ip
N8N_HOST=localhost 
N8N_PORT=5678
N8N_PROTOCOL=http
NODE_ENV=production
GENERIC_TIMEZONE=Asia/Shanghai

# 基础认证配置（默认禁用，启用请设置为true并设置用户名和密码）
N8N_BASIC_AUTH_ACTIVE=false
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your_password_here

# 执行模式配置（可选regular或queue）
EXECUTIONS_MODE=regular

# 数据库配置
POSTGRES_DB=n8n
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8n
POSTGRES_SSL=false

# 强制修正配置文件权限
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true  
```

### 3. 启动服务

进入n8n-compose文件夹，执行以下命令：

```bash
docker-compose up -d
```

启动完成后，访问 `http://your_ip:5678` 即可进入n8n管理界面。