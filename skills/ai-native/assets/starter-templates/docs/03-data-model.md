# 数据模型（Phase 1 产出）· 入口与选择指南

> 本文件由 Phase 1 契约骨架阶段产出。**根据项目类型选一个变体填写，
> 删掉其他变体文件**。变体之间结构差异较大，统一模板会强行套用反而误导。

---

## 选择指南

按你项目的"主要持久化方式"选：

| 项目类型 | 用哪个变体 | 典型例子 |
|----------|-----------|----------|
| 后端有自己的关系型数据库（PG / MySQL / SQLite / SQL Server） | [`03-data-model.relational.md`](03-data-model.relational.md) | SaaS、企业系统、CRUD 类应用、多租户产品 |
| 后端用文档型 / KV / 图数据库（MongoDB / Firestore / DynamoDB / Redis 主存 / Neo4j） | [`03-data-model.document-nosql.md`](03-data-model.document-nosql.md) | 实时协作、日志聚合、内容管理、社交图谱 |
| 客户端项目，没有自己的服务端 / 数据全部走第三方 API（Supabase / Firebase / 自家 BaaS） | [`03-data-model.frontend-mobile.md`](03-data-model.frontend-mobile.md) | 纯前端 SPA、Mobile App、桌面应用、浏览器扩展 |

### 混合场景

- **关系型 + 缓存（Redis）+ 对象存储（S3 / MinIO）** → 用 `relational` 变体，缓存 / 对象存储作为辅助列在 §5
- **关系型 + 搜索引擎（Elasticsearch / Meilisearch）** → 用 `relational` 变体，搜索索引作为派生数据列在 §5
- **关系型 + 向量数据库（pgvector / Pinecone / Weaviate）** → 用 `relational` 变体，向量字段 / 索引列在 §3
- **多种持久化共存且没有明显主存** → 复制 2 个变体并改名（如 `03-data-model.users.relational.md` + `03-data-model.events.nosql.md`）

### 不在表格里的项目类型

- **数据管道 / ETL / 批处理** → 数据模型不是 schema，是 lineage。当前 starter 不覆盖，可参考 dbt / Dagster 文档自建模板
- **ML 训练 / 推理服务** → feature store 模型差异极大，单独记到 ADR 即可，不必走数据模型模板

---

## 通用纪律（任何变体都适用）

1. **第一版只覆盖核心实体**——不要写到 50 张表 / 50 个 collection，那是 LLM 帮你想象出来的
2. **多租户 / 用户隔离方式在第一版就要明确**——事后改是地狱
3. **不可逆决策**（数据库选型、隔离方式、加密方案）写到 02-architecture.md §4
4. 这份文件 Phase 2-3 期间增量更新（每加一个核心实体就更新），不必立 ADR；但**结构变更**（隔离方式 / 加密方案）必须立 ADR

---

## 如何使用本入口

1. 按上面表格选一个变体
2. 把对应的变体文件改名为 `03-data-model.md`（覆盖本文件）
3. 删掉其他变体文件
4. 按变体内说明填写
