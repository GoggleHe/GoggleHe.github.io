---
layout: post
title: Hermes Agent - 开源AI Agent框架
categories: [AI, Linux]
description: Hermes Agent 开源AI Agent框架介绍与实践
keywords: Hermes Agent, AI Agent, Nous Research, OpenClaw
---

## Hermes Agent

Hermes Agent 是由 Nous Research 开发的开源 AI Agent 框架，与 Claude Code、OpenAI Codex 同类。支持多平台消息网关、持久化记忆、技能系统、定时任务等完整 Agent 能力。

仓库：https://github.com/NousResearch/hermes-agent

## 核心特性

- **多模型支持**：20+ 提供商（OpenRouter、Anthropic、DeepSeek、OpenAI、本地模型等），可随时切换
- **多平台网关**：Telegram、Discord、QQ Bot、Slack、WhatsApp 等 20+ 消息平台
- **技能系统**：可复用的流程知识库，Agent 自我改进
- **持久化记忆**：跨会话记忆，支持内置/Honcho/Mem0 等多种后端
- **定时任务**：内建 cron 调度器
- **子 Agent 委派**：并行/串行任务分发
- **插件体系**：可扩展的插件系统
- **TUI + Web Dashboard**：终端界面和网页管理面板

## 安装

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

## 对比同类工具

| 功能 | Hermes Agent | OpenClaw |
|------|:-----------:|:--------:|
| 语言 | Python | TypeScript |
| 消息平台 | 20+ | 19+ |
| 技能系统 | ✅ | ✅ |
| 定时任务 | ✅ | ✅ |
| 子Agent | ✅ delegate_task | ✅ subagent（任务注册表） |
| 开源 | ✅ MIT | ✅ MIT |

## 常用命令

```bash
hermes                        # 启动聊天
hermes chat -q "query"        # 单次查询
hermes model                  # 切换模型/提供商
hermes gateway run            # 启动消息网关
hermes dashboard              # Web管理面板
hermes cron list              # 查看定时任务
hermes doctor                 # 健康检查
hermes update                 # 更新到最新版
```

## 配置文件

主配置：`~/.hermes/config.yaml`
密钥：`~/.hermes/.env`
会话数据库：`~/.hermes/state.db`（SQLite，FTS5全文检索）
技能目录：`~/.hermes/skills/`

## 体验总结

Hermes Agent 的自我进化是其区别于同类工具的核心设计。Agent 在完成复杂任务后会自动将流程沉淀为 Skill，下次遇到同类问题直接复用，不需要重复调试。配合 Curator 后台维护机制，长期使用的边际成本越来越低。
