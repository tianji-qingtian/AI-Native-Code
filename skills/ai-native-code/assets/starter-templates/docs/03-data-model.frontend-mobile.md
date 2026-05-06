# 数据模型 · 前端 / Mobile 变体（无自有服务端 / 全部走第三方 API）

> 适用：纯前端 SPA、Mobile App、桌面应用、浏览器扩展。
> **没有自己的服务端，数据全部走第三方 API（Supabase / Firebase / 自家 BaaS / 公司中台 API）**。
>
> 选择本变体后，把本文件改名为 `03-data-model.md` 并删掉其他变体。

---

## 0. 关键差异

这类项目的"数据模型"不是 schema，是 4 件事：

1. **远端 API 契约**：调谁的接口、什么 schema、谁负责维护
2. **客户端状态结构**：内存里的 store / context 长什么样
3. **本地持久化**：哪些数据要存到 IndexedDB / AsyncStorage / localStorage
4. **同步策略**：何时拉、何时推、离线怎么办

如果你这部分都靠 LLM 想象，**回去先把 API 契约拿到再填本模板**——
没有契约谈不上数据模型。

---

## 1. 远端 API 契约

<列出本项目调的所有远端 API。每个 API 至少标：endpoint、请求 schema、响应 schema、错误码、版本管理方式、契约文档来源。>

| Endpoint | 用途 | 请求 | 响应 | 错误码 | 契约文档 |
|----------|------|------|------|--------|---------|
| `GET /api/v1/projects` | 列表 | query: page, status | `{ items: Project[], total }` | 401, 403, 500 | <OpenAPI URL> |
| `POST /api/v1/projects` | 创建 | body: CreateProjectInput | `Project` | 400, 401, 409 | 同上 |
| ... | | | | | |

**强烈建议**：用 OpenAPI / GraphQL schema 自动生成 TS 类型，**不要手写**。

```bash
# 例：从 OpenAPI 生成 TS 类型
npx openapi-typescript https://api.example.com/openapi.json -o src/api/types.ts
```

---

## 2. 客户端状态结构

按状态管理方案（Pinia / Redux / Zustand / Recoil / Riverpod / ...）描述顶层 store：

```typescript
// 例：Pinia 风格
interface AuthStore {
  user: User | null
  accessToken: string | null
  refreshToken: string | null
}

interface ProjectStore {
  list: Project[]            // 缓存的列表
  currentId: string | null   // 当前选中
  loading: boolean
  error: string | null
}

interface UIStore {
  theme: 'light' | 'dark' | 'auto'
  sidebarCollapsed: boolean
}
```

**纪律**：
- 服务端数据 vs UI 状态分开（服务端用 SWR / React Query / TanStack Query 之类管缓存，UI 状态另立 store）
- 不要把整个 API 响应原样存进 store——按 UI 实际需要做投影

---

## 3. 本地持久化

<列出哪些数据要存到本地、用什么机制、TTL 多久。>

| 数据 | 存储位置 | 加密？ | TTL | 失效策略 |
|------|----------|--------|-----|----------|
| accessToken / refreshToken | secure storage（Keychain / EncryptedSharedPreferences）/ httpOnly cookie | ✅ | refresh: 7d | 401 时清空 |
| 用户偏好（主题 / 语言） | localStorage / AsyncStorage | ❌ | 永久 | 用户主动改 |
| 草稿数据（离线编辑） | IndexedDB / SQLite | 视敏感度 | 直到同步成功 | 同步成功后清 |
| 服务端列表缓存 | React Query persist / SWR cache | ❌ | 5 分钟 | 时间 + invalidate |

**关键决策**：
- token 不要存 localStorage（XSS 风险）
- 大数据用 IndexedDB / SQLite，不要塞 localStorage
- iOS / Android 平台特定 secure storage 选型差异要明确

---

## 4. 同步策略 / 离线模式

如果项目需要离线可用：

| 操作 | 在线策略 | 离线策略 | 冲突解决 |
|------|----------|----------|----------|
| 读列表 | 走网络 + 缓存 | 用缓存 | — |
| 写操作（创建草稿） | 立即推 + 乐观更新 | 入本地队列 | 时间戳 last-write-wins |
| 写操作（关键数据） | 立即推 + 等响应 | 拒绝 / 提示 | — |

如果完全在线（不需要离线），**明确写"无离线支持"**——这本身是产品决策。

---

## 5. 数据流图

```
<画一张数据流图。表达：
 - 远端 API
 - 客户端缓存层（React Query / SWR / Apollo / ...）
 - Store / 全局状态
 - 本地持久化
 - UI 组件
 - 它们之间的读 / 写 / 同步方向
>

例：
  [Remote API] <───HTTP───> [TanStack Query Cache] ──→ [Component]
                                       │
                                       ▼
                                 [persist plugin]
                                       │
                                       ▼
                                 [IndexedDB]

  [Component] ─→ [Pinia Store: UI / Auth] ─→ [localStorage]
```

---

## 6. 错误处理 / 重试策略

| 场景 | 策略 |
|------|------|
| 401（token 过期） | 自动 refresh，失败则跳登录 |
| 403 | 提示"无权限"+ 不重试 |
| 网络错误 | 指数退避 3 次 |
| 5xx | 指数退避 2 次 + 提示 |
| 超时 | 用户可手动重试 |

---

## 7. 待澄清 / 后续

<Phase 1 没决定、要 Phase 2 实施时再决定。>

例：
- 是否支持多账号切换 → 必须在 B3 之前决
- 是否需要端到端加密（消息类）→ 必须在 B5 之前决

---

## 8. 提示

- API 契约是**远端团队的产物**，前端项目数据模型本质是**读取契约 + 客户端建模**
- 状态管理方案选好后不要轻易换（迁移成本高），写到 02-architecture §3
- 本地持久化的加密 / 失效策略在第一版就要明确——产品上线后改是地狱
- 离线 vs 在线是产品决策，不是技术决策——必须有明确答案，不能"看情况"
