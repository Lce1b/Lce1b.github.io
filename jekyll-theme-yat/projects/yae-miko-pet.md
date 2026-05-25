---
layout: post
title: 八重神子桌面宠物 — AI 桌宠 Agent
banner: ""
hidden:
  - header
  - navigator
  - related_posts
---

<div class="project-detail" markdown="1">

## 项目概述

基于 **Tauri v2 (Rust)** 的透明置顶 AI 桌面伴侣。像素精灵角色「八重神子」通过 **17 种动画状态** + **5 维情感系统** + **LLM Agent 循环**，实现从被动状态展示到主动智能伴侣的进化。集成 **ReAct Agent 工具调用**、**三层记忆体系**、**实时 WebSocket 通信**、**GPT-SoVITS 语音合成** 与 **Claude Code 深度联动**。

## 系统架构

```
┌──────────────────────────────────────────────────────────┐
│                    八重神子 桌面宠物 v2                      │
├──────────────┬───────────────────────────────────────────┤
│  Tauri 窗口   │  后端服务 (Rust · Axum)                    │
│  透明 200×260 │  ┌────────────────────────────────────┐  │
│  置顶 无边框  │  │ :9527  HTTP REST API (18 端点)      │  │
│              │  │  · Agent 聊天 · 巡查 · 问候          │  │
│  精灵动画     │  │  · TTS · 情感 · 记忆 · 屏幕感知     │  │
│  · 17 状态    │  │  · WebSocket 实时推送               │  │
│  · 气泡 UI    │  ├────────────────────────────────────┤  │
│  · Live2D†    │  │ :9528  MCP JSON-RPC                │  │
│              │  │  · yae_miko_show / ask / play       │  │
│  WebSocket   │  └────────────────────────────────────┘  │
│  实时通信     │                                          │
├──────────────┴──────────────────────────────────────────┤
│  Claude Code Hooks → bridge.py → 状态驱动动画            │
│  GPT-SoVITS v2ProPlus (:9874) → 流式语音合成              │
│  DeepSeek V4 → Agent 推理 + 巡查 + 问候                  │
│  wttr.in + 腾讯新闻 → 启动问候 (天气+新闻)                 │
└──────────────────────────────────────────────────────────┘
```

## 技术栈

| 层级 | 技术 |
|------|------|
| 桌面框架 | Tauri v2 (Rust) |
| HTTP 后端 | Axum · Tokio · Tower |
| 实时通信 | WebSocket (即时推送) + HTTP 轮询 (30s 兜底) |
| MCP 服务 | JSON-RPC 2.0 |
| 前端 | 原生 JavaScript · CSS 动画 · Web Audio API |
| AI Agent | DeepSeek V4 · ReAct 循环 · 6 个内置工具 |
| 记忆系统 | JSON 长期记忆 + SessionBuffer 工作记忆 + Episode 情节记忆 |
| TTS | GPT-SoVITS v2ProPlus · 流式分块播放 |
| 精灵系统 | 自定义非均匀网格 + validation.json |

## 核心功能

### 1. AI Agent 循环 (ReAct Pattern)
- **Think → Act → Observe** 多步推理，最多 5 步工具调用链
- 6 个内置工具：`get_weather` / `get_news` / `search_memory` / `get_screen_context` / `read_file` / `read_clipboard`
- Trait 抽象工具注册表，可扩展
- 托盘菜单"聊天模式"一键触发，Win/Mac 原生交互

### 2. 三层记忆体系
- **工作记忆**：会话缓冲区，自动维护最近 20 条消息上下文
- **情节记忆**：对话结束后 LLM 自动生成摘要 → 下次对话注入
- **语义记忆**：embedding 向量 + 余弦相似度检索（DeepSeek Embedding API）
- Fallback 机制：API 不可用时退回到关键词匹配

### 3. 情感系统
- 5 种心情：Happy / Neutral / Bored / Annoyed / Sleepy
- 好感度 + 精力双维度数值驱动
- 6 种触发事件：摸头、互动、空闲衰减、深夜、巡查正向反馈、离开
- 心情影响搭话频率和语气："哼，让神子等了这么久..."

### 4. 代码巡查 (30min 间隔)
- 收集上下文：剪贴板 · git log · git status · Claude Code 对话记录
- 剪贴板哈希去重，Claude 对话裁剪至 500 字/5 轮
- 注入长期记忆 + 活动窗口感知 → DeepSeek 生成八重神子风格观察

### 5. 语音合成系统
- **三级体系**：预录 WAV (42 条) → TTS API 动态合成 → 流式分块播放
- GPT-SoVITS v2ProPlus 自训练模型，八重神子声线
- AudioContext 即刻创建，WebSocket 进度推送
- TTS 优先级队列：气泡语音插队，API 合成排队

### 6. Claude Code 深度集成
- `SessionStart` 自动启动 exe
- `PreToolUse` → 动画状态映射 (Read→reading, Write→building, Bash→running...)
- `PostToolUse` → Celebrating / `PostToolUseFailure` → Failed
- 权限弹窗 → Waving 状态 + "需要您进行确认" 语音

### 7. 启动问候 & 实时天气新闻
- wttr.in 宁波天气 + 腾讯新闻热点 → DeepSeek 生成八重神子风格简报
- 3 秒延迟自动触发，30 秒缓存防抖

### 8. 前端特性
- WebSocket 即时推送 + 30 秒 HTTP 轮询兜底
- 布局编辑器 (Ctrl+Shift+L)：拖拽定位精灵和气泡，实时显示坐标
- 长文本大气泡自动切换 (>80 字)
- 系统托盘菜单：显示/隐藏 · 聊天模式 · 巨化 2× · 缩放 2×

### 9. 开发面板
- 嵌入式调试面板 (`/dev/panel`)，编译时 `include_str!` 内联
- 问候刷新 · 巡查触发 · 自定义 TTS 播报 · 操作日志
- 独立的 Web Audio 播放管道

## 测试覆盖

- **102 个 Rust 单元测试** (cargo test)，覆盖状态机/TTS/记忆/情感/工具
- 前 端测试清单：22 项手动 E2E 验证

## API 端点

| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/api/current` | 当前状态 (animation/bubble/mood/scale/drag) |
| `POST` | `/api/state` | 设置状态 (Claude hook 桥接) |
| `POST` | `/api/agent/chat` | Agent 聊天 (ReAct 循环 + 工具调用) |
| `POST` | `/api/patrol` | 代码巡查 |
| `POST` | `/api/pet` | 摸头交互 (好感度 +2) |
| `POST` | `/api/mood/event` | 情感事件 (idle/latenight/interact/useraway) |
| `GET` | `/api/tts` | TTS 语音合成 |
| `GET` | `/api/tts/stream` | TTS 流式合成 |
| `GET` | `/api/ws` | WebSocket 实时推送 |
| `GET` | `/api/greeting` | 缓存问候 |
| `POST` | `/api/greeting/refresh` | 刷新问候 |
| `GET` | `/dev/panel` | 调试面板 |

† Live2D：模型已就绪 (Cubism 3 八重神子，含 8 组动作 + 物理 + 语音)，懒加载架构设计中。

[查看项目代码](https://github.com/Lce1b/yae-miko-claude)
{: .project-link-btn}

</div>
