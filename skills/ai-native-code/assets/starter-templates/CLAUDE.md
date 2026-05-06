# <项目名> · Claude 项目上下文

> 这份文件每个 Claude Code 会话自动加载。**项目纪律的唯一权威源**。
> 模板填法见末尾"如何使用本模板"。

---

## 必读

**每个新 session 的第一件事**：读 `EXECUTION.md`（仓库根）和 `docs/execution/PROGRESS.md`，按里面的纪律办事。

---

## 项目一句话

<在这里写：项目是什么 + 给谁用 + 解决什么核心问题。≤ 30 字。>

例：
> AI 小说生成 SaaS，给独立作者用，解决"想写完一本书但写不下去"的问题。

---

## 顶层意图（来自 docs/01-vision.md）

<把 docs/01-vision.md 里 ≤ 5 条顶层意图的标题列在这里，作为 Claude 每次会话的"为什么做这个"提示。>

例：
1. 作者能从 0 写完一本 ≥ 10 万字的完整小说
2. 作品质量稳定且可观测
3. AI 成本对作者透明可控

---

## 硬约束（忘了会出事）

<这一段在 Phase 1 起草时填。每条都必须能在某条 ADR 找到理由——指向 DECISIONS.md 编号。>

例：
1. **DDD 分层**：`interfaces → application → domain ← infrastructure`，由 importlinter 强制（D-002）
2. **多租户**：所有 Repository 查询带 tenant_id（D-007）
3. **LLM 调用不传 temperature**：用 provider API 默认值（D-004）
4. **Mock 优先**：默认用 MockLLMService，真 LLM 只在 smoke 跑（D-006）
5. **规格默认只读**：`docs/0X-*.md` 默认不要改，例外条件见 D-XXX
6. **未知库先查**：陌生 API 先用 context7 查，别靠记忆

---

## 协作模式：Agent Team

Claude Code 支持 Agent Team（多 Agent 协同 + 共享 TaskList）。**默认用 Team 完成跨层 / 多步任务**。

触发场景：
- 跨层实现（同一 Batch 要动 domain + application + infrastructure + interfaces）
- 可并行的独立工作（后端 + 前端 + 测试）
- 多步骤项目（研究 → 规划 → 编码 → 校验）

标准流程：
1. `TeamCreate` 建 team
2. `TaskCreate` 拆解 Batch 内子任务
3. `Agent` 派 teammate（写代码 → general-purpose；研究 → Explore；设计 → Plan）
4. `TaskUpdate` 用 `owner` 指派给 teammate
5. teammate 完成会自动回消息，**不要轮询**
6. 收尾：`SendMessage shutdown_request` → `TeamDelete`

并行度纪律：
- Phase 2 主链路：3-5 个 agent
- Phase 3 实施意图卡：3-5 个 agent
- 独立子任务（如领域建模）：可达 5-9
- 已有项目改老代码：1-2 + review agent

不开 team：单文件改 / 纯查询 / Phase 0 / 已无契约网络的项目。

---

## 技术栈

<在这里写已冻结的技术栈选型。每个版本号 + 理由都应该在 DECISIONS.md 有 ADR。>

例：
- Python 3.12 + FastAPI + SQLAlchemy 2 async + Pydantic v2
- PostgreSQL 16 + Redis 7 + MinIO
- Vue 3.5 + TypeScript + Vite + Naive UI + Pinia
- 测试：pytest + pytest-asyncio + factory-boy
- 质量：ruff + mypy --strict + importlinter + bandit + pre-commit

---

## 路线图

<在这里写：当前在哪个 Phase。MVP 路线图 ≤ N 个 Batch，详见 docs/execution/PROGRESS.md。>

例：
> 当前 Phase 2，B0-B9 共 10 个 Batch；MVP 验收意图："作者能创建小说 → 生成 10 章 → 导出 DOCX"。

---

## 规划范式

<这一段在 Phase 3 启用 INTENT-GRAPH 时加进来。Phase 0/1/2 期间删掉本节即可。>

MVP 已落地后，未实现的需求改走**意图驱动 + Living Spec**，文件在 `docs/execution/INTENT-GRAPH.md`。

**规则**：
- 用户问"X 功能要不要做" → **不要直接开 Batch**。先看 INTENT-GRAPH 有无对应意图卡；有则评估证据、无则先写成意图卡
- 完成任何 Batch 时，需回写 INTENT-GRAPH §3 对账表
- 收到关于未实现入口的真实信号时，把对应意图从 `pending` 升 `probing`
- 检测到证伪条件命中时，状态置 `falsified`，并按 ADR 流程反向修剪规格

**禁止**：
- 把 INTENT-GRAPH 当 PRD（不写人天 / deadline / 甘特图）
- 不带证据 + 没有 user override 就升 `validated`
- 用主观词（"用户体验差"）写证伪条件

---

## 语言

<在这里写：会话语言、commit 语言、注释语言、变量名语言。>

例：
> 回复、commit message 和注释用简体中文；变量名、API 字段、日志用英文。

---

## 如何使用本模板

1. 把每段 `<...>` 占位符替换成项目实际内容
2. 删掉所有"例：" 段（那些只是参考）
3. Phase 0/1/2 期间"规划范式"段可以先删；Phase 3 启用时加回来
4. 任何硬约束新增 / 修改都必须先在 DECISIONS.md 立 ADR，再回填本文件
