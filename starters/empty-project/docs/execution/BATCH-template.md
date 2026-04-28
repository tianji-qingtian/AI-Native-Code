# BATCH-XX · <Batch 名称>

> 单 Batch 任务书模板。每个 Batch 拷一份本文件改名为 `BATCH-XX-<short-name>.md`。
> 启动前必须由用户审定。完工后冻结，不再改。

---

## 1. 推进的顶层意图

<本 Batch 完成后，docs/01-vision.md 哪条顶层意图变得"e2e 可验证"？>

例：
> Intent #1 「作者能从 0 写完一本完整小说」——本 Batch 完成后，作者可以
> 创建小说 + 生成 1 章。

---

## 2. 验收意图（Definition of Done）

<本 Batch 完成的具体可验证标准。≥ 1 条 e2e 可验证。>

例：
- [ ] e2e 测试：调用 `POST /api/v1/novels` → 返回 novel_id；调用 `POST /api/v1/engine/chapters/generate` → 返回 chapter_id 且内容长度 ≥ 1000 字
- [ ] 单元测试覆盖率：domain ≥ 90%，application ≥ 75%
- [ ] mypy 0 err / lint 全绿 / importlinter 契约 KEPT

---

## 3. 依赖

- **前置 Batch**：B<X>, B<Y>（必须 completed）
- **不依赖**：B<Z>（可与其并行）

---

## 4. 子任务清单

| 子任务 | 文件 / 模块 | 子代理模板 | 并行 |
|--------|--------------|------------|------|
| <名称 1> | `<src>/<...>/*.py` | T2 | ✓ |
| <名称 2> | `<src>/<...>/*.py` | T2 | ✓ |
| <名称 3> | `<src>/<...>/*.py` | T3 | |

> **并行度**：本 Batch 推荐 N 个 agent 并行（理由：<...>）。

---

## 5. 硬约束（Batch 特定）

<不在 CLAUDE.md 通用硬约束里、但本 Batch 特别要注意的。>

例：
- 本 Batch 涉及 LLM 调用，必须走 LLMService（D-004），不允许直接 import openai
- 本 Batch 改动 schema，必须配套 Alembic 迁移
- ...

---

## 6. 验收命令

```bash
<跑测试 + lint + type 的具体命令清单。复制粘贴可运行。>

例：
cd <项目根>
uv run pytest -q tests/integration/<batch-area>
uv run mypy src
ruff check
lint-imports
```

---

## 7. 完成后必做

- [ ] 更新 `PROGRESS.md`：本 Batch 状态置 `completed` + 完成时间 + 关键产出 / 踩坑
- [ ] 必要时追加 `DECISIONS.md`（新决策 / 偏离规格的记录）
- [ ] 如果本 Batch 关联 INTENT-GRAPH 中某条意图，回写 §3 对账表把状态置 `implemented`（Phase 3 起）
- [ ] 报告本批摘要 + 下一批编号给用户

---

## 8. 风险与已知问题

<本 Batch 已知的风险 / 待澄清。如已记入 DECISIONS.md，引用 ADR 编号。>

例：
- LLM 厂商优先级未定，本 Batch 用 Mock；真 LLM 调通在 B<X>（D-XXX 待澄清）
- ...
