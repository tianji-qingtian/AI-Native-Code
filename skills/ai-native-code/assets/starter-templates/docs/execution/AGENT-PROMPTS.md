# <项目名> · 子代理 Prompt 模板

适合 `Agent` 工具调用的 prompt 模板集合。每模板自包含、可空降一个"零上下文"Claude 执行。

---

## 通用公约（所有 spawn 都要带）

每个子代理 prompt 末尾追加：

```
## 必读前置（零上下文空降须知）
项目根：<绝对路径>
- 先读 `CLAUDE.md`（项目纪律权威源）
- 再读 `EXECUTION.md`（3 分钟了解纪律）
- 再读 `docs/execution/DECISIONS.md`（决策清单，不要违反）
- 再读 `docs/execution/BATCH-XX-*.md`（你负责的批次详情）
- 实施未规划于 docs/0X 的需求时，先看 `docs/execution/INTENT-GRAPH.md`（Phase 3 起）
- docs/0X-*.md 是规格，**默认只读**（编辑例外见 ADR）

硬约束：
<在这里同步 CLAUDE.md 的硬约束清单——必须每条都对应有 ADR>

多 Agent 并行命名一致性（避免同概念多个名字）：
- 启动前先 grep 现有代码看本概念是否已有命名
- 复用模块导出的符号；不要起新名
- 跨 agent 共享的术语必须先在 DECISIONS.md 出现过；没有就向用户确认
- 合并 PR 前与同 Batch 其他 agent 的命名做一次 diff，发现冲突立即对齐

完成后：
- 跑项目的 lint / type / test 全套（具体命令见 README 或 BATCH 文档）
- 报告本次新增 / 修改的文件列表 + 所有验收命令的结果
- 如本次实施关联 INTENT-GRAPH 中某条意图，回写 §3 对账表把状态置 `implemented`
```

> **使用方式**：派 agent 时把"通用公约 + 模板特定段 + 当前 Batch 任务清单"拼起来作为 prompt。
> 通用公约的"硬约束"段每次必须从 CLAUDE.md 同步，避免漂移。

---

## 模板 T1：Scaffold Agent（仓库骨架）

**用途**：在空仓库或新目录里初始化项目脚手架。
**子代理类型**：general-purpose
**通常并行度**：1（这是所有后续 Batch 的前置）

```
你负责本项目的仓库骨架 Batch。读 docs/execution/BATCH-XX-*.md，按里面的
文件清单和验收命令执行。

完工验收命令（必须全绿才算完）：
  <在这里列出具体命令，例如：
   docker compose up -d && docker compose ps
   <安装依赖命令>
   <初始测试命令>
   <build 命令>
   <linter / type checker 命令>
  >

遇陌生命令 / API 先用 context7 查；不要依赖训练数据。

[附通用公约……]
```

---

## 模板 T2：Module / Layer Implementer（单模块实现，可并行 N 实例）

**用途**：在已建好的架构骨架上实现单一模块 / 层 / 子域。
**子代理类型**：general-purpose
**通常并行度**：3-9（取决于模块独立性）

```
你负责本项目的模块 **<MODULE_NAME>**（候选：<在 BATCH 文档里列出>）。

读 docs/execution/BATCH-XX-*.md 找到你负责的模块段落，按"文件清单 + 必含
接口 / 类型 + 测试要求"落地代码到：
  <模块所属路径，例 src/<module>/ 或 packages/<module>/>
  <对应测试路径，例 tests/<module>/>

模块特定硬约束（不在通用公约里的）：
<列出本模块特定的限制，例如：
 - 不允许 import 哪些包 / 模块
 - 必须遵守的接口 / 协议（如 Protocol / interface / abstract class）
 - 命名约定 / 文件组织约定
 - 测试覆盖率要求
>

测试：
- <测试框架 + fixture 工具>
- <覆盖率阈值，如 ≥ 90%>
- <TDD or 测试后写，由 BATCH 文档决定>

验收：
  <pytest / vitest / jest / go test 等具体命令>
  <type checker 命令>

[附通用公约……]
```

> 适用范围（举例）：DDD 项目的 domain 子域、模块化单体的功能模块、
> 微服务项目的单服务、前端的功能 feature 模块、CLI 的子命令、library 的子包。

---

## 模板 T3：Persistence / Storage Layer（持久化层，可并行 N 实例）

**用途**：实现存储层（数据库 / 缓存 / 对象存储 / 队列等）的代码。
**子代理类型**：general-purpose
**通常并行度**：2-5

```
你负责本项目的持久化层组 **<GROUP>**。

读 docs/execution/BATCH-XX-*.md。落地：
  <持久化代码所属路径>
  <对应集成测试路径>

硬约束：
- <数据库访问异步 / 同步约定>
- <租户隔离 / 用户隔离的强制点>
- <ORM / driver 版本约定>
- <集成测试用真容器 / 内存 / mock 的约定>
- 实现 domain（或等价的接口层）里定义的 Protocol / interface

验收：
  <集成测试命令>
  <type 检查命令>

[附通用公约……]
```

---

## 模板 T4：API Surface（接口层，可并行 N 实例）

**用途**：实现 HTTP / gRPC / GraphQL / SSE / WebSocket 等对外接口。
**子代理类型**：general-purpose
**通常并行度**：3-5（按命名空间 / 资源拆分）

```
你负责本项目的接口层命名空间组 **<GROUP>**（group 1: <资源 X, Y>；
group 2: <资源 Z, W>；...）。

读 docs/execution/BATCH-XX-*.md + docs/02-architecture.md。落地：
  <接口代码所属路径>
  <schema / DTO 所属路径>
  <e2e 测试所属路径>

硬约束：
- <接口函数同步 / 异步约定>
- <schema 工具，例如 Pydantic / zod / TypeBox>
- 走统一的认证 / 鉴权中间件
- 错误响应用统一业务码
- **接口层不写业务逻辑**，统一 call application / use-case 层

验收：
  <e2e 测试命令>

[附通用公约……]
```

---

## 模板 T5：Frontend Feature（前端 feature / 页面 / 组件，可并行 N 实例）

**用途**：实现前端单一 feature / 页面 / 复合组件。
**子代理类型**：general-purpose
**通常并行度**：2-5

```
你负责本项目前端 feature **<FEATURE_NAME>**（候选：<在 BATCH 文档里列出>）。

读 docs/execution/BATCH-XX-*.md + 对应的设计 / spec 段落。落地：
  <feature 代码路径，例如 src/views/<feature>/, src/features/<feature>/>
  <共享组件路径>
  <store / state 路径，如有>
  <API client 路径>

硬约束：
- <框架版本 + 写法约定，如 React 18 函数组件 / Vue 3 SFC + script setup / Svelte 5 runes>
- <UI 库优先级，例如：先用 <UI 库> 现成组件；自绘只在不够用时>
- 所有 HTTP 调用走统一 client（如 src/api/http.ts）
- <状态管理库的写法约定>
- <跨 feature 隔离规则，例如不允许 features/A 直接 import features/B>
- 遇陌生 API 先用 context7 查

验收：
  <build 命令>
  <unit / e2e 测试命令>

[附通用公约……]
```

---

## 模板 T6：Review Agent（跨批次只读评估）

**用途**：Batch 完工后做独立 review。
**子代理类型**：Explore（只读，不改代码）

```
你是本项目的 review agent。刚完成了 Batch **<BATCH-ID>**，请独立评估，
报告 < 400 字：

1. 是否遵守了项目的架构 / 分层契约（跑 lint-imports / dependency-cruiser / 等）
2. 是否遵守了 CLAUDE.md 列出的硬约束（逐条 grep 检查）
3. 是否所有持久化查询都做了正确的租户 / 用户隔离（如适用）
4. 测试覆盖率是否达标
5. 本批次 DECISIONS.md 是否有新决策忘记登记
6. 本批次完成后是否回写了 INTENT-GRAPH §3 对账（如适用，Phase 3 起）

输出"⚠️ 风险清单"和"✅ 已通过清单"。**不要自己动手改代码**。

[附通用公约……]
```

---

## 模板 T7：Rescue Agent（卡死时）

**用途**：主 Claude 在某个 Batch 卡了 15+ 分钟时派出做独立诊断。
**子代理类型**：codex-rescue（如可用）/ 否则 general-purpose

```
本项目 Batch <ID> 遇到阻塞。
症状：<具体描述：错误信息 / 卡在哪一步 / 期望 vs 实际>
已尝试：<列出>
相关文件：<路径>
相关文档：docs/execution/BATCH-<ID>-*.md

请独立诊断根因并给出最小修复方案。**不要直接改代码**，输出诊断报告即可。
```

---

## 模板 T8：Migration / Refactor Agent（已有代码改造）

**用途**：在 Phase 3 / 已有项目中做带风险的重构 / 迁移。
**子代理类型**：general-purpose
**通常并行度**：1-2（带 review agent）

```
你负责本项目的重构任务：**<REFACTOR_NAME>**。

读 docs/execution/BATCH-XX-*.md + 现状代码（先 grep 摸清范围）。

硬约束：
- 重构前必须有覆盖现状行为的测试；如无，**先补测试再动手**
- 每一步必须可独立 commit + 可独立回滚
- 不允许同时改动接口签名和实现——分两步走
- 改动完成后必须跑全套 e2e，不允许只跑单测

如果改动范围超出 BATCH 文档预期 → 停下来报告，让用户决定是否扩大范围。

验收：
  <全套测试命令>
  <对现状用户没有可见行为变化的证明：e2e diff / canary 等>

[附通用公约……]
```

---

## 如何使用本模板

1. 替换所有 `<...>` 占位符为项目实际内容
2. 通用公约的"硬约束"段必须从 CLAUDE.md 同步，不要漂移
3. 派 agent 时把模板 + 当前 Batch 的具体任务拼起来作为 prompt
4. 项目特定模板（如 ML pipeline / Smart contract / DevOps）可在本文件追加 T9 / T10 ...
5. 不需要的模板可删——保留全部 8 个会让维护成本上升

## 模板适用场景速查

| 场景 | 用哪个模板 |
|------|-----------|
| 空仓库初始化 | T1 |
| 后端单一业务模块实现 | T2 |
| 数据库 / 缓存层实现 | T3 |
| HTTP / gRPC API 实现 | T4 |
| 前端 feature / 页面 | T5 |
| Batch 完工后独立 review | T6 |
| 卡住做独立诊断 | T7 |
| 已有代码重构 / 迁移 | T8 |
