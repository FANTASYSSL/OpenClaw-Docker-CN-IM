# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 重要提示

- **Docker 命令格式**：使用新版 `docker compose`（中间没有横杠），不要使用旧版 `docker-compose`

## 项目概述

OpenClaw 中国 IM 插件整合版 Docker 镜像项目，预装飞书、钉钉、QQ机器人、企业微信等中国主流 IM 平台支持。

## 常用命令

### 本地构建与运行

```bash
# 构建镜像
docker build -t openclaw:local .

# 使用 docker compose 启动（推荐）
docker compose up -d

# 查看日志
docker compose logs -f

# 停止服务
docker compose down
```

### 发布镜像

```bash
# 推送到 Docker Hub（交互式）
./push-to-dockerhub.sh

# 推送到 GitHub Container Registry（交互式）
./push-to-ghcr.sh
```

版本号从 `version.txt` 读取，修改此文件并推送到 main 分支会触发 GitHub Actions 自动构建。

## 项目架构

```
├── Dockerfile           # 镜像构建文件，基于 node:22-slim
├── docker-compose.yml   # 服务编排配置
├── init.sh              # 容器入口脚本，根据环境变量生成 openclaw.json
├── .env.example         # 环境变量模板
├── openclaw.json.example # OpenClaw 配置文件示例
└── version.txt          # 版本号，用于镜像标签
```

### 配置生成流程

1. 容器启动时执行 `init.sh`
2. 检查 `/home/node/.openclaw/openclaw.json` 是否存在
3. 如不存在，从环境变量动态生成配置文件
4. 根据提供的 IM 平台凭证自动启用对应通道和插件
5. 以 `node` 用户身份启动 `openclaw gateway --verbose`

### 环境变量分类

- **模型配置**: `MODEL_ID`, `BASE_URL`, `API_KEY`, `API_PROTOCOL`, `CONTEXT_WINDOW`, `MAX_TOKENS`
- **IM 平台**: 各平台凭证（飞书、钉钉、QQ、企业微信）均为可选，留空则不启用
- **Gateway**: `OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PORT`, `OPENCLAW_GATEWAY_BIND`

### API 协议

- `openai-completions`: OpenAI 兼容协议，Base URL 需要 `/v1` 后缀
- `anthropic-messages`: Claude 协议，Base URL 不需要 `/v1` 后缀

### 预装插件

镜像中通过 `openclaw plugins install` 预装：
- 钉钉: `https://github.com/soimy/clawdbot-channel-dingtalk.git`
- QQ机器人: 从 `/tmp/qqbot` 本地安装
- 飞书和企业微信使用 OpenClaw 内置支持

### 数据持久化

- 宿主机当前目录下的 `./.openclaw` 挂载到容器的 `/home/node/.openclaw`
- `extensions` 目录使用匿名卷，保留镜像中预装的插件
