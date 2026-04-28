# 空项目 Starter（AI-Native 工作流）

> 一份从 0 启动新项目时可直接拷贝使用的目录骨架。配套方法论：
> [`../../AI-Native-开发工作流方法论.md`](../../AI-Native-开发工作流方法论.md)

---

## 快速开始

```bash
# 1. 拷贝骨架到新项目
cp -r ../empty-project /path/to/your-new-project

# 2. 进入新项目并初始化 git
cd /path/to/your-new-project
git init
git add .
git commit -m "feat: 项目初始化（AI-native 工作流骨架）"

# 3. 启动第一个 Claude Code 会话
claude
> 帮我做 Phase 0：意图凝练。我想做一个 <你的 idea>
```

---

## 目录结构

```
your-new-project/
├── CLAUDE.md                              ← 项目纪律权威源（每个会话自动加载）
├── EXECUTION.md                           ← 续跑剧本（新会话第一件事读这个）
├── README.md                              ← 项目自身的 README（待填）
├── docs/
│   ├── 01-vision.md                       ← Phase 0 产出（顶层意图 ≤ 5 条）
│   ├── 02-architecture.md                 ← Phase 1 产出（架构骨架）
│   ├── 03-data-model.md                   ← Phase 1 入口：选择数据模型变体
│   ├── 03-data-model.relational.md        ← 变体 A：关系型数据库（PG / MySQL / SQLite）
│   ├── 03-data-model.document-nosql.md    ← 变体 B：文档型 / NoSQL（MongoDB / Firestore）
│   ├── 03-data-model.frontend-mobile.md   ← 变体 C：纯前端 / Mobile（无自有服务端）
│   └── execution/
│       ├── README.md                      ← execution 目录索引
│       ├── PROGRESS.md                    ← Phase 2 路线图看板
│       ├── DECISIONS.md                   ← ADR-lite 决策日志
│       ├── INTENT-GRAPH.md                ← Phase 3 才启用的意图图谱
│       ├── AGENT-PROMPTS.md               ← 子代理 prompt 模板（8 个）
│       └── BATCH-template.md              ← 单 Batch 任务书模板
└── .gitignore
```

**数据模型变体使用方式**：选好你项目对应的变体后，把对应文件改名为
`03-data-model.md`（覆盖入口文件），并删掉其他变体文件。详见
`03-data-model.md` §选择指南。

---

## 4-Phase 路线图

| Phase | 输入 | 输出 | Claude Code 角色 |
|-------|------|------|-------------------|
| 0 · 意图凝练 | 自然语言愿景 | `docs/01-vision.md` | 反问者 + 起草者 |
| 1 · 契约骨架 | 顶层意图 | `CLAUDE.md` / `02-arch` / `03-data` / `DECISIONS.md` 初版 | 起草者 |
| 2 · MVP + 契约网络 | 架构骨架 | 主链路代码 + 测试 / 类型 / linter / Batch 历史 | 实施者（Agent Team） |
| 3 · 意图驱动 | 真实使用证据 | `INTENT-GRAPH` 状态流转 + 增量 feature | 实施者 + 对账者 |

详见 `../../AI-Native-开发工作流方法论.md` §3-§5。

---

## 第一次启动该做什么

### 1. 读两份关键文档（5 分钟）

- `CLAUDE.md`（看硬约束）
- `../../AI-Native-开发工作流方法论.md`（看方法论）

### 2. 在 Claude Code 里说

```
我准备做一个新项目，想法是：<5-15 句话描述你的 idea>

请按方法论 Phase 0 启动：先反问我至少 1 轮（关于动机、场景、用户行为），
然后起草 ≤ 5 条顶层意图填到 docs/01-vision.md，让我审。
不要开 Agent Team，Phase 0 只是凝练。
```

### 3. Phase 0 完成后说

```
顶层意图已定稿。请按 Phase 1 起草：
- CLAUDE.md（项目纪律 + 技术栈选型理由）
- docs/02-architecture.md（架构骨架）
- docs/03-data-model.md（数据库第一版 schema）
- docs/execution/DECISIONS.md（≥ 5 条初始 ADR）

每个不可逆决策都要立 ADR。技术栈备选先列再让我拍板。
```

### 4. Phase 1 完成后说

```
架构骨架已定稿。请按 Phase 2 拆 ≤ 10 个 Batch：
- 每个 Batch 推进哪条顶层意图
- 严格依赖图
- 每个 Batch 的测试 / 类型 / linter 配置同步建

写到 docs/execution/PROGRESS.md，并给每个 Batch 起草 BATCH-XX-*.md。
我审完后我们开第一个 Batch。
```

---

## 关键纪律（最容易忘）

- **项目纪律入仓库，不入 memory**：memory 绑用户机器，仓库走分发
- **Phase 0/1 不要启用 INTENT-GRAPH**：会写空想意图
- **测试 / 类型 / linter 在 Phase 1 配，每 Batch 跑**：契约网络是真相源
- **每个 Batch 末尾人类必须验收**："AI 报告全绿"≠"真的对"
- **Agent Team 并行度看上下文风险**：不熟的领域从 2-3 开始

---

## 如果你已经在 AI-Native 仓库里看到这份 starter

它**只是模板**，不是要你在 AI-Native 仓库内启动新项目。要用就：

```bash
cp -r ./starters/empty-project /path/to/your-new-project
```

拷出去之后再 `git init`，与 AI-Native 仓库无关。

