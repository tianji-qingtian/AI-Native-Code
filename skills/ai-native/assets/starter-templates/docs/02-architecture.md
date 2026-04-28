# 架构骨架（Phase 1 产出）

> 本文件由 Phase 1 契约骨架阶段产出。这是给后续 Batch 提供稳定地基的契约——
> 一旦定稿，跨 Batch 改动需要立 ADR。

---

## 1. 核心组件图

```
<在这里画 ASCII 架构图（或用 Mermaid 画图）。
 至少要表达：
 - 主要组件 / 服务
 - 它们之间的数据流方向
 - 对外暴露的接口（HTTP / gRPC / SSE / 队列等），如果需要的话
>

例：
 ┌─────────────┐         ┌──────────────┐
 │   Frontend  │ ──HTTP─>│   FastAPI    │
 │   (Vue 3)   │ <─SSE──│  (interfaces)│
 └─────────────┘         └──────┬───────┘
                                │
                ┌───────────────┼───────────────┐
                ▼               ▼               ▼
        ┌──────────────┐ ┌──────────┐ ┌──────────────┐
        │ application  │ │  domain  │ │infrastructure│
        │  use-cases   │ │  models  │ │  pg / redis  │
        └──────────────┘ └──────────┘ └──────────────┘
                                ▲
                                │
                        importlinter 强制
                        (D-002)
```

---

## 2. 分层与依赖契约

<在这里写：分层规则 + 谁能 import 谁。最好用 importlinter / dependency-cruiser 等
工具强制，不是靠口头约定。>

例：
- DDD 四层：`interfaces → application → domain ← infrastructure`
- domain 层不允许 import 任何框架（FastAPI / SQLAlchemy / Celery）
- 由 `.importlinter` 强制（见 D-002）

---

## 3. 关键技术决策（指向 ADR）

| 决策点 | 选择 | ADR |
|--------|------|-----|
| 后端语言 | Python 3.12 | D-001 |
| Web 框架 | FastAPI | D-001 |
| ORM | SQLAlchemy 2 async | D-001 |
| 数据库 | PostgreSQL 16 | D-002 |
| 前端 | Vue 3.5 | D-003 |
| 认证 | JWT + 多租户 | D-007 |
| 异步任务 | Celery | D-005 |
| ... | ... | ... |

每条选择背后的"为什么"在 `./execution/DECISIONS.md` 对应 ADR。

---

## 4. 不可逆边界

<列出"现在定了之后改不动 / 改起来代价巨大"的决策。这一节是给未来的自己看的：
告诉自己哪些边界不要轻易碰。>

例：
- 数据库选型（迁移 50M 行表 = 数月工作）
- 多租户隔离方式（schema vs row-level，事后切换 = 全部 Repository 改）
- 认证流（JWT vs session，事后切换 = 全部 API 改）

---

## 5. 待澄清

<Phase 1 结束时还没拍板、留到 Phase 2 早期再决定的决策点。
每个待澄清项必须有"什么时候必须决定"的截止 Batch。>

例：
- LLM 厂商优先级 → 必须在 B4 之前决
- 前端 SSR vs SPA → 必须在 B14 之前决

---

## 6. 如何使用本模板

1. 每个技术决策必须有 ADR 编号，没有就先去写 ADR
2. 不可逆边界一定要明确写——避免 6 个月后的自己想不起来
3. 架构图复杂度按项目实际需要——简单项目几行就够，复杂项目可拆多张子图（前端 / 后端 / 数据流 / 部署拓扑各一张）
4. 这份文件 Phase 2-3 期间默认只读，例外按 D-XXX（编辑条件 ADR）走
