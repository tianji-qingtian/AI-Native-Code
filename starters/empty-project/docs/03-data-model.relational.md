# 数据模型 · 关系型变体（PG / MySQL / SQLite / ...）

> 适用：后端有自己的关系型数据库的项目。
> 选择本变体后，把本文件改名为 `03-data-model.md` 并删掉其他变体。

---

## 1. 实体关系图

```
<在这里画 ER 图（ASCII 或 Mermaid图）。表达：
 - 核心实体
 - 关键外键关系（标 1:N / N:M）
 - 多租户 / 用户隔离字段（如 tenant_id / owner_id）
>

例：
  Tenant ─1:N─> Member
    │
    └─1:N─> Project ─1:N─> Document ─1:N─> Comment
              │
              └─1:N─> Tag
```

---

## 2. 核心表清单

| 表名 | 用途 | 关键字段 | 索引 | 备注 |
|------|------|----------|------|------|
| tenants | 租户 | id, slug, settings (JSON) | slug uniq | |
| members | 租户成员 | id, tenant_id, user_id, role | (tenant_id, user_id) uniq | |
| projects | 项目 | id, tenant_id, name, status | (tenant_id, status) | |
| ... | | | | |

> 第一版列 ≤ 15 张表覆盖核心；其余 Phase 2 实施时增量补。

---

## 3. 多租户 / 用户隔离方式

<选项：row-level（推荐 MVP） / schema-per-tenant / database-per-tenant>

例：
- **方案**：row-level，所有业务表带 `tenant_id` 列
- **强制点**：所有 Repository 查询带 `WHERE tenant_id = :tid`，由 `AuthMiddleware` 注入
- **理由 / ADR**：D-XXX
- **加密字段策略**：敏感字段（手机 / 身份证 / 邮箱）在应用层 AES-256-GCM 加密；ADR D-XXX

> 如果是单租户 / 无多租户：删掉本节，但需在 §1 ER 图标注"无租户隔离"。

---

## 4. 索引与性能基线

第一版必要索引：

- `<table>(<column>)` 唯一约束 / 普通索引 / 复合索引
- 全文搜索字段（如 GIN 索引 in PG）
- 时序字段（created_at desc）

> 不必完美，但**常见查询路径必须覆盖**。Phase 2 实施时如发现慢查询，在 DECISIONS.md 立 ADR 加新索引。

---

## 5. 辅助持久化

如果项目还用了缓存 / 对象存储 / 搜索引擎 / 向量库等，列在这里：

| 系统 | 用途 | 数据形态 | 一致性策略 |
|------|------|----------|-----------|
| Redis | 会话缓存 | `session:<token>` → JSON | TTL 7 天 |
| MinIO / S3 | 用户上传文件 | bucket: `tenant-<id>/...` | 永久 |
| pgvector | 文档语义搜索 | `documents.embedding vector(1536)` | 异步生成 |

---

## 6. 迁移策略

<在这里写：用什么工具做迁移、文件命名、CI 如何跑。>

例：
- Alembic + autogenerate
- 文件命名 `<utc_timestamp>_<short_desc>.py`
- 每个 PR 必须可向前向后双向迁移
- CI 跑 `alembic upgrade head` + `alembic downgrade -1` 双向验证

---

## 7. 待澄清 / 后续

<Phase 1 没决定、要 Phase 2 实施时再决定的字段 / 表。每个待澄清项有"什么时候必须决定"的截止 Batch。>

例：
- 大表（events / audit_log）分区策略 → 必须在 B5 之前决
- 加密字段 KMS 接入 → 必须在 B6 之前决

---

## 8. 提示

- ER 图复杂度按实际需要——10 个实体的 SaaS 1 屏装下，50 个实体的 ERP 拆多张子图
- 多租户字段在第一版就要明确——事后加 tenant_id 是地狱
- 索引可以在 Batch 实施时补，但**核心查询路径**必须先有索引
- 加密 / 审计 / 软删除 等横切关注点在第一版就要明确策略，不必每张表都列字段
