# New API 部署到 Zeabur 指南

## 📋 部署前准备

### 1. 注册 Zeabur 账号
访问 [Zeabur](https://zeabur.com) 并注册账号（支持 GitHub 登录）

### 2. 准备 GitHub 仓库
确保你的项目已推送到 GitHub 仓库

## 🚀 部署步骤

### 方式一：通过 Zeabur Dashboard 部署（推荐）

#### 步骤 1：创建项目
1. 登录 [Zeabur Dashboard](https://dash.zeabur.com)
2. 点击 "New Project" 创建新项目
3. 选择部署区域（推荐选择离你最近的区域）

#### 步骤 2：添加服务
1. 在项目中点击 "Add Service"
2. 选择 "Git" 作为服务来源
3. 授权并选择你的 GitHub 仓库
4. Zeabur 会自动检测到 Dockerfile 并开始构建

#### 步骤 3：配置环境变量
在服务设置中添加以下环境变量：

**必需的环境变量：**
```bash
# 端口配置（Zeabur 会自动注入 PORT，但建议显式设置）
PORT=3000

# 时区设置
TZ=Asia/Shanghai

# 会话密钥（多机部署必须设置，建议使用随机字符串）
SESSION_SECRET=your_random_secret_string_here

# 数据库配置（选择以下一种）
# 选项1：使用 SQLite（需要持久化存储）
# 不设置 SQL_DSN 将自动使用 SQLite

# 选项2：使用 PostgreSQL（推荐）
# SQL_DSN=postgresql://username:password@host:5432/database

# 选项3：使用 MySQL
# SQL_DSN=username:password@tcp(host:3306)/database
```

**可选的环境变量：**
```bash
# Redis 缓存（推荐配置，提升性能）
REDIS_CONN_STRING=redis://your-redis-host:6379

# 加密密钥（使用 Redis 时必须设置）
CRYPTO_SECRET=your_crypto_secret_here

# 错误日志
ERROR_LOG_ENABLED=true

# 批量更新
BATCH_UPDATE_ENABLED=true

# 流式超时时间（秒）
STREAMING_TIMEOUT=300

# 生成默认令牌
GENERATE_DEFAULT_TOKEN=false

# Dify 调试
DIFY_DEBUG=true

# 图片 Token 统计
GET_MEDIA_TOKEN=true
GET_MEDIA_TOKEN_NOT_STREAM=true

# 异步任务更新
UPDATE_TASK=true

# Gemini 最大图片数量
GEMINI_VISION_MAX_IMAGE_NUM=16

# 最大文件下载大小（MB）
MAX_FILE_DOWNLOAD_MB=20

# Azure 默认 API 版本
AZURE_DEFAULT_API_VERSION=2025-04-01-preview

# 通知限制
NOTIFICATION_LIMIT_DURATION_MINUTE=10
NOTIFY_LIMIT_COUNT=2

# 按次计费的异步任务模型（逗号分隔）
TASK_PRICE_PATCH=sora-2-all,sora-2-pro-all
```

#### 步骤 4：配置持久化存储（使用 SQLite 时必需）
1. 在服务设置中找到 "Volumes" 选项
2. 添加持久化卷：
   - Mount Path: `/data`
   - 这将确保 SQLite 数据库文件不会在重启后丢失

#### 步骤 5：配置域名（可选）
1. 在服务设置中找到 "Domains" 选项
2. 可以使用 Zeabur 提供的免费域名，或绑定自定义域名
3. Zeabur 会自动配置 HTTPS 证书

#### 步骤 6：部署
1. 保存所有配置后，Zeabur 会自动开始构建和部署
2. 等待构建完成（首次构建可能需要 5-10 分钟）
3. 部署成功后，访问分配的域名即可使用

### 方式二：使用 Zeabur CLI 部署

#### 步骤 1：安装 Zeabur CLI
```bash
# macOS/Linux
curl -fsSL https://cli.zeabur.com/install.sh | bash

# Windows (PowerShell)
iwr https://cli.zeabur.com/install.ps1 -useb | iex
```

#### 步骤 2：登录
```bash
zeabur auth login
```

#### 步骤 3：初始化项目
```bash
# 在项目根目录执行
zeabur init
```

#### 步骤 4：部署
```bash
zeabur deploy
```

## 🗄️ 数据库配置建议

### 使用 Zeabur 的 PostgreSQL 服务（推荐）

1. 在 Zeabur 项目中点击 "Add Service"
2. 选择 "Marketplace" → "PostgreSQL"
3. Zeabur 会自动创建数据库并提供连接信息
4. 在 New API 服务的环境变量中设置：
   ```bash
   SQL_DSN=postgresql://${POSTGRES_USERNAME}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DATABASE}
   ```
   注意：Zeabur 会自动注入 PostgreSQL 的环境变量，你可以直接引用

### 使用 Zeabur 的 Redis 服务（推荐）

1. 在 Zeabur 项目中点击 "Add Service"
2. 选择 "Marketplace" → "Redis"
3. 在 New API 服务的环境变量中设置：
   ```bash
   REDIS_CONN_STRING=redis://${REDIS_HOST}:${REDIS_PORT}
   CRYPTO_SECRET=your_crypto_secret_here
   ```

## 📊 监控和日志

### 查看日志
1. 在 Zeabur Dashboard 中打开你的服务
2. 点击 "Logs" 标签查看实时日志
3. 支持日志搜索和过滤

### 监控指标
Zeabur 提供以下监控指标：
- CPU 使用率
- 内存使用率
- 网络流量
- 请求响应时间

## 🔧 常见问题

### 1. 构建失败
- 检查 Dockerfile 是否正确
- 查看构建日志中的错误信息
- 确保所有依赖都已正确配置

### 2. 服务无法访问
- 检查端口配置是否正确（默认 3000）
- 确认服务状态为 "Running"
- 检查域名配置是否正确

### 3. 数据丢失
- 如果使用 SQLite，确保已配置持久化存储卷
- 建议使用 PostgreSQL 作为生产环境数据库

### 4. 性能问题
- 配置 Redis 缓存以提升性能
- 在 Zeabur Dashboard 中升级服务规格
- 启用 `BATCH_UPDATE_ENABLED` 批量更新功能

### 5. 多机部署
- 必须设置 `SESSION_SECRET` 环境变量
- 如果使用 Redis，必须设置 `CRYPTO_SECRET`
- 建议使用外部数据库（PostgreSQL/MySQL）而非 SQLite

## 🔄 更新部署

### 自动部署
Zeabur 支持 Git 自动部署：
1. 在服务设置中启用 "Auto Deploy"
2. 每次推送到 GitHub 仓库时，Zeabur 会自动重新构建和部署

### 手动部署
1. 在 Zeabur Dashboard 中打开服务
2. 点击 "Redeploy" 按钮
3. 选择要部署的分支或提交

## 💰 费用说明

Zeabur 提供：
- **Developer Plan**：免费额度，适合开发和测试
- **Pro Plan**：按使用量计费，适合生产环境

详细定价请访问：https://zeabur.com/pricing

## 📚 相关资源

- [Zeabur 官方文档](https://zeabur.com/docs)
- [New API 官方文档](https://docs.newapi.pro/)
- [New API GitHub 仓库](https://github.com/Calcium-Ion/new-api)

## 🆘 获取帮助

如遇到问题：
1. 查看 [Zeabur 文档](https://zeabur.com/docs)
2. 访问 [Zeabur Discord 社区](https://discord.gg/zeabur)
3. 查看 [New API 帮助文档](https://docs.newapi.pro/support)
