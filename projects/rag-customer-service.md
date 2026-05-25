---
layout: post
title: 企业级 RAG 智能客服 Agent
banner: ""
hidden:
  - header
  - navigator
  - related_posts
---

<div class="project-detail" markdown="1">

## 项目概述

基于 **LangGraph** 多阶段 Agent 编排的企业级智能客服系统。集成 **RAG (检索增强生成)** 技术，通过 ChromaDB 向量数据库实现语义级别知识检索，配合风险评估节点与人工转接机制，构建生产级客服 Agent。

## 系统架构

```
用户浏览器 (:3000)
    │
    ▼
Next.js 前端 ──── POST /api/chat ────► FastAPI 后端 (:8000)
                                              │
                                              ▼
                                       LangGraph Agent
                                    /    │    │    \
                     classify_intent   extract_fields   check_completeness
                                    \    │    │    /
                         search_knowledge_base  business_lookup  assess_risk
                                    \    │    │    /
                                       generate_reply
                                            │
                               ┌────────────┴────────────┐
                               │                         │
                          ChromaDB                In-Memory Session Store
                          (向量检索)               (会话上下文)
```

## Agent Pipeline 详解

| 节点 | 功能 | 说明 |
|------|------|------|
| `classify_intent` | 意图分类 | 识别 FAQ / 订单查询 / 退款投诉 / 风险升级 |
| `extract_fields` | 字段提取 | 提取订单号、产品名等结构化信息 |
| `check_completeness` | 完整性检查 | 判断信息是否充分，是否需追问 |
| `search_knowledge_base` | 知识库检索 | ChromaDB 向量相似度检索相关文档 |
| `business_lookup` | 业务查询 | 查询订单、会员等业务数据 |
| `assess_risk` | 风险评估 | 低 / 中 / 高风险分级，触发人工转接 |
| `generate_reply` | 回复生成 | LLM 综合上下文生成最终回复 |

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | Next.js 14 · React 18 · Tailwind CSS |
| 后端 | FastAPI · Python 3.11 · LangGraph |
| 向量存储 | ChromaDB · text-embedding-3-small |
| LLM | DeepSeek (OpenAI-compatible) |
| 部署 | Docker · Docker Compose |

## 核心亮点

- **SSE 流式响应**：实时推送 `node_start` 事件，用户可观察 Agent 每一步决策过程
- **审计追踪**：每个会话的决策链路持久化为 JSONL 审计日志 (`data/audit/{session_id}.jsonl`)
- **会话管理**：服务端 UUID 生成 session_id，前端 localStorage 持久化，支持清空重置
- **风险分级**：三级风险评估自动决定是否触发人工转接

## API 端点

| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/health` | 健康检查 |
| `POST` | `/chat` | 同步对话 |
| `POST` | `/chat/stream` | SSE 流式对话 |
| `GET` | `/session/{id}/history` | 会话历史查询 |

[查看项目代码](https://github.com/Lce1b/rag_test)
{: .project-link-btn}

</div>
