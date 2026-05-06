# AI-Native 开发工作流

> AI-native 开发的工程方法论——核心不是"让 AI 自主"，是让多 Agent 在共享契约下并行干活。

包含：

- **4-Phase 通用框架**：意图凝练 → 契约骨架 → 机器可读契约网络 → 意图驱动多 Agent
- **两种场景**：空项目（前瞻）/ 已有源码（考古）
- **反模式速查表**：踩过的坑都写在里面，别再踩
- **真实加速幅度**：30-50% / 10-30%，按场景给数，不吹 10x

适用读者：用 Claude Code（或同类 AI agent CLI）作为主力开发工具的工程师 / tech lead。
只想偶尔让 AI 改改代码的，这套方法论成本不回本。

> 如需通用方法论（适用于写作、设计、研究等任何领域），见 [`https://github.com/tianji-qingtian/AI-Native`](https://github.com/tianji-qingtian/AI-Native) 仓库。

## 文件一览

| 文件 | 内容 |
|------|------|
| [`AI时代的编程范式革命：从人类逻辑PRD到AI-native自主规划.md`](AI时代的编程范式革命：从人类逻辑PRD到AI-native自主规划.md) | 原始文章 |
| [`AI-Native-开发工作流方法论.md`](AI-Native-开发工作流方法论.md) | 方法论本体：4-Phase 框架 + 空项目 / 已有源码两种场景 + 反模式速查 |
| [`skills/ai-native-code/`](skills/ai-native-code/) | Claude Code skill：方法论的可执行版本，Claude 自动加载并引导 4-Phase 流程 |

---

## 快速使用

### 安装 skill（一次性）

要把这套方法论用到你自己的项目（无论新项目还是已有项目），先把 skill 装到用户级目录：

```bash
cp -r ./skills/ai-native-code ~/.claude/skills/ai-native-code
```

装完后，在任何项目里启动 `claude`，skill 都会自动加载。

### 我要开新项目

在**空目录**里启动 Claude Code，说：

```
我想做一个 <你的 idea>，用 AI-native 方式启动
```

Claude 会引导 Phase 0（意图凝练）→ Phase 1（契约骨架）→ Phase 2（MVP + 契约网络）→ Phase 3（意图驱动）。

### 我要把这套方法论用到已有项目

在**已有项目目录**里启动 Claude Code，说：

```
用 AI-native 方式接入这个项目
```

Claude 会引导 Phase 0（考古：扫描代码库、标注隐性约束）→ Phase 1（反推意图）→ Phase 2（建测试/lint/type 契约网络）→ Phase 3（增量挂意图卡）。

也可手动读 [`AI-Native-开发工作流方法论.md`](AI-Native-开发工作流方法论.md) §5。

---

## 关键原则

1. **AI 不是自主，是受约束的局部规划**——人类负责商业判断、架构选型、验收
2. **Phase 0/1 不能纯意图驱动**——AI 没素材可推
3. **机器可读契约（pytest / mypy / linter）是真相源**——docs 会腐烂，契约不会
4. **项目纪律入仓库（不入 memory）**——memory 绑机器，仓库走分发
5. **INTENT-GRAPH 是 Phase 3 工具**——MVP 之前不写意图卡

---

> **AI 是强大的执行者，但权力和责任必须留在人手里，而契约是连接两者的唯一可靠桥梁。**

## 反馈与版本

- v0.1（2026-04-28）：初版
- 每次试点出口（成立 / 失败）后更新

如果你在自己的项目里用了这套方法论，欢迎提交issue，作为方法论修订的素材。
