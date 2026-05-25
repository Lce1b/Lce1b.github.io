---
layout: post
title: 八重神子桌面宠物
banner: ""
hidden:
  - header
  - navigator
  - related_posts
---

<div class="project-detail" markdown="1">

## 项目概述

基于 **Tauri v2** 的透明置顶桌面伴侣应用。像素动画角色「八重神子」通过 **17 种精灵动画状态**实时反映 Claude Code 的 AI 工作状态，集成双端口后端架构 (HTTP + MCP)、AI 顾问对话模式与 GPT-SoVITS 语音合成。

## 系统架构

```
┌──────────────────────────────────────────────────┐
│              八重神子 桌面宠物                      │
├───────────────┬──────────────────────────────────┤
│  Tauri 窗口    │  后端服务 (Rust)                  │
│  透明 138×210  │  ┌───────────────────────────┐  │
│  置顶 无边框   │  │ :9527  HTTP REST API       │  │
│               │  │  · hook 事件 · 顾问对话      │  │
│  精灵动画      │  │  · 巡查 · Obsidian 导入     │  │
│  · 17种状态    │  │  · TTS 代理                │  │
│  · 气泡文字    │  ├───────────────────────────┤  │
│  · 拖拽交互    │  │ :9528  MCP JSON-RPC        │  │
│               │  │  · yae_miko_show            │  │
│               │  │  · yae_miko_ask             │  │
│               │  │  · yae_miko_play            │  │
│               │  └───────────────────────────┘  │
├───────────────┴──────────────────────────────────┤
│  Claude Code Hook 桥接 → 状态驱动动画             │
│  GPT-SoVITS (:9874) → 语音合成                   │
│  DeepSeek API → AI 顾问 + 巡查                    │
└──────────────────────────────────────────────────┘
```

## 技术栈

| 层级 | 技术 |
|------|------|
| 桌面框架 | Tauri v2 (Rust) |
| HTTP 后端 | Axum · Tower |
| MCP 服务 | JSON-RPC 2.0 |
| 前端 | 原生 JavaScript · CSS 动画 |
| AI | DeepSeek API · GPT-SoVITS |
| 精灵系统 | 自定义非均匀网格 + validation.json |

## 核心功能

### 1. 精灵动画系统
- 17 种动画状态：idle / running / jumping / waving / chatting / thinking / searching / working / done / error / idle_prompt / permission 等
- `SpriteAnimator` 类读取 `validation.json` 中的自定义网格帧位置，逐帧播放
- 400ms 状态轮询防抖，3 秒无后续动作自动回 Idle

### 2. Claude Code 深度集成
- Hook 桥接：`SessionStart` 启动 exe、`PreToolUse` 工作状态、`PostToolUse` 庆祝状态
- 全局 `~/.claude/settings.json` 通过 `bridge.py` 转发所有事件
- 每个 AI 操作对应特定动画状态（搜索 → searching、构建 → working）

### 3. AI 顾问模式
- Shift + 双击触发，窗口扩展至 420×540
- 读取剪贴板内容，多轮对话精炼 prompt
- 八重神子人设 System Prompt 驱动对话风格
- 结果自动写回剪贴板

### 4. 语音合成系统
- **三级语音体系**：预录 WAV（状态语音 + 空闲搭话）→ TTS API 合成（动态文本）
- GPT-SoVITS 推理 API（:9874），按强标点分句
- AudioContext 即刻创建，拖动宠物即可激活

### 5. 系统托盘
- 显示/隐藏、巨化 2×、缩放 2×、关于、退出
- 巨化/缩放互斥（原子布尔锁）

[查看项目代码](https://github.com/Lce1b/yae-miko-claude)
{: .project-link-btn}

</div>
