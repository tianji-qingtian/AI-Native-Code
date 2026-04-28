# 数据模型 · 文档型 / NoSQL 变体（MongoDB / Firestore / DynamoDB / ...）

> 适用：后端用文档型 / KV / 图数据库的项目。
> 选择本变体后，把本文件改名为 `03-data-model.md` 并删掉其他变体。

---

## 0. 关键差异（与关系型相比）

NoSQL 数据建模的主要范式差异：

- **没有外键约束** → 一致性靠应用层 / 事务
- **嵌套优于 join** → 强相关数据嵌入同一文档；弱相关数据用 reference + 多次读
- **查询模式驱动 schema** → 先列出所有查询场景，再决定文档结构（与关系型"先建模再查询"相反）
- **去规范化** → 同一字段允许冗余在多个文档（写多读少 vs 读多写少的取舍）

如果你不熟悉这套思维，**先去搞清楚再回来填本模板**——把 NoSQL 当 SQL 用是常见地雷。

---

## 1. 查询场景清单（先于 schema）

<在这里列出**所有**业务关键查询。schema 必须能回答这些查询，且不需要二级 join。>

例：
1. 「按 tenant_id + status 列出所有 projects，分页」
2. 「按 user_id 找出最近 30 天内的所有 events，按时间倒排」
3. 「按 project_id 找出 owner + 所有 collaborators 的资料」
4. 「全文搜索 documents.title + documents.content」
5. ...

> 这个清单是 NoSQL schema 的真正驱动者。少列一条查询，schema 可能就要重写。

---

## 2. 集合（Collection）/ 文档（Document）清单

| 集合名 | 用途 | 文档结构概要 | 索引 | 备注 |
|--------|------|--------------|------|------|
| tenants | 租户 | `{ _id, slug, settings, created_at }` | slug uniq | |
| projects | 项目 | `{ _id, tenant_id, name, owner: { user_id, name }, members: [...], status }` | (tenant_id, status), (tenant_id, created_at desc) | owner 嵌入；members 嵌入数组 |
| documents | 文档 | `{ _id, project_id, title, content, tags: [], updated_at }` | (project_id, updated_at desc), title text | content 大文本 |
| events | 事件流 | `{ _id, tenant_id, user_id, type, payload, ts }` | (user_id, ts desc), TTL 90d | 写多读少 |
| ... | | | | |

> 标注每个嵌入字段——是因为查询场景需要而嵌入的，避免无脑嵌入导致文档膨胀。

---

## 3. 嵌入 vs 引用决策表

每个跨实体关系都要明确：

| 关系 | 嵌入还是引用？ | 理由 |
|------|----------------|------|
| project → owner | 嵌入（部分字段） | 列表查询时一起返回，避免 N+1 |
| project → all members | 嵌入数组（≤ 50 人项目） | 同上；超过阈值则改引用 |
| project → documents | 引用 | documents 数量大且独立查询 |
| document → comments | 引用（独立 collection） | comments 写频高，嵌入会触发整文档锁 |

---

## 4. 多租户 / 用户隔离方式

<选项：tenant_id 字段（推荐） / database-per-tenant / collection-per-tenant>

例：
- **方案**：所有业务文档带 `tenant_id` 字段
- **强制点**：所有查询带 `tenant_id` 过滤；中间件在 `find` / `update` 拦截器自动注入
- **理由 / ADR**：D-XXX

> 如果是 Firestore：用 subcollection（`/tenants/{tid}/projects/{pid}`）天然隔离，写到 §1 路径 layout 段。

---

## 5. 一致性 / 事务策略

NoSQL 的硬骨头。明确每个写操作的一致性预期：

| 写操作 | 一致性级别 | 原因 |
|--------|-----------|------|
| 创建 user + 默认 tenant | 单文档原子 / 跨文档事务（MongoDB 4+） | 不可分割 |
| 更新 project + 写 audit_log | eventual（用 outbox / change stream） | 可容忍 |
| 删除 project（级联删除 documents） | 异步 worker | 慢但安全 |

---

## 6. 索引策略

NoSQL 索引昂贵且影响写性能：

- 列出每个核心查询对应的索引
- 复合索引顺序按查询 selectivity（高基数字段在前）
- TTL 索引（如 events 90 天过期）
- 全文 / 地理 / 向量索引按需

例：
```
db.events.createIndex({ user_id: 1, ts: -1 })
db.events.createIndex({ ts: 1 }, { expireAfterSeconds: 7776000 })  # 90d TTL
db.documents.createIndex({ title: "text", content: "text" })
```

---

## 7. 辅助持久化

如果项目还用了缓存 / 对象存储 / 搜索引擎 / 向量库等：

| 系统 | 用途 | 数据形态 |
|------|------|----------|
| Redis | 热点数据缓存 | `cache:<entity>:<id>` → JSON |
| ElasticSearch | 全文搜索 | document mirror |
| ... | | |

---

## 8. 迁移策略

NoSQL "迁移"通常是数据迁移脚本而非 schema 迁移：

- 工具：mongo-migrate / Firebase migration scripts / 自写 Python
- 命名：`<utc_ts>_<desc>.py`
- 大集合迁移：分批 + 幂等 + 可恢复
- 字段重命名 / 类型变更：先双写、再切读、再删旧字段（蓝绿）

---

## 9. 待澄清 / 后续

<Phase 1 没决定的、要 Phase 2 实施时再决定的字段。>

例：
- events 集合分片策略（按 tenant_id 还是按 ts？）→ 必须在 B5 之前决
- ...

---

## 10. 提示

- 第一版只覆盖**已知查询场景**对应的 collection——不要预判未来查询
- 嵌入数组要有上限（如 ≤ 50 项），超过迁移到独立集合
- 跨集合事务在 MongoDB 4.x+ 才有；在更老或 Firestore 等场景下要显式接受最终一致
- TTL / 软删除 / 版本号 等横切关注点在第一版就要明确
