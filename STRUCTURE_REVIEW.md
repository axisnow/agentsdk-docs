# AxisNow SDK 文档结构对齐评估报告

> 日期：2026-05-19
> 对标参考：Sentry / PostHog / Supabase 三家"多平台 SDK 文档"流派
> 评估范围：`/docs` 全部 25 篇 Markdown + `mkdocs.yml` 导航 + 白皮书源文档（Lark Wiki）

---

## 一、当前结构梳理

### 1.1 mkdocs nav 五段式

```
首页
├── 简介 (introduction/)
│   ├── 概览 overview.md
│   ├── Quickstart quickstart.md            ← 新版 4 步流程
│   ├── 白皮书 whitepaper.md                  ← 新增
│   └── 深入介绍
│       ├── 系统架构 architecture.md
│       ├── DNS 配置 dns.md
│       ├── 控制面路由 control-plane.md       ← 标注"设计文档"
│       └── 术语表 glossary.md
├── 指南 (guides/)
│   ├── 接入准备 / 初始化 SDK / 集成网络库
│   ├── 启用 EdgeDoH / 配置 DNS 路由
│   └── 验证接入 / 平台与框架适配
├── 参考 (reference/)
│   ├── API 概念（init-parameters / proxy-config）
│   ├── 各平台 API（Android / iOS / Flutter / Unity，全为占位）
│   ├── 错误码 errors.md                       ← 实际在 docs/ 顶级
│   └── 排障 troubleshooting.md                ← 实际在 docs/ 顶级
└── 更新日志 changelog.md
```

### 1.2 文档内容定位画像

| 文档 | 受众 | 抽象层 | 篇幅 | 状态 |
|---|---|---|---|---|
| `index.md` 首页 | 全员 | 入口 | 短 | ✅ 完整 |
| `introduction/overview.md` | 应用开发者 | SDK 视角 | 短 | ✅ 完整 |
| `introduction/quickstart.md` | 应用开发者 | 概念 4 步 | 中 | ✅ 新版完整 |
| `introduction/whitepaper.md` | 决策者 / 架构师 | 产品全景 | 长 | ✅ 新增完整 |
| `introduction/architecture.md` | 应用开发者 | SDK 工程视角 | 中 | ✅ 完整 |
| `introduction/dns.md` | 应用开发者 | 配置详解 | 中 | ✅ 完整 |
| `introduction/control-plane.md` | SDK 内部开发 | 设计文档（含代码） | 长 | ⚠️ 受众错位 |
| `glossary.md` | 全员 | 索引 | 短 | ✅ 完整 |
| `guides/*` (7 篇) | 应用开发者 | 任务流程 | 中 | ✅ 流程清晰 |
| `reference/init-parameters.md` | 应用开发者 | 概念参数 | 中 | ✅ 完整 |
| `reference/proxy-config.md` | 应用开发者 | 概念接口 | 短 | ⚠️ 自标"骨架" |
| `reference/platform-api/*` (4 篇) | 应用开发者 | 占位 | 极短 | 🚧 待生成 |
| `errors.md` | 应用开发者 | 速查 | 中 | ✅ 完整 |
| `troubleshooting.md` | 应用开发者 | 决策树 | 中 | ⚠️ 与 errors 内容重叠 |
| `changelog.md` | 全员 | 文档变更 | 极短 | ✅ 完整 |

---

## 二、对标分析：Sentry / PostHog / Supabase

三家代表的是"多平台 SDK 文档"的主流派，共同特征如下。

### 2.1 通用规律

| 规律 | Sentry | PostHog | Supabase |
|---|---|---|---|
| **Platform-first 顶级入口** | `Platforms` 顶级，下设每个平台一棵完整子树 | `Libraries` 顶级，每个 SDK 一棵子树 | `Reference` 下按 client 库分 JS/Dart/Swift/Python |
| **顶部全局平台选择器** | 有（影响整站代码示例） | 有 | 部分有 |
| **跨平台概念单独成章** | `Concepts` / `Product` | `Features` | `Guides` |
| **API Reference 独立** | 是（REST API） | 是 | 是（每个 client 一棵） |
| **示例项目独立成章** | `Sample Apps` | `Tutorials` | `Examples` |
| **Migration / 版本** | 每个平台内有 Migration 页 | 有 | 有 |
| **FAQ / Troubleshooting** | 每平台或全局都有 | 全局 | 全局 |
| **Changelog/Releases** | 与 SDK 版本绑定（不止文档） | 与产品绑定 | 与库版本绑定 |

### 2.2 关键模式

1. **以 "平台 × 任务" 为坐标系**——开发者第一次进站会问"我用 iOS，怎么做 X？"，因此最短路径是"先选平台 → 再选任务"。当前的 "先读简介 → 再到 Guides 看 tabs" 是另一种路径。
2. **跨平台一致内容做"概念页"，平台特有内容沉到平台子树**——避免同一段话在 3 个 tab 里写 3 次。
3. **白皮书/Why 类内容是营销/决策入口，不放在主流程**——通常归到 "Product" / "Introduction" 顶级，或直接在主站营销页。开发者文档里只放精简版"What is X"。

---

## 三、当前文档的核心问题（按优先级）

### 🔴 P0：品牌与术语不统一（必须先解决）

横跨多份文档的命名不一致：

| 命名 | 出现位置 |
|---|---|
| **AgentSDK** | `index.md`、`overview.md`、所有 guides、所有 reference、`glossary.md`、`changelog.md` |
| **AxisNow SDK** | `mkdocs.yml` site_name（最新已改） |
| **AxisNow 移动/原生应用增强套件** | `whitepaper.md` |
| **AXService** / **AXServiceConfig** | 代码示例（Android / iOS） |
| **AgentSDK.init / AgentConfig** | 白皮书 Kotlin 示例 |
| **AXHTTPService** / **AxClient** / **AxHttpClient** | network-integration / platforms |

**术语层面**：

- "Edge DoH" / "EdgeDoH" / "AED"（白皮书称 AED，guides/reference 称 EdgeDoH，glossary 写 Edge DoH）
- "Multi-Edge" / "数据面" / "Data Plane"（白皮书三种叫法都用了）
- "控制面节点 / 数据面节点" vs "Private Edge / Public Edge"——前者是工程视角，后者是产品视角，但文档里混着用
- "SecureProxy" / "Secure Proxy" / "加密隧道"

**为何 P0**：术语漂移会让开发者怀疑"是不是看错文档了"。在调整结构之前必须先冻结一套术语表。

---

### 🔴 P0：白皮书 / 概览 / 架构总览 三者重叠严重

| 文档 | 应负责的视角 | 当前实际内容 |
|---|---|---|
| `whitepaper.md`（新加） | **产品全景**（Cloud + Edge DoH + Multi-Edge + SDK） | ✅ 符合预期 |
| `overview.md` | **SDK 是什么**（开发者第一站） | 部分内容（核心能力矩阵）与白皮书重复 |
| `architecture.md` | **SDK 内部工作机制**（如何排障/深入） | 含整体工作流程图，与白皮书"请求链路"图重叠 |

三者的"核心能力"列表互不一致：

- `overview.md`：Context-aware Routing / Multi-Edge Proxy / 链路加密 / 设备资源信任
- `whitepaper.md`：客户端即第一道防线 / 按信任开放 / 端到端可观测 / 多云边缘灵活组合（4 设计理念）+ 5 能力维度矩阵
- `architecture.md`：DNS 模块 / Proxy 模块 / 控制面节点 / 数据面节点 / AxisNow Cloud

**问题**：开发者读完三份得不到一致的心智模型。

---

### 🟠 P1：`control-plane.md` 受众错位

- 头部标注 **"受众：SDK 开发者、各平台 Wrapper 开发者"，"状态：设计文档"**
- 内容包含具体 Go 代码引用（`pkg/sdk/sdk.go`、`net.ParseIP()`、`routing.StartPreloadLoop()` 等）
- 当前位置在 "简介 / 深入介绍" 下，与应用开发者的"概览/Quickstart"并列

**问题**：把内部设计文档暴露在应用开发者第一层，既污染信息密度，也可能泄露不希望对外的实现细节。

---

### 🟠 P1：导航不是 Platform-first

对标 Sentry/PostHog/Supabase：

- 当前路径：`Guides → 平台与框架适配 → 拿到 Demo 链接 → 跳出站点到 GitHub`
- 主流路径：`顶级 Platforms → 选 Android → 该平台一整棵 Quickstart/Install/Config/API/Troubleshoot`

**当前"平台与框架适配"页其实只是一张外链表**——把它当成"平台入口"是不够的。

---

### 🟡 P2：`troubleshooting.md` 与 `errors.md` 内容重复

`troubleshooting.md` 的 §1 把 `errors.md` 整张错误码表完整复制了一遍。这违反"单一来源"原则。

---

### 🟡 P2：`reference/platform-api/*` 4 个占位页

- Android / iOS / Flutter / Unity 全是 stub，每页都写 "建设中"
- nav 里以 4 个独立条目占位，会让目录看起来很"空"
- 对标 Sentry：自动生成的 API doc 通常是子站点（不出现在主 nav），主 nav 显示的是手写的 "Configuration / Usage" 等

---

### 🟡 P2：`glossary.md` 的位置奇怪

- 物理位置：`docs/glossary.md`（顶级）
- nav 位置：`简介 / 深入介绍 / 术语表`
- 对标做法：术语表通常是 **顶级资源区**（Resources / Reference 平级），方便从任何页面跳转

---

### 🟢 P3：缺少几个主流 SDK 文档常有的章节

| 缺失章节 | Sentry | PostHog | Supabase | 价值 |
|---|---|---|---|---|
| **Samples / 示例项目** | ✅ | ✅ | ✅ | 当前在 platforms.md 内"折叠"了，未单独成章 |
| **Migration Guide** | ✅ | ✅ | ✅ | 版本迭代必备 |
| **FAQ** | ✅ | ✅ | ⚠️ | 高频问题集中页，减少 Issue |
| **Security / 安全合规** | ✅ | ✅ | ✅ | API 密钥、PII、数据驻留 |
| **Performance / SDK 包大小** | ✅ | ✅ | ⚠️ | 移动 SDK 关心 |
| **Release Notes**（SDK 版本） | ✅ | ✅ | ✅ | 与文档 changelog 分开 |

---

### 🟢 P3：Quickstart 步骤数与白皮书对不上

- `quickstart.md`：**4 步**（接入准备 → 初始化 → 集成网络库 → 启用 EdgeDoH → 验证）
- `whitepaper.md` 部署集成：**2 步**（init + 替换客户端）
- `overview.md`：未分步

虽然受众不同，但同一站点内步骤数差异容易让人困惑。建议白皮书的 "Quickstart" 小节直接链向 quickstart.md，不再独立列代码。

---

## 四、调整建议

### 4.1 顶层导航重组（建议方案 A：Platform-first）

参照 Sentry/PostHog 的成熟模式：

```
首页 (index.md)
│
├── 简介 (introduction/)                  ← 精简，回答 "What/Why"
│   ├── 概览 overview.md                   ← 仅 SDK 视角，去掉与白皮书重叠
│   ├── Quickstart quickstart.md           ← 保留 4 步流程
│   └── 白皮书 whitepaper.md                ← 不动
│
├── 平台 (platforms/)                     ← 🆕 顶级 Platform-first 入口
│   ├── Android/
│   │   ├── 安装与初始化
│   │   ├── 集成 OkHttp / Retrofit / HttpsURLConnection
│   │   ├── 平台特有问题（ProGuard / 多进程）
│   │   └── Demo 仓库
│   ├── iOS/
│   │   ├── 安装与初始化
│   │   ├── 集成 URLSession (OC / Swift)
│   │   ├── 平台特有问题（ATS / Bitcode）
│   │   └── Demo 仓库
│   ├── Flutter/ (AxClient / AxHttpClient + Demo)
│   └── Unity/  (C# + Demo)
│
├── 指南 (guides/)                        ← 跨平台任务式
│   ├── 启用 EdgeDoH
│   ├── 配置 DNS 路由（缓存/容错/预解析）
│   ├── 选择代理模式（HTTP / SOCKS5 / TCP）
│   ├── 验证接入
│   └── 多边缘协同与灾备
│
├── 概念 (concepts/)                      ← 🆕 跨平台概念
│   ├── 系统架构（架构总览 → 移入）
│   ├── DNS 解析架构（dns.md → 移入）
│   ├── 调度域名 vs 业务域名（从白皮书抽出）
│   └── 多边缘协同（从白皮书抽出）
│
├── 参考 (reference/)                     ← API 类
│   ├── 初始化参数
│   ├── 代理配置
│   ├── 错误码
│   └── 自动生成 API（Android / iOS / Flutter / Unity 子站）
│
└── 资源 (resources/)                     ← 🆕 顶级资源区
    ├── 排障 troubleshooting.md
    ├── FAQ                                ← 🆕
    ├── 术语表 glossary.md                  ← 上移
    ├── 安全与合规                          ← 🆕
    ├── 文档更新日志 changelog.md
    └── SDK 版本说明（Release Notes）       ← 🆕
```

**关键变化**：

| 变化 | 原因 |
|---|---|
| 新增 `platforms/` 顶级 | 对齐 Sentry/PostHog 第一入口习惯，把 "平台与框架适配" 从外链页升级为完整子树 |
| 拆出 `concepts/` 顶级 | 把"深入介绍"组提升为顶级；移除"控制面路由"（见下） |
| `control-plane.md` 移到 `internal/` 或不发布 | 受众是 SDK 内部开发，不应在公开文档树中 |
| 新增 `resources/` 顶级 | 集中放术语表、FAQ、排障、安全、版本——访问频率分散但都属"非主线" |
| `errors.md` / `troubleshooting.md` 合并 | 错误码作为速查表保留，troubleshooting 收敛为"决策树 + 平台特有问题"两节 |

### 4.2 备选方案 B：维持当前 5 段但收紧

如果暂不想做 Platform-first 大改，最小调整：

```
首页
├── 简介                                  ← 拆掉"深入介绍"子组
│   ├── 概览 / Quickstart / 白皮书
├── 指南                                  ← 保留 7 篇任务式
├── 概念与架构 (concepts/)                 ← 🆕 把 "深入介绍" 提升为顶级
│   ├── 系统架构 / DNS / 多边缘协同
│   └── （control-plane.md 移出）
├── 参考                                  ← 不动
└── 资源                                   ← 🆕
    ├── 术语表 / 错误码 / 排障 / FAQ / 更新日志
```

适合的场景：发布前不想动太多文件路径，但要解决"深入介绍"嵌套过深 + glossary/errors 位置错乱。

### 4.3 关键页面职责对齐

| 页面 | 重新明确的职责 | 必要的内容裁剪 |
|---|---|---|
| `overview.md` | "SDK 是什么 + 和白皮书的关系 + 平台支持" | 删除核心能力矩阵（移到 whitepaper），保留产品边界 |
| `whitepaper.md` | 完整产品全景 + 决策视角 | 把 Kotlin Quickstart 删掉，只留链接到 quickstart.md |
| `architecture.md` | SDK **内部工程视角**（不重复白皮书的 Cloud / Multi-Edge 视角） | 整体工作流程图改为"以 SDK 为中心"的视角，避免和白皮书"请求链路"图重复 |
| `dns.md` | DNS 解析架构 + 路由配置（合并 `enable-edgedoh.md` 和 `dns-routing.md` 的概念部分） | 把和 guides 重复的"启用步骤"移走 |
| `troubleshooting.md` | 决策树 + 平台特有问题 + 最佳实践 | 删掉 §1 错误码表（指向 errors.md 即可） |
| `control-plane.md` | 内部设计文档，**移出公开文档树** | 或单独建 `internal/` 分组，加 `nav_exclude: true` |

### 4.4 术语统一（强制冻结一份术语表）

建议立刻产出一份"术语规范"，全站替换：

| 唯一术语 | 替换以下变体 |
|---|---|
| **AxisNow SDK** | AgentSDK / AxisNow 移动/原生应用增强套件 |
| **Edge DoH** | EdgeDoH / AED（保留 AED 仅作首次出现的别名） |
| **Multi-Edge / 数据面节点** | Data Plane / Multi-Edge Network（统一中文 + 英文括注） |
| **Private Edge / Public Edge** | AxisNow 自建边缘 / 第三方边缘 |
| **AXService / AXServiceConfig** | （选其一，白皮书的 `AgentSDK.init/AgentConfig` 应替换为正式 API 名） |

### 4.5 新增章节优先级

| 优先级 | 章节 | 投入 | 价值 |
|---|---|---|---|
| 🔴 High | Platform 顶级章节（4 个平台子树） | 中 | 对齐主流 SDK 文档，提升新用户上手速度 |
| 🟠 Mid | FAQ | 低 | 收敛常见 Issue，减少支持成本 |
| 🟠 Mid | Security & 合规 | 低 | 企业客户必看 |
| 🟢 Low | Migration Guide | 暂无版本迭代时不急 | 等 SDK 出新版本再补 |
| 🟢 Low | SDK Release Notes | 暂无对外发布时不急 | 接入 GitHub Releases 后自动同步 |

---

## 五、落地路径建议（分三波）

### 第一波（1-2 天内可完成，不涉及大改文件结构）

1. **冻结术语**：写一份 `STYLE_GUIDE.md`，全站 grep + replace（重点：AgentSDK → AxisNow SDK；AED → Edge DoH）
2. **`troubleshooting.md` 去重**：删掉 §1 错误码表，改为指向 `errors.md`
3. **`control-plane.md` 从 nav 移除**：保留文件但加 `nav: false`，或移到独立 `internal/`
4. **`overview.md` 去重**：核心能力矩阵改为单句 + 链向白皮书
5. **白皮书内的 Kotlin 示例**：删除或改为链向 `quickstart.md`

### 第二波（一周内，结构调整）

6. **拆掉"深入介绍"子组**：术语表上移到顶级"资源"，DNS / 架构提升到顶级"概念"或保留在简介
7. **新增 `resources/` 顶级**：归集 glossary / errors / troubleshooting / changelog
8. **新增 FAQ 占位页**：先放结构，从 Issue 和支持渠道汇总实际问题

### 第三波（视实际优先级，大改）

9. **新增 `platforms/` 顶级**：把"平台与框架适配"升级为 4 棵完整子树（Android/iOS/Flutter/Unity）
10. **每平台内补：安装、初始化、网络库集成、平台特有问题、Demo 链接**——可从现有 guides 内的 tabs 抽离生成
11. **Reference 内的 `platform-api/*` 占位页**：等自动生成接入后改为外链或子站点，目前可从 nav 隐藏

---

## 六、与白皮书的具体对齐点

读完《AxisNow 移动/原生应用 SDK 白皮书》后，建议把以下白皮书独有内容补回开发者文档：

| 白皮书的内容 | 应放到开发者文档的位置 | 当前状态 |
|---|---|---|
| 4 条设计理念 | `whitepaper.md` 内 | ✅ 已有 |
| 核心能力矩阵（5 维度） | `whitepaper.md` 内 | ✅ 已有 |
| 调度域名 vs 业务域名 | `concepts/` 新增页面 | ❌ 缺失（只在白皮书） |
| 基于上下文的节点隔离 | `concepts/` 新增页面 | ❌ 缺失 |
| 动态接入与故障切换（SDK 侧 + 控制面侧） | `architecture.md` 内 | ⚠️ 部分覆盖，需补 |
| 多边缘协同（地域/风险/灾备/成本 4 种策略） | `concepts/` 或 `guides/` | ❌ 缺失 |
| 去中心化机制 | `architecture.md` 内 | ❌ 缺失 |
| 1 小时部署目标 | `quickstart.md` / `index.md` | ⚠️ 未引用 |
| 7 大收益清单 | `whitepaper.md` 内即可，无需散播 | ✅ 已有 |

---

## 七、一句话总结

**当前 5 段 nav 框架合理，主要问题是**：

1. 品牌/术语跨文档不统一（必须先冻结一套词表）；
2. 白皮书/概览/架构总览三者职责未拆清；
3. 内部设计文档（`control-plane.md`）暴露在应用开发者首层；
4. 与 Sentry/PostHog 的 **Platform-first** 模式相比，平台入口被埋在 guides 下作为外链表，没有形成各平台独立子树。

**最值得做的一件事**：第一波清理（术语统一 + 重复内容去重 + 把内部设计文档移出公开树）能在 1-2 天内完成，立刻把整站观感拉齐。Platform-first 改造是较大工程，可放到第三波。
