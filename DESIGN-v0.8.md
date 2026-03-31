# OpenClaw Harness v0.8 — Design Document

> Author: Jarvis | Date: 2026-03-31
> Status: Draft — pending Xiang review

## 1. DeerFlow 深度分析

### DeerFlow 是什么
ByteDance 开源的 LangGraph-based SuperAgent 框架（2.0 是完全重写）。
GitHub Trending #1，架构扎实：

```
DeerFlow 架构：
├── LangGraph Server (port 2024)     ← Agent runtime + workflow
├── Gateway API (port 8001)          ← REST API
├── Frontend (port 3000)             ← Next.js Web UI
├── Nginx (port 2026)                ← Reverse proxy
└── Provisioner (optional)           ← K8s sandbox
```

**核心组件：**
- **Harness/App 分层** — harness 是可发布的 framework（`deerflow-harness` package），app 是应用层。harness 永远不 import app（有 CI 测试强制）
- **12 个有序 Middleware** — ThreadData → Uploads → Sandbox → DanglingToolCall → Guardrail → Summarization → TodoList → Title → Memory → ViewImage → SubagentLimit → Clarification
- **Subagent 系统** — Registry + Executor，支持 Claude Code / Codex CLI 作为 ACP provider
- **Sandbox 隔离** — 每个 thread 有独立的 workspace/uploads/outputs 目录
- **Memory** — 异步提取，长期存储
- **Skills** — 可发现、可加载

### DeerFlow 的瓶颈（我们的机会）

| DeerFlow 痛点 | 为什么是痛点 | OpenClaw 的天然优势 |
|---------------|-------------|-------------------|
| **Docker + LangGraph + Python** | 安装要 30+ 分钟，普通开发者劝退 | `clawhub install` 一行命令 |
| **重复造轮子** | 自建 memory/channels/sandbox | OpenClaw 已有全套（memory_search、Telegram/Discord、exec sandbox） |
| **Claude Code 是后加的** | 通过 `ClaudeChatModel` wrapper，不是原生 | OpenClaw ACP 是原生集成，`sessions_spawn` 即用 |
| **没有 chat-first 体验** | 必须开 Web UI | 你在 Telegram 里说一句话就能启动 |
| **config.yaml 管理地狱** | 改模型要改 yaml + 重启 | OpenClaw config 热加载 |
| **没有人类反馈循环** | Agent 自己跑完就完了 | Lead Agent（我）可以随时问 Xiang 确认 |

### DeerFlow 值得学的

1. **Harness/App 分层架构** — 核心 framework 不依赖应用层，这个设计哲学要带过来
2. **Middleware 模式** — 把 orchestration 拆成可组合的步骤
3. **TodoList/Plan Mode** — 结构化任务追踪（SPRINT.md 的进化版）
4. **Subagent Registry** — 不硬编码 agent，通过注册表动态发现
5. **Boundary 测试** — CI 强制依赖方向，防止架构腐化

## 2. OpenClaw Harness v0.8 定位

### 一句话
**"DeerFlow 的 orchestration 智慧 + OpenClaw 的零配置体验"**

### 差异化

```
DeerFlow:  "我是一个完整的 agent 运行时，什么都自己建"
Harness:   "我是一个 orchestration layer，站在 OpenClaw 的肩膀上"
```

| 维度 | DeerFlow 2.0 | OpenClaw Harness v0.8 |
|------|-------------|----------------------|
| 安装 | Docker + 30min setup | `clawhub install harness-factory` |
| Runtime | LangGraph Server | OpenClaw Gateway（已运行） |
| Builder | Codex/Claude via wrapper | Claude Code via native ACP |
| Memory | 自建 memory system | OpenClaw memory_search / memory_get |
| Channels | Feishu/Slack/Telegram wrapper | OpenClaw 原生（Telegram/Discord/WhatsApp/etc） |
| Sandbox | Docker/Local sandbox | OpenClaw exec（已有 elevated mode） |
| Skills | 自建 skills loader | OpenClaw skills + ClawHub |
| UI | Web UI (port 2026) | Chat-first（Telegram/Discord） |
| Cost | 需要独立服务器 | 跑在现有 OpenClaw 上，零额外成本 |

## 3. v0.8 架构设计

### 核心理念：5 个 Phase 变成 5 个可执行的 Agent Role

```
v0.7:  SKILL.md 文档教你怎么做
v0.8:  Lead Agent 自动驱动整个 pipeline

User: "给 ArkRoute 加个 subscription page"
  │
  ▼
┌─────────────────────────────────────────────┐
│  LEAD AGENT (Jarvis / 你的 OpenClaw agent)  │
│                                              │
│  Phase 1: SCOUT                              │
│  → 自动读代码、理解架构、识别风险            │
│  → 输出: SPRINT.md                           │
│                                              │
│  Phase 2: BUILD                              │
│  → sessions_spawn(runtime:"acp") Builder     │
│  → Builder 读 SPRINT.md、写代码、跑检查      │
│  → 输出: code changes + BUILDER_REPORT.md    │
│                                              │
│  Phase 3: REVIEW                             │
│  → sessions_spawn(runtime:"acp") Reviewer    │
│  → 独立 session 只看代码不看 Builder 推理     │
│  → 输出: REVIEWER_REPORT.md + 评分            │
│                                              │
│  Phase 4: ITERATE (if score < 7.0)           │
│  → Lead 写 REVIEW.md                         │
│  → 重新 spawn Builder，附带 review 反馈      │
│  → 最多 3 轮                                  │
│                                              │
│  Phase 5: SHIP                               │
│  → git commit + push                          │
│  → deploy (项目特定)                          │
│  → 输出: HARNESS_REPORT.md                    │
│  → 通知 User (Telegram/Discord)               │
└─────────────────────────────────────────────┘
```

### Agent Templates

```
agents/
  builder.md       ← Builder 的 system prompt（给 Claude Code ACP）
  reviewer.md      ← Reviewer 的 system prompt（独立 review session）
  planner.md       ← Planner prompt（可选，复杂任务用）
```

**builder.md 核心约束：**
- 只能修改 SPRINT.md 声明的文件
- 每个改动必须跑 compile check
- 必须写 BUILDER_REPORT.md
- 不能添加未声明的依赖
- 不能有 TODO/FIXME/console.log

**reviewer.md 核心约束：**
- 看不到 Builder 的推理过程
- 必须从 4 个维度评分
- 必须标记 Critical / Warning / Info
- 安全问题自动 FAIL

### Middleware 模式（学自 DeerFlow）

Lead Agent 在驱动 pipeline 时，按顺序执行以下 middleware：

```python
# 概念性伪代码 — 实际实现在 SKILL.md 的流程指令中

middlewares = [
  ScopeCheck(),       # 检查改动范围是否超出声明
  CompileCheck(),     # py_compile / tsc --noEmit
  SecurityScan(),     # grep auth/CORS/secrets
  RegressionCheck(),  # 确保 DO NOT BREAK 清单通过
  ScoreEvaluate(),    # 4维度评分
  IterateOrShip(),    # <7.0 → iterate, ≥7.0 → ship
]
```

这不是真正的 Python middleware — 而是 Lead Agent SKILL.md 中的结构化检查步骤。与 DeerFlow 的区别是：DeerFlow 用 Python 代码实现 middleware，我们用 Lead Agent 的推理能力 + 工具调用实现同等效果。

### Sprint Templates（v0.8 核心交付）

```
templates/
  backend-api.md          ← API endpoint sprint 模板
  frontend-component.md   ← React/Next.js 组件模板
  fullstack-feature.md    ← 前后端联合模板（自动拆分）
  security-audit.md       ← 安全审计模板
  data-pipeline.md        ← ETL/scraping 模板
```

每个模板包含：
- 预填的 SPRINT.md 骨架
- 该类型任务的 DO NOT BREAK 清单
- 常见 anti-patterns
- 评分权重调整（安全类任务 Security 权重提高到 40%）

### 与 OpenClaw ACP 的集成点

```yaml
# Builder spawn — 最核心的集成
sessions_spawn:
  runtime: "acp"
  agentId: "claude"         # 或未来支持 codex/gemini
  mode: "run"               # 一次性任务
  task: |
    [从 builder.md 模板生成的完整 prompt]
    [包含 SPRINT.md 内容]
    [包含项目 context]
  cwd: "/path/to/project"
  sandbox: "inherit"

# Reviewer spawn — 独立 session
sessions_spawn:
  runtime: "acp"
  agentId: "claude"
  mode: "run"
  task: |
    [从 reviewer.md 模板生成的 prompt]
    [只包含代码 diff，不包含 Builder 推理]
  cwd: "/path/to/project"
  sandbox: "inherit"
```

## 4. 真实 Use Cases（验证载体）

| # | Project | Sprint | 验证什么 |
|---|---------|--------|---------|
| 1 | **ArkRoute** — Subscription page | 前端 Sprint | Dashboard 增强，Stripe checkout 完善 |
| 2 | **TrendMuse** — Favorites API | 后端 Sprint | 新 API endpoint + DB migration |
| 3 | **TrendMuse** — Favorites UI | 前端 Sprint | React 组件，调后端 API |
| 4 | **FashionFlow** — AI Stylist | 全栈 Sprint | 自动拆分前后端 |

每个 use case 生成完整 artifacts：
```
examples/
  arkroute-subscription/
    SPRINT.md
    BUILDER_REPORT.md
    REVIEWER_REPORT.md
    REVIEW.md (if iterated)
    HARNESS_REPORT.md
  trendmuse-favorites-api/
    ...
```

## 5. 文件结构（v0.8 目标）

```
openclaw-harness/
├── README.md                    ← 更新：对标 DeerFlow 的对比表
├── ROADMAP.md                   ← 更新：v0.8 → v1.0
├── DESIGN-v0.8.md               ← 本文档
├── CHANGELOG.md                 ← 更新
├── skills/
│   └── harness-engineering/
│       └── SKILL.md             ← 核心：从文档升级为可执行 pipeline
├── agents/                      ← 新增：agent role templates
│   ├── builder.md
│   ├── reviewer.md
│   └── planner.md
├── templates/                   ← 新增：Sprint 模板库
│   ├── backend-api.md
│   ├── frontend-component.md
│   ├── fullstack-feature.md
│   ├── security-audit.md
│   └── data-pipeline.md
├── examples/                    ← 升级：真实 artifacts
│   ├── arkroute-subscription/
│   ├── trendmuse-favorites-api/
│   ├── trendmuse-favorites-ui/
│   └── ...
└── docs/
    ├── vs-deerflow.md           ← 新增：详细对比
    └── acp-integration.md       ← 新增：ACP 集成指南
```

## 6. 成功标准

v0.8 发布前必须满足：

1. **至少 2 个真实 Sprint** 用完整 ACP pipeline 跑完（Builder spawn → Review → Ship）
2. **每个 Sprint 有完整 artifacts** 在 examples/ 中
3. **SKILL.md 是可执行的** — Lead Agent 读完就能自动驱动，不需要人手动写 SPRINT.md
4. **Agent templates 经过验证** — builder.md 和 reviewer.md 在真实任务中产出了 ≥7.0 的代码
5. **README 对标 DeerFlow** — 有清晰的 "Why not DeerFlow?" 对比表

## 7. 非目标（v0.8 不做）

- ❌ CLI 工具（`npx harness init`）— 留给 v1.0
- ❌ Web UI — OpenClaw 已有 chat-first 体验
- ❌ 自建 sandbox — 用 OpenClaw exec
- ❌ 自建 memory — 用 OpenClaw memory_search
- ❌ 多 agent runtime 支持 — v0.8 只支持 Claude Code ACP
- ❌ GitHub Action — 留给 v1.0

## 8. 执行计划

### Step 1: Agent Templates（今天）
写 builder.md 和 reviewer.md，基于 4 个 TrendMuse sprint 的经验

### Step 2: 第一个真实 ACP Sprint（今天/明天）
用 ArkRoute subscription page 作为验证，完整跑一遍 pipeline

### Step 3: Sprint Templates（本周）
从 Step 2 的经验提炼 backend-api.md 和 frontend-component.md

### Step 4: 第二个真实 Sprint + Examples（本周）
TrendMuse favorites feature，生成完整 artifacts

### Step 5: SKILL.md 重写 + README 更新（本周末）
把所有学到的东西固化到 SKILL.md 和 README

### Step 6: ClawHub 发布 v0.8
