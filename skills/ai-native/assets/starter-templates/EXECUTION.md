# <项目名> · 执行入口（读这份就够）

> 你是一个 Claude Code 会话。这份文件是续跑剧本——**即使上下文被清空，
> 只要重新读这份 + `docs/execution/PROGRESS.md` 就能原地续跑**。

---

## 0. 当前会话该做什么

按顺序执行：

1. **读** `CLAUDE.md`（项目纪律权威源）
2. **读** `docs/execution/PROGRESS.md` → 找出"下一个 pending 的 Batch"
3. **读** `docs/execution/DECISIONS.md` → 了解已定的技术决策（避免重复讨论）
4. **读** 对应的 `docs/execution/BATCH-XX-*.md` → 得到该 Batch 的详细任务
5. **执行** 该 Batch 的所有子任务（必要时 spawn Agent Team，模板见 `AGENT-PROMPTS.md`）
6. **验收** 跑 Batch 文档里列出的所有命令，全绿才算完
7. **更新** `PROGRESS.md` 把该 Batch 勾掉；必要时追加 `DECISIONS.md`
8. 停止，把本批摘要和下一批编号告诉用户

**不要**擅自跳批、不要重新规划路线图、不要改 `docs/0X-*.md`（默认只读，例外见 D-XXX）。

---

## 1. 项目一句话

<在这里写：项目是什么 + 给谁用 + 解决什么核心问题。>

## 2. 技术栈

<在这里写：已冻结的技术栈，每个版本号有 ADR。>

## 3. 纪律（Claude 必须遵守）

### 3.1 路线图纪律

- 路线图按 Phase 推进（Phase 0 / 1 / 2 / 3）
- Phase 2 拆 ≤ 10 个 Batch，每 Batch 有独立文档 `docs/execution/BATCH-XX-*.md`
- 严格按依赖顺序执行（依赖图见 PROGRESS.md）

### 3.2 代码纪律

<把 CLAUDE.md 硬约束段简要复述，给独立子代理空降时用>

例：
- DDD 严格分层
- 所有 Repository 查询带 tenant_id
- LLM 调用不传 temperature
- Mock 优先

### 3.3 测试纪律

- 每 Batch 必须有对应测试（具体覆盖率阈值见 CLAUDE.md）
- Batch 验收前跑 `<lint + type + test 命令>`必须全绿

### 3.4 未知库纪律

遇到 docs 未指定版本的库 / 训练数据可能过时的 API，**先用 `context7` 查最新文档**；context7 覆盖不到再 WebSearch。**不要靠记忆乱写**。

### 3.5 决策纪律

- 小决策（库选型、文件组织）：自主决定 → 追加写入 `DECISIONS.md`（带日期 + 理由）
- 大决策（架构变更、不可逆选型）：找用户确认
- 撞上规格冲突或空白：记录到 `DECISIONS.md` 的"待澄清"段，先按合理默认走

### 3.6 Agent Team 纪律

- 大批量独立子任务用 `Agent` 并行
- 串行瓶颈任务（架构骨架 / DB 迁移）不要并行
- 卡住 15 分钟未解 → spawn `codex-rescue` 做独立诊断

---

## 4. 目录结构

```
<项目名>/
├── EXECUTION.md                ← 你现在读的文件
├── CLAUDE.md                   ← 每次 session 自动加载的纪律
├── docs/
│   ├── 01-vision.md            （Phase 0 顶层意图）
│   ├── 02-architecture.md      （Phase 1 架构骨架）
│   ├── 03-data-model.md        （Phase 1 数据模型）
│   └── execution/
│       ├── README.md
│       ├── PROGRESS.md         （进度看板）
│       ├── DECISIONS.md        （技术决策日志）
│       ├── INTENT-GRAPH.md     （Phase 3 才启用）
│       ├── AGENT-PROMPTS.md    （子代理 prompt 模板）
│       └── BATCH-XX-*.md       （每 Batch 一份）
├── <backend 或 src>/           （Phase 2 创建）
├── <frontend>/                 （Phase 2 创建，如有）
└── <配置文件，如 docker-compose.yml>
```

---

## 5. 续跑剧本

**情况 A：全新 session / 刚启动 /clear**

```
用户："继续"
你：
  1. 读 CLAUDE.md（项目纪律）
  2. 读 EXECUTION.md（本文件）
  3. 读 docs/execution/PROGRESS.md
  4. 找到第一个 status ≠ completed 的 Batch
  5. 读 docs/execution/DECISIONS.md
  6. 读 docs/execution/BATCH-XX-*.md
  7. 开干
```

**情况 B：Batch 执行中断**

- PROGRESS.md 里该 Batch 已标 `in_progress` 并记录了完成的子任务
- 从"下一个未完成的子任务"接着做
- **不要重新建已经建好的文件**（先 `ls` / 读文件看）

**情况 C：用户喊停 / 切 Batch**

- 把已完成的子任务状态更新到 PROGRESS.md
- 等用户指示

---

## 6. 本轮会话的起点

如果 `docs/execution/PROGRESS.md` 还不存在 —— 说明这是整条路线图第一次被触发。

- **如果 `docs/01-vision.md` 也不存在**：还没做 Phase 0，向用户确认是否启动 Phase 0 意图凝练
- **如果 `docs/02-architecture.md` 不存在但 `01-vision.md` 已定稿**：进入 Phase 1
- **如果架构骨架已定稿但 PROGRESS.md 还没建**：进入 Phase 2 拆 Batch

---

## 7. 如何使用本模板

1. 替换所有 `<...>` 占位符
2. §1-§3 在 Phase 1 完成时填齐（之前可以是空的占位）
3. 每次新会话来都从本文件读起
