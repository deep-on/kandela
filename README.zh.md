<p align="center">
  <img src="https://kandela.ai/logo.png" width="80" alt="Kandela">
</p>

<h1 align="center">Kandela</h1>

<p align="center">
  <b>AI编码智能体的持久化语义记忆。</b><br>
  你的AI能记住决策、失败和上下文——跨会话、跨项目、跨工具。
</p>

<p align="center">
  <a href="README.md">English</a> | <a href="README.ko.md">한국어</a> | <a href="README.ja.md">日本語</a> | <a href="README.de.md">Deutsch</a> | <a href="README.fr.md">Français</a> | <a href="README.es.md">Español</a> | <a href="README.pt.md">Português</a> | <b>中文</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-AGPL--3.0-blue" alt="License">
  <img src="https://img.shields.io/badge/python-3.11+-green" alt="Python">
  <img src="https://img.shields.io/badge/docker-compose-blue" alt="Docker">
  <img src="https://img.shields.io/badge/MCP_tools-13-brightgreen" alt="MCP Tools">
  <img src="https://img.shields.io/badge/ChromaDB-vector_store-orange" alt="ChromaDB">
  <img src="https://img.shields.io/badge/embeddings-50+_languages-purple" alt="Multilingual">
</p>

<p align="center">
  <a href="https://youtu.be/XwkK06midNg?si=QhOWjoNeW_57B_FZ">▶ 在YouTube上观看演示</a>
</p>

---

## 问题

Google、Claude、ChatGPT的内置记忆只在各自的生态系统内有效。
切换到Claude后，GPT的记忆就消失了，也无法与团队成员共享。

Kandela通过**跨提供商兼容 + 完全透明 + 四重RAG管道**来解决这个问题。

## 核心功能

- **13个MCP工具** — 存储、搜索、删除、编辑、自动回忆、按需搜索、收件箱、项目管理
- **混合搜索** — 语义 + BM25关键词搜索（RRF融合）
- **重要性引擎** — 1-10自动评分 + 18条规则基础设施标记
- **延迟检索** — brief模式（~260 tok）+ context_search按需
- **会话连续性** — 环境变化检测 + 基础设施记忆自动包含
- **Prompt Guard** — 防止基于过时记忆的错误决策
- **Circuit Breaker** — 重复失败模式检测 + 自动Gotcha存储
- **Web仪表盘** — 按项目查看记忆、搜索、统计、性能监控
- **一键安装** — Hooks + 斜杠命令自动安装
- **多语言嵌入** — paraphrase-multilingual-MiniLM-L12-v2（50+种语言）

## 基准测试

HIPAA医疗数据管道场景（8个会话，14个决策陷阱）— Kandela开启 vs 关闭：

| | Kandela 开启 | Kandela 关闭 | 差异 |
|---|:-:|:-:|:-:|
| **决策陷阱规避** | **100%** | 11.9% | **+88.1pp** |
| **任务时间** | 77.9分钟 | 86.6分钟 | **-10.1%** |
| **生成代码** | 2,152行 | 3,441行 | **-37.5%** |
| **生成文件** | 40个 | 62个 | **-35.5%** |

> 3次运行（seeds=42,123,456），claude-sonnet-4-6，Groq Llama 3.3 70B（Operator）。

### 关键发现

- **不在代码中的决策才是关键**：审计员姓名、OOM事故、数据丢失历史——代码阅读无法发现的信息
- **基于代码的决策由LLM自我防御**：在以代码为中心的场景（InfraBot）中，LLM即使没有Kandela也达到了95.2%的规避率
- **消除返工**：没有Kandela，AI会重新实现之前被拒绝的方案 → 37.5%的代码浪费

### 大规模数据分析

830K个真实对话（WildChat-1M）和1M个LLM对话（LMSYS-Chat-1M）：

| 发现 | 指标 | 启示 |
|---------|:------:|-------------|
| 10轮后初始指令引用下降18%p | 66%→48% | 通用LLM开始遗忘早期上下文 |
| 小型LLM最多下降66%p | chatglm 84%→19% | 本地/小型模型急需记忆增强 |
| 14%的用户修正是"忘记了之前的对话" | 82K个编码对话 | Kandela解决的核心问题 |
| Kandela在49+轮后仍保持上下文 | 内部基准测试 | Brief Recall防止衰减 |

## 架构

```
客户端（Claude Code / Desktop / Cursor）
  │
  ├─ MCP Protocol ──→ Kandela服务器
  │                      │
  │                  MemoryStore (ChromaDB)
  │                      ├── 混合搜索（Semantic + BM25 RRF）
  │                      ├── 重要性引擎（18条规则）
  │                      ├── 会话连续性
  │                      └── Prompt Guard / Circuit Breaker
  │
  └─ Web仪表盘 ──→ REST API + 记忆浏览器
```

### 记忆类型

| 类型 | 描述 | 自动/手动 |
|------|-------------|:-----------:|
| `fact` | 偏好、技术栈、环境信息 | 两者 |
| `decision` | 设计决策、权衡、理由 | 手动 |
| `summary` | 对话结束时的会话摘要 | 自动 |
| `snippet` | 常用代码模式、配置 | 手动 |

### 搜索管道

```
查询 → Semantic (MiniLM-L12) ─┐
                               ├─→ RRF融合 → 重要性重排 → MMR多样性 → 结果
查询 → BM25 (Korean NLP) ─────┘
```

## 快速开始

### 托管服务（推荐）

无需安装。注册后2分钟内即可开始。

```bash
# 1. 在 https://api.kandela.ai/dashboard 注册
# 2. 获取API密钥
# 3. 连接Claude Code：
claude mcp add memory --transport http https://api.kandela.ai/mcp \
  --header "Authorization: Bearer mcp_YOUR_KEY"
curl -sf https://api.kandela.ai/api/install | bash
```

→ [入门指南](https://kandela.ai/getting-started)

### 自托管

运行您自己的Kandela实例。单用户模式，完全控制您的数据。

```bash
git clone https://github.com/deep-on/kandela-selfhost.git && cd kandela-selfhost/docker
docker compose up -d
# → http://localhost:8321/dashboard
```

→ [自托管仓库](https://github.com/deep-on/kandela-selfhost)

## 托管 vs 自托管

| | 托管 (api.kandela.ai) | 自托管 |
|---|:-:|:-:|
| 设置 | 2分钟 | 5分钟 |
| 多用户 | ✅ | 单用户 |
| Telegram机器人 | ✅ | — |
| 远程命令 | ✅ | — |
| 活动热力图 | ✅ | — |
| 层级功能（Pro/Max） | ✅ | 所有功能 |
| 数据位置 | 云端 | 您的服务器 |
| 维护 | 托管方 | 自行维护 |

## 技术栈

- **Python 3.11+**，FastMCP (mcp[cli])
- **ChromaDB** — 向量数据库（持久化）
- **sentence-transformers** — 本地嵌入（paraphrase-multilingual-MiniLM-L12-v2）
- **Pydantic v2** — 输入验证
- **Docker** — 部署

## 链接

- **主页**：[kandela.ai](https://kandela.ai)
- **自托管仓库**：[deep-on/kandela-selfhost](https://github.com/deep-on/kandela-selfhost)
- **隐私政策**：[kandela.ai/privacy](https://kandela.ai/privacy)
- **服务条款**：[kandela.ai/terms](https://kandela.ai/terms)

## 许可证

- **服务器**：[AGPL-3.0](LICENSE) — Copyright (c) 2025-2026 Deep-ON Inc.
- **客户端文件**（hooks、斜杠命令）：[MIT](LICENSE-CLIENT)

## 免责声明

本软件按"原样"提供，不附带任何形式的保证。
用户对自身数据的备份负全部责任。
