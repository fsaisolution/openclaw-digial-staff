# OpenClaw 数字员工部署指南

## 什么是 OpenClaw？

OpenClaw 是一款开源、可自托管的 AI 智能体平台（数字员工），前身为 ClawdBot 与 Moltbot。它具备主动任务执行能力，区别于单纯的对话式 AI，核心价值在于"执行"而非"聊天"。

### 主要特性

- **多平台支持**：WhatsApp、Telegram、Slack、Discord、Google Chat、Signal、iMessage、Microsoft Teams、Matrix 等
- **本地部署**：数据本地存储，隐私性与安全性有保障
- **7×24 小时自动化**：无需复杂编程，最低 5 分钟即可完成部署
- **语音支持**：支持 Voice Wake + Talk Mode 语音交互
- **Canvas 工作区**：支持 Agent 驱动的可视化工作空间

---

## 部署方式

OpenClaw 支持多种部署方式，本指南介绍两种主流方案：

1. **npm/pnpm 全局安装（推荐）**
2. **Docker 容器化部署**

---

## 方式一：npm/pnpm 安装（推荐）

### 环境要求

- Node.js ≥ 22
- npm、pnpm 或 bun

### 安装步骤

#### 1. 安装 OpenClaw

```bash
# 使用 npm 安装
npm install -g openclaw@latest

# 或使用 pnpm 安装
pnpm add -g openclaw@latest
```

#### 2. 运行初始化向导

```bash
openclaw onboard --install-daemon
```

向导会引导你完成以下配置：
- Gateway 设置
- 工作区配置
- 频道连接（WhatsApp/Telegram/Slack 等）
- 技能安装

#### 3. 启动 Gateway

```bash
openclaw gateway --port 18789 --verbose
```

#### 4. 发送消息测试

```bash
openclaw message send --to +1234567890 --message "Hello from OpenClaw"
```

#### 5. 与助手对话

```bash
openclaw agent --message "帮我整理桌面文件" --thinking high
```

---

## 方式二：Docker 部署

### 环境要求

- Docker
- Docker Compose

### 安装步骤

#### 1. 克隆官方仓库

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

#### 2. 使用 Docker Compose 启动

官方仓库提供了 `docker-setup.sh` 脚本：

```bash
./docker-setup.sh
```

或手动启动：

```bash
docker-compose up -d
```

#### 3. 配置环境变量

创建 `.env` 文件，配置必要的环境变量：

```bash
# 必需：模型配置
ANTHROPIC_API_KEY=your_api_key_here

# 可选：Gateway 端口配置
GATEWAY_PORT=18789

# 可选：频道配置
# Telegram
TELEGRAM_BOT_TOKEN=your_telegram_token

# Slack
SLACK_BOT_TOKEN=your_slack_bot_token
SLACK_APP_TOKEN=your_slack_app_token

# Discord
DISCORD_BOT_TOKEN=your_discord_token
```

#### 4. 查看日志

```bash
docker-compose logs -f
```

#### 5. 停止服务

```bash
docker-compose down
```

---

## 配置说明

### 配置文件位置

- 配置目录：`~/.openclaw/`
- 工作区：`~/.openclaw/workspace`（可通过 `agents.defaults.workspace` 配置）
- 凭证存储：`~/.openclaw/credentials/`

### 最小配置示例

创建 `~/.openclaw/openclaw.json`：

```json
{
  "agent": {
    "model": "anthropic/claude-opus-4-6"
  }
}
```

### 频道配置

#### Telegram

```json
{
  "channels": {
    "telegram": {
      "botToken": "123456:ABCDEF"
    }
  }
}
```

#### Discord

```json
{
  "channels": {
    "discord": {
      "token": "1234abcd"
    }
  }
}
```

#### Slack

```json
{
  "channels": {
    "slack": {
      "botToken": "xoxb-...",
      "appToken": "xapp-..."
    }
  }
}
```

### 浏览器控制（可选）

```json
{
  "browser": {
    "enabled": true,
    "color": "#FF4500"
  }
}
```

---

## 安全配置

### DM 访问控制

OpenClaw 默认安全策略：
- **配对模式（pairing）**：未知发送者需配对码
- **开放模式（open）**：需明确配置 allowFrom

推荐配置：

```bash
# 运行安全检查
openclaw doctor
```

### 沙盒模式

对于群组/频道会话，可启用 Docker 沙盒隔离：

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main"
      }
    }
  }
}
```

---

## 远程访问

### Tailscale Serve/Funnel

OpenClaw 支持自动配置 Tailscale 暴露服务：

```json
{
  "gateway": {
    "tailscale": {
      "mode": "serve"  // 或 "funnel"（需设置密码）
    }
  }
}
```

- `serve`：仅 Tailscale 网络内可访问
- `funnel`：公开 HTTPS 访问（需设置密码认证）

---

## 常用命令

| 命令 | 说明 |
|------|------|
| `openclaw gateway` | 启动 Gateway |
| `openclaw agent` | 与 AI 助手对话 |
| `openclaw message send` | 发送消息 |
| `openclaw channels login` | 登录频道 |
| `openclaw doctor` | 诊断配置问题 |
| `openclaw update` | 更新 OpenClaw |

---

## 常见问题

### Q1: API Key 如何获取？

访问以下平台获取 API Key：
- [Anthropic Console](https://console.anthropic.com/)
- [OpenAI Platform](https://platform.openai.com/)
- [阿里云百炼](https://dashscope.console.aliyun.com/)

### Q2: 如何查看日志？

```bash
# Docker 方式
docker-compose logs -f

# 本地安装
tail -f ~/.openclaw/logs/gateway.log
```

### Q3: 如何进入容器调试？

```bash
docker exec -it openclaw bash
```

### Q4: 内存不足怎么办？

建议服务器内存至少 2GB 以上。如果使用 Docker 部署，确保分配足够内存。

---

## 升级更新

```bash
# 更新到最新版本
openclaw update

# 或指定版本
npm install -g openclaw@latest
```

---

## 相关资源

- 官方网站：https://openclaw.ai
- GitHub 仓库：https://github.com/openclaw/openclaw
- 官方文档：https://docs.openclaw.ai
- Discord 社区：https://discord.gg/openclaw
