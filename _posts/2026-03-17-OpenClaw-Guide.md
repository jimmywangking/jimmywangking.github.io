---
layout: post
title:  "OpenClaw 使用手册：把 AI 助手装进你的口袋"
date:   2026-03-17 20:30:00 +0800
categories: AI Tools
tags: [OpenClaw, AI, 自托管, 聊天机器人, 效率工具]
image: /assets/images/openclaw-cover.png
---

> "EXFOLIATE! EXFOLIATE!" —— 一只太空龙虾，大概是这么说的

## 什么是 OpenClaw？

[OpenClaw](https://docs.openclaw.ai) 是一个**自托管的 AI 网关**，让你可以通过 WhatsApp、Telegram、Discord、iMessage 等任意聊天软件，随时随地调用 AI 助手。

你在自己的电脑或服务器上跑一个 Gateway 进程，它就成了你的消息 App 和 AI 之间的桥梁。

![OpenClaw 架构示意](https://mintcdn.com/clawdhub/FaXdIfo7gPK_jSWb/whatsapp-openclaw.jpg)

---

## 它和普通 AI 助手有什么不同？

| 对比项 | 普通 AI 助手（如豆包、Copilot） | OpenClaw |
|---|---|---|
| 数据归属 | 存在厂商服务器 | 完全在你自己机器上 |
| 渠道 | 固定 App | WhatsApp/Telegram/Discord 等多渠道 |
| 可扩展性 | 有限 | 开源，可装 Skills/Plugins |
| 控制权 | 厂商说了算 | 你说了算 |
| 费用 | 订阅制 | 自备 API Key，按量付费 |

**一句话：** 你自己的 AI，跑在你自己的机器上，通过你最常用的聊天软件来用。

---

## 核心能力

- 🌐 **多渠道接入**：一个 Gateway，同时服务 WhatsApp、Telegram、Discord、iMessage
- 🔌 **插件扩展**：通过 Skills 扩展能力（天气、日历、文件整理、小红书等）
- 🤖 **多 Agent 路由**：不同场景、不同用户可以隔离会话
- 📱 **移动端节点**：配对 iOS/Android，支持摄像头、语音、Canvas
- 🧠 **长期记忆**：跨会话记住你的偏好和上下文
- 🖥️ **Web 控制台**：浏览器打开 `http://127.0.0.1:18789` 管理一切

---

## 快速安装

### 环境要求

- Node.js 22 LTS（`22.16+`）或 Node 24（推荐）
- 一个 AI 模型的 API Key（OpenAI、Anthropic、Gemini 等均可）
- 5 分钟时间

### 三步启动

**第一步：安装**

```bash
npm install -g openclaw@latest
```

**第二步：初始化并安装服务**

```bash
openclaw onboard --install-daemon
```

这一步会引导你配置 API Key、选择模型、设置渠道。

**第三步：连接渠道并启动**

```bash
openclaw channels login
openclaw gateway --port 18789
```

启动后打开浏览器访问 `http://127.0.0.1:18789`，控制台就在那里。

---

## 连接你的聊天软件

### Telegram（推荐新手）

1. 去 [@BotFather](https://t.me/BotFather) 创建一个 Bot，拿到 Token
2. 在 `~/.openclaw/openclaw.json` 里配置：

```json
{
  "channels": {
    "telegram": {
      "token": "你的BotToken",
      "allowFrom": ["你的TelegramUserID"]
    }
  }
}
```

3. 重启 Gateway，直接在 Telegram 里发消息给你的 Bot

### WhatsApp

```bash
openclaw channels login --channel whatsapp
```

扫码绑定你的 WhatsApp 账号，之后发消息给自己即可触发 AI。

### Discord

在 Discord Developer Portal 创建 Bot，配置 Token 后加入你的服务器。

---

## 配置文件详解

配置文件位于 `~/.openclaw/openclaw.json`，核心字段：

```json5
{
  // 渠道配置
  "channels": {
    "whatsapp": {
      "allowFrom": ["+8613800138000"],  // 白名单，只允许这些号码
      "groups": {
        "*": { "requireMention": true }  // 群里需要@才响应
      }
    }
  },
  // 消息规则
  "messages": {
    "groupChat": {
      "mentionPatterns": ["@openclaw"]  // 触发词
    }
  }
}
```

**安全建议**：务必配置 `allowFrom` 白名单，防止陌生人调用你的 AI。

---

## Skills 技能扩展

OpenClaw 的能力可以通过 Skills 无限扩展。官方和社区提供了大量现成技能：

| Skill | 功能 |
|---|---|
| weather | 查天气预报 |
| schedule-skill | 日历/日程管理 |
| file-skill | 智能文件整理 |
| xiaohongshu | 小红书内容抓取分析 |
| healthcheck | 系统安全检查 |

安装方式：在 [ClawdHub](https://clawhub.com) 找到技能，按说明放入 Skills 目录即可。

---

## 实际使用场景

**场景一：随时随地问问题**
出门在外，打开 Telegram 发一条消息，AI 帮你查天气、查路线、查资料。

**场景二：自动化任务**
设置定时任务，每天早上 7:30 推送天气预报到你的微信。

**场景三：文件管理**
发一条"帮我整理桌面文件"，AI 自动扫描、分类、归档，全程不删文件。

**场景四：信息聚合**
"帮我看看小红书上最近关于 XX 的讨论"，AI 抓取分析后直接给你报告。

---

## 常见问题

**Q：需要一直开着电脑吗？**
A：是的，Gateway 需要在你的机器上运行。如果想 24 小时可用，可以部署在 VPS 上。

**Q：数据安全吗？**
A：所有数据在你自己的机器上，AI 调用走你自己的 API Key，OpenClaw 本身不收集数据。

**Q：支持哪些 AI 模型？**
A：支持 OpenAI、Anthropic Claude、Google Gemini、本地模型（Ollama）等主流提供商。

**Q：手机上能用吗？**
A：通过配对 iOS/Android 节点，可以实现摄像头调用、语音交互等移动端功能。

---

## 资源链接

- 📖 官方文档：[docs.openclaw.ai](https://docs.openclaw.ai)
- 💻 GitHub：[github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
- 🦞 技能市场：[clawhub.com](https://clawhub.com)
- 💬 社区 Discord：[discord.com/invite/clawd](https://discord.com/invite/clawd)

---

## 总结

OpenClaw 适合那些**想要真正掌控自己 AI 助手**的人。它不是最简单的方案，但它给你最大的自由度：你的数据、你的模型、你的渠道、你的规则。

如果你已经在用 Telegram 或 Discord，花 5 分钟装上它，体验一下从聊天软件直接调用 AI 的感觉——值得。

---

*本文基于 OpenClaw 官方文档整理，如有更新请以 [docs.openclaw.ai](https://docs.openclaw.ai) 为准。*
