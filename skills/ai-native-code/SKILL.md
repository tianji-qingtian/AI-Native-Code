---
name: ai-native
description: >
  AI-Native 开发工作流方法论——用 Claude Code 作为主力开发工具的工程纪律。
  同时支持空项目和已有代码库两种场景。
  触发场景：用户说"新项目"、"启动项目"、"AI-native"、"意图凝练"、"契约骨架"、
  "INTENT-GRAPH"、"意图卡"、"Batch"、"Agent Team"、"空项目 starter"、
  "已有项目接入"、"老项目"、"现有代码"、"改造现有项目"、"Phase 0/1/2/3"、
  "顶层意图"、"ADR"、"DECISIONS"、"反推意图"、"codebase 考古"、
  "机器可读契约"、"多 Agent 并行"、"项目纪律"、"续跑剧本"、
  "想给项目加测试网络"、"代码库太乱想整理"、"隐性约束"、
  或用自然语言描述一个想做的产品 idea。
  即使用户只是模糊地说"想做一个XX"或"想整理现有代码库"，也应该触发此 skill。
---

# AI-Native 开发工作流

用 Claude Code 作为主力开发工具的工程方法论。核心不是"让 AI 自主"，是让多 Agent 在共享契约下并行干活。

## 0. 责任分工铁律

任何阶段下，这张表不可违反。**把人类的事委托给 AI 是这套方法论失败的最大原因**。

| 工作 | 谁做 | AI 能替代？ |
|------|------|------------|
| 商业判断 / 用户访谈 | 人类 | ❌ |
| 顶层意图定义 | 人类（AI 可起草，人类定稿） | ❌ |
| 架构选型 / 不可逆决策 | 人类（AI 列选项，人类拍板） | ❌ |
| 验收（"AI 报告全绿 ≠ 真的对"） | 人类 | ❌ |
| Batch 任务书起草 | AI 起草 + 人类审 | ✅ |
| 单 Batch 内代码实施 | AI（Agent Team） | ✅ |
| 测试 / 类型 / linter 守门 | 工具自动化 | ✅ |
| 决策溯源（DECISIONS.md） | AI 起草 + 人类审 | ✅ |

## 1. 4-Phase 框架

无论空项目还是已有源码，全部走这 4 个阶段：

```
Phase 0  上下文凝练     人类主笔 + AI 反问
   │     空项目 → 顶层意图（前瞻 ≤ 5 条）
   │     已有项目 → codebase 考古（回顾）
   ▼
Phase 1  契约骨架       传统规格 + ADR
   │     空项目 → 架构 / 分层 / DB schema 第一版
   │     已有项目 → 反推意图 + 隐性约束显性化
   ▼
Phase 2  机器可读契约   测试 / 类型 / linter / importlinter
   │     这是真相源，所有后续工作的安全网
   ▼
Phase 3  意图驱动       INTENT-GRAPH + Agent Team
         此时 AI 才能真正发挥规模优势
```

**两个铁律**：
1. Phase 0/1 不能纯意图驱动——AI 没素材可推
2. Phase 2 不能跳过——没有契约网络，Phase 3 Agent Team 并行会崩

## 2. 场景识别

首先判断用户面对的是哪种场景：

**场景 A：空项目**——用户脑里有 idea，可能伴随口语描述、原型图。
**场景 B：已有源码**——代码已跑在生产，可能有过时 PRD、缺测试、有合规约束。

询问用户确认场景，然后进入对应 Phase。

---

## 3. 场景 A：空项目

### Phase 0 · 意图凝练

**目标**：把自然语言愿景凝练为 ≤ 5 条可证伪的顶层意图。

**Claude 角色**：反问者 + 起草者。**不要直接写代码，不要启用 Agent Team。**

**工作流**：
1. 用户用自然语言描述愿景（5-15 句话）
2. Claude 走强制反问 ≥ 1 轮：
   - "你为什么需要这个？市面上没有同类产品吗？"
   - "什么场景下会用？什么场景下不会用？"
   - "如果不做，用户怎么解决同一问题？"
   - "如果只能做 1 件事，是哪件？"
3. 反问后起草 ≤ 5 条顶层意图，填入 `docs/01-vision.md`
4. 用户审 / 改 / 砍 / 合并，定稿

**顶层意图格式**（不是功能清单，是可证伪的成功假设）：

| ❌ 功能式 | ✅ 意图式 |
|----------|----------|
| "做一个 AI 写小说的 SaaS" | "作者能从 0 写完一本 ≥ 10 万字的完整小说，且质量稳定" |
| "支持多个 LLM 厂商" | "作者切换 LLM 不需重新训练使用习惯，且成本可观测" |

**完成判定**：≤ 5 条顶层意图 + 每条都能用日志 / 指标 / 数据库查询验证。

**输出文件**：`docs/01-vision.md`（从 assets/starter-templates/docs/01-vision.md 模板创建）

### Phase 1 · 契约骨架

**Claude 角色**：起草者。**必须传统范式——人类主笔、AI 起草框架、人类填理由。**

**输出文件清单**（全部从 assets/starter-templates/ 拷贝模板后填入）：

1. **`CLAUDE.md`**：项目纪律权威源。含硬约束（每条指向 ADR）、协作模式、技术栈、规划范式
2. **`EXECUTION.md`**：续跑剧本。新会话第一件事读这个
3. **`docs/02-architecture.md`**：架构骨架（≤ 1 屏，≤ 5 个核心组件）
4. **`docs/03-data-model.md`**：数据模型第一版。先让用户按选择指南选变体（关系型 / 文档型 / 纯前端），再填
5. **`docs/execution/DECISIONS.md`**：≥ 5 条初始 ADR。每个不可逆决策必有一条
6. **`docs/execution/AGENT-PROMPTS.md`**：子代理 prompt 模板（8 个模板，选需要的保留）

**工作流**：
1. 逐一询问技术栈选型（语言、框架、数据库、前端等），每项列 2-3 个选项让用户拍板
2. 每个不可逆决策写入 DECISIONS.md（D-001, D-002, ...）
3. 起草架构图（ASCII 或 Mermaid）
4. 填 CLAUDE.md 硬约束段——每条指向 ADR 编号
5. 用户逐文件审定

**完成判定**：
- CLAUDE.md 已含项目硬约束
- DECISIONS.md ≥ 5 条 ADR
- 任何新会话读 CLAUDE.md + EXECUTION.md 能 5 分钟入门

### Phase 2 · MVP 路线图 + 契约网络

**前提**：Phase 1 全部定稿。

**工作流**：
1. 拆 ≤ 10 个 Batch，写入 `docs/execution/PROGRESS.md`
2. 每个 Batch 创建 `docs/execution/BATCH-XX-<name>.md`（从 BATCH-template.md 模板）
3. 按依赖顺序执行 Batch，每个 Batch：
   - 用户审任务书 → 派 Agent Team 实施 → 跑验收命令 → 人类验收 → 更新 PROGRESS.md
4. 测试 / 类型 / linter 配置与代码同步建（Phase 1 就配好工具，每 Batch 跑）

**Batch 拆分原则**：
- ≤ 10 个 Batch
- 每个 Batch 验收的是"哪条顶层意图变得可 e2e 验证"，不是功能列表
- 严格依赖顺序

**INTENT-GRAPH 此时不启用**——它是 Phase 3 工具。

**完成判定**：
- 顶层意图至少 1 条 e2e 可验证
- CI 全绿（lint / type / test）
- DECISIONS.md 已记录所有 Batch 内的非平凡决策

### Phase 3 · 意图驱动（永续）

**触发条件**：MVP 主链路真跑通，开始有人用。

**此时 INTENT-GRAPH 才发力**：
- 用户问"X 功能要不要做" → **不要直接开 Batch**，先看 INTENT-GRAPH 有无对应意图卡
- 有则评估证据，无则先写成意图卡
- 意图卡走状态机：`pending → probing → validated → implemented`（或被 `falsified`）

**意图卡字段强制清单**：
```
### Intent #NNN · <一句话标题>
**假设**：<我们相信什么>
**对应规格差异**：<docs/0X 引用，或"无对应——新意图">
**当前替代**：<不实施时用户怎么完成>
**证据触发条件**：<可观测条目>
**证伪条件**：<可观测条目>
**实施空间**：<极简 / 中量 / 重量>
**默认状态**：pending | probing
```

**关键纪律**：
- MVP 之前不写意图卡（会变空想）
- active 意图 ≤ 12 条（超出强制合并 / falsify）
- 证伪条件必须可观测（数据库 / 指标 / 日志能查出数字）
- 禁止主观词（"用户体验差"→ 改成"前端 RUM LCP > 2.5s 占比 > 20%"）
- user override 是紧急通道，不是默认（窗口内比例 > 50% 视为范式失败）

---

## 4. 场景 B：已有源码

比空项目复杂：代码里有未文档化的隐性约束，已有 PRD 可能脱节，团队有既定工作流。

**入口提示**：如果用户在当前项目里说"接入 AI-native 工作流"、"整理现有代码库"、"给项目加测试网络"、"代码太乱想理一下"，直接进入场景 B Phase 0。

### Phase 0 · 考古

**Claude 角色**：考古者。**只列事实，不写"为什么"。**

**工作流**：
1. **扫描代码库**：派 `Explore` agent 扫描项目结构
   ```
   请扫描当前代码库，输出：
   - 目录结构与模块划分
   - 路由表 / API 端点清单
   - 核心领域模型（有哪些主要实体）
   - 依赖图（谁 import 谁）
   - 主要数据流（请求 → 处理 → 存储的路径）
   - 测试覆盖现状（哪些文件有测试，哪些没有）
   - 现有 CI / lint / type 配置情况
   只列事实，不要写"为什么这段代码存在"——那是人类的工作。
   ```
2. **创建 CODEBASE-MAP.md**：将扫描结果整理为 `docs/execution/CODEBASE-MAP.md`，每个核心模块一段，留出"人类批注"空位
3. **人类逐段标注**：把文件交给用户，逐段问：
   - "这个模块是核心业务还是辅助？"
   - "这段代码有没有历史原因不能动？（合规/兼容/政治）"
   - "哪些是已知技术债？哪些是死代码？"
4. **隐性约束入 ADR**：人类标注中每条"不能动的原因"写入 DECISIONS.md（如 D-001 "GDPR 整改：5层if嵌套不可简化"）

**输出**：`docs/execution/CODEBASE-MAP.md` + ≥ 5 条隐性约束 ADR

**关键风险**：AI 容易把"过度设计"当"待简化目标"。5 层 if 嵌套可能对应两年前的 GDPR 整改。**人类标注不可省。**

**完成判定**：
- CODEBASE-MAP.md 覆盖核心模块
- 每个核心模块有"为什么存在"批注
- DECISIONS.md 已记录 ≥ 5 条隐性约束

### Phase 1 · 反推意图

**目标**：从现有代码反推产品意图，发现"代码有但没意图"的僵尸功能和"意图有但代码弱"的产品弱点。

**工作流**：
1. 基于 CODEBASE-MAP，AI 起草反推意图清单（≤ 10 条），每条写：
   - "代码里看到什么"→ 推断"产品意图可能是什么"
   - 标注属于哪类：🟢 代码和意图对齐 / 🟡 代码有但没意图（疑似僵尸）/ 🔴 意图有但代码弱（产品弱点）
2. 用户逐条审：哪些是真意图、哪些是 AI 误读、哪些漏了
3. 定稿入 `docs/execution/INTENT-GRAPH.md`，**状态全部 pending**——证据通道等 Phase 2 测试网建好再启用

**两类关键产出**：
- **代码有但没意图**（🟡）→ "待证伪意图"——若长期无人维护无人调用 → 删
- **意图有但代码弱**（🔴）→ 普通意图卡，进入证据通道，等待真实使用信号

**完成判定**：≤ 10 条反推意图 + 每条有类型标注 + 状态全部 pending

### Phase 2 · 建机器可读契约网络

**这是最不能省的阶段。跳过会出大事故。**

**核心原则**：只补核心路径，不全量覆盖。已有项目追求 100% 覆盖率会无限期拖延。

**工具阵**：mypy / pyright / TypeScript strict | pytest / vitest | importlinter / dependency-cruiser | bandit / semgrep

**工作流**（按顺序，不要并行跳步）：
1. **先配工具后补测试**：
   - 加 lint 配置（如 ruff/pylint/eslint），先设宽松规则只检查核心路径
   - 加 type checker 配置（mypy/pyright），先设 `--strict=false` 逐步收紧
   - 配 pre-commit hook，只跑 lint 先不跑 test
2. **补核心路径测试**（并行度 2-3）：
   - 识别核心路径（请求入口 → 业务处理 → 数据持久化的主链路）
   - 从最外层 API 测试（e2e/integration）开始写，往内层补
   - 不追求覆盖率数字，追求"核心路径有基线测试"
3. **起草追溯 ADR**：
   - 跑 `git log --since="2 years ago" --oneline` 找重大提交
   - AI 根据 git log 起草 ADR 草稿（每条 ≤ 3 句话）
   - **人类审定后**入 DECISIONS.md
4. **收紧质量门**：逐步提高 lint 规则严格度、type checker 严格度、测试覆盖率阈值

**完成判定**：
- 核心路径测试覆盖率 ≥ 60%
- CI 全绿（lint / type / test）
- importlinter（或等价工具）能跑
- DECISIONS.md ≥ 10 条追溯 ADR

### Phase 3 · 增量挂意图（永续）

**触发条件**：Phase 2 核心路径测试网建好。

**规则**：
- 所有**新 feature / 改动**强制挂意图卡（走 INTENT-GRAPH 状态机）
- **老代码维持现状**——直到有人触碰时才反向补意图
- 意图图谱会逐步覆盖代码库
- Agent Team 并行度低（2-3 + 互相 review）
- 每次改动派独立 agent 做反向核验——已有项目里"看起来对实际错"的概率高得多
- 每个 PR 必须报告"动了哪些已有约束"——避免无声破坏隐性合同

**意图卡写法、状态机、纪律同场景 A Phase 3。**

---

## 5. Agent Team 使用规范

### 何时开 Agent Team

**应该开**：跨层实现、可并行的独立工作（后端+前端+测试）、多步骤项目（研究→规划→编码→校验）、≥ 2 个独立子任务

**不应该开**：单文件小改、纯查询/探索、Phase 0/Phase 1、Phase 2 测试网络未建好的已有项目

### 并行度选择

| 项目类型 | 推荐并行度 | 理由 |
|---------|-----------|------|
| 空项目 Phase 2 主链路 | 3-5 | 有测试网兜底 |
| 空项目 Phase 3 实施意图卡 | 3-5 | 同上 |
| 空项目领域建模等独立子任务 | 5-9 | NovelForge 验证过 |
| 已有项目 Phase 2 补测试 | 2-3 | 没有兜底 |
| 已有项目 Phase 3 改老代码 | 1-2 + review agent | 隐性约束风险大 |

### 命名一致性（多 Agent 并行时最关键）

- 启动前先 grep 现有代码看本概念是否已有命名
- 复用模块导出的符号，不要起新名
- 跨 agent 共享术语必须先出现在 DECISIONS.md 中
- 合并前与同 Batch 其他 agent 的命名做 diff

---

## 6. 反模式速查（绝对不要）

| 反模式 | 后果 |
|--------|------|
| 让 AI 扫一下自动生成完整规格 | 充满想象的虚假 PRD |
| Phase 0 就启用 INTENT-GRAPH | 空想意图 |
| 跳过 ADR / DECISIONS.md | 隐性决策必死 |
| 没有契约网络就开 Agent Team 大改 | 看似对实际崩 |
| 把项目纪律写进 ~/.claude/projects/.../memory/ | 拷贝丢失 |
| 让单一 Agent 跨多 Batch 上下文连续工作 | 上下文爆炸 |
| 不验收就 commit AI 产出 | "AI 报告全绿"≠"真的对" |
| 证伪条件用主观词（"用户体验差"） | 永远证伪不掉 |
| Phase 0/1 纯意图驱动 | AI 没素材可推，写出空想 |
| 已有项目跳过 Phase 2 直接大改 | 隐性约束被破坏，且无回滚锚点 |

---

## 7. 模板文件使用

所有模板位于 `assets/starter-templates/`。创建项目文件时：

1. 读取对应模板文件
2. 替换所有 `<...>` 占位符为项目实际内容
3. 删掉模板中的"例："参考段落
4. 不要原样拷贝——根据项目实际情况裁剪

### 模板索引

| 模板文件 | 用途 | Phase |
|---------|------|-------|
| `CLAUDE.md` | 项目纪律权威源 | Phase 1 |
| `EXECUTION.md` | 续跑剧本 | Phase 1 |
| `docs/01-vision.md` | 顶层意图 | Phase 0 |
| `docs/02-architecture.md` | 架构骨架 | Phase 1 |
| `docs/03-data-model.md` | 数据模型入口+选择指南 | Phase 1 |
| `docs/03-data-model.relational.md` | 关系型数据库变体 | Phase 1 |
| `docs/03-data-model.document-nosql.md` | 文档型/NoSQL 变体 | Phase 1 |
| `docs/03-data-model.frontend-mobile.md` | 纯前端/Mobile 变体 | Phase 1 |
| `docs/execution/PROGRESS.md` | 进度看板 | Phase 2 |
| `docs/execution/DECISIONS.md` | 技术决策日志 | Phase 1 |
| `docs/execution/INTENT-GRAPH.md` | 意图图谱（Phase 3 启用） | Phase 3 |
| `docs/execution/AGENT-PROMPTS.md` | 子代理 prompt 模板 | Phase 1 |
| `docs/execution/BATCH-template.md` | 单 Batch 任务书 | Phase 2 |

---

## 8. 现实预期

**真实加速幅度**（不要吹 10x）：

- 空项目 MVP：30-50% 加速（vs 纯人类团队）
- 已有项目改造：10-30% 加速（隐性约束风险吃掉部分增益）
- 重复模式（CRUD、表单、API）：50-80% 加速
- 创新性工作（架构/算法/产品判断）：~0% 加速，AI 是辅助

**任何声称"AI 让你 10x 速度"的都没把 Phase 0/1/2 算进去。**

---

## 9. 执行入口

当用户说"继续"或被要求执行具体 Batch 时：

1. 读 CLAUDE.md（项目纪律）
2. 读 EXECUTION.md（续跑剧本）
3. 读 docs/execution/PROGRESS.md（找下一个 pending Batch）
4. 读 docs/execution/DECISIONS.md（已定决策）
5. 读对应 BATCH-XX-*.md（任务详情）
6. 执行 → 验收 → 更新 PROGRESS.md

**不要擅自跳批、不要重新规划路线图、不要改 docs/0X-*.md（默认只读）。**
