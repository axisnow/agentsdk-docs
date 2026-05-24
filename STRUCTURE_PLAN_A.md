# 方案 A：Platform-first 详细结构方案

> 配合 `STRUCTURE_REVIEW.md` 阅读
> 设计原则：跨平台接入步骤用 **一份带 tabs 的 Guides 说明**，`platforms/` 仅承担"平台导览"——一张支持矩阵 + 一份 Demo / API ref 索引。平台特有问题（ProGuard / ATS 等）留在 `resources/troubleshooting.md`，安装配置交给 Demo README。
>
> **2026-05-19 两次修订**：
>
> 1. 不同平台的接入步骤基本一致，没必要每个平台都重复完整教程。原"每平台 4 页子树"压扁为"每平台 1 页"，跨平台共性内容保留在 `guides/` 用 tabs 切代码。
> 2. **进一步收敛**：`platforms/{P}.md` 单页与 Demo README 高度重叠（安装片段），且平台特有问题已在 troubleshooting.md §4 存在。因此**取消 4 个平台单页**，只保留 `platforms/index.md` 作为单页导览，4 平台的入口信息（版本、依赖、Demo、API ref）以矩阵形式集中在一张表里。

---

## 一、最终目录树

```
docs/
│
├── index.md                              # 首页（hero + 4 张平台卡 + 4 张任务卡）
│
├── introduction/                         # 简介组（What / Why，决策与认知）
│   ├── index.md
│   ├── overview.md                       # 概览：SDK 是什么 + 平台支持矩阵
│   ├── quickstart.md                     # 跨平台 4 步概念流程（保留现版）
│   └── whitepaper.md                     # 完整产品白皮书
│
├── platforms/                            # 🔄 单页导览：1 张支持矩阵 + Demo / API ref 跳转
│   └── index.md                          # 唯一一页，4 平台并列展示
│
├── guides/                               # 跨平台任务式（用 tabs 切 4 平台代码）
│   ├── index.md
│   ├── prerequisites.md                  # 接入准备：AccessKey、Edge 地址、控制台资源
│   ├── initialize.md                     # 🔄 保留并扩展：初始化 SDK（Android/iOS/Flutter/Unity tabs）
│   ├── network-integration.md            # 🔄 保留并扩展：集成网络库（axhttp + 手动代理，多框架 tabs）
│   ├── enable-edgedoh.md                 # 启用 EdgeDoH 防 DNS 劫持（带 tabs）
│   ├── dns-routing.md                    # DNS 路由进阶（缓存/容错/预解析）
│   ├── proxy-modes.md                    # 🆕 选择代理模式（HTTP/SOCKS5/TCP）
│   ├── verification.md                   # 通过控制台日志验证接入
│   └── multi-edge.md                     # 🆕 多边缘协同与灾备配置（从白皮书抽）
│
├── concepts/                             # 🆕 跨平台概念（"X 的工作原理"）
│   ├── index.md
│   ├── architecture.md                   # 系统架构：SDK 内部工程视角
│   ├── dns-resolution.md                 # DNS 解析架构（原 introduction/dns.md）
│   ├── domain-types.md                   # 🆕 调度域名 vs 业务域名
│   ├── node-isolation.md                 # 🆕 基于上下文的节点隔离
│   ├── failover.md                       # 🆕 动态接入与故障切换
│   └── decentralization.md               # 🆕 去中心化机制
│
├── reference/                            # API 参考（"X 的精确定义"）
│   ├── index.md
│   ├── init-parameters.md                # 初始化参数（跨平台概念，含 DNS / 代理配置子节）
│   ├── proxy-config.md                   # 代理获取接口（跨平台概念）
│   ├── errors.md                         # 错误码完整定义（从顶级移入）
│   └── platform-api/                     # 自动生成的 API 文档（4 个子站点入口）
│       ├── index.md
│       ├── android.md                    # 跳转 Javadoc/Dokka 子站
│       ├── ios.md                        # 跳转 DocC/Jazzy 子站
│       ├── flutter.md                    # 跳转 Dartdoc 子站
│       └── unity.md                      # 跳转 DocFX 子站
│
└── resources/                            # 🆕 顶级资源区（"非主线但常用"）
    ├── index.md
    ├── troubleshooting.md                # 排障决策树 + 平台特有问题（不含错误码表）
    ├── faq.md                            # 🆕 高频问题集中页
    ├── glossary.md                       # 术语表（从顶级移入）
    ├── security.md                       # 🆕 安全与合规（AccessKey 管理 / PII / 数据驻留）
    ├── release-notes.md                  # 🆕 SDK 版本说明（链 GitHub Releases）
    └── changelog.md                      # 文档更新日志（从顶级移入）

# 不发布到公开站
internal/
└── control-plane-design.md               # 移出公开树（原 introduction/control-plane.md）
```

**文件总数对比**：

| 区块 | 当前 | 方案 A（二次修订版） | 净增 |
|---|---|---|---|
| introduction/ | 7 | 4 | -3（control-plane 移出，dns/architecture 移到 concepts） |
| platforms/ | 0 | 1（仅 index.md 单页） | +1 |
| guides/ | 8 | 9 | +1（删除 platforms.md，新增 proxy-modes / multi-edge） |
| concepts/ | 0 | 7 | +7（含从 introduction 移入 + 4 篇白皮书抽出） |
| reference/ | 7 | 8 | +1（errors 移入） |
| resources/ | 0 | 7 | +7 |
| 顶级零散 | 4 (errors/troubleshooting/glossary/changelog) | 0 | -4 |
| **总计** | **26** | **36** | **+10** |

净增 10 个文件，其中**新写内容约 8 篇**（含白皮书抽出的 4 个 concepts + faq/security/release-notes + platforms/index），其余是把现有顶级 / introduction 文件搬入子目录。

---

## 二、`mkdocs.yml` 完整 nav

```yaml
nav:
  - 首页: index.md

  - 简介:
      - introduction/index.md
      - 概览: introduction/overview.md
      - Quickstart: introduction/quickstart.md
      - 白皮书: introduction/whitepaper.md

  - 平台与框架: platforms/index.md

  - 指南:
      - guides/index.md
      - 接入准备: guides/prerequisites.md
      - 初始化 SDK: guides/initialize.md
      - 集成网络库: guides/network-integration.md
      - 启用 EdgeDoH: guides/enable-edgedoh.md
      - 配置 DNS 路由: guides/dns-routing.md
      - 选择代理模式: guides/proxy-modes.md
      - 验证接入: guides/verification.md
      - 多边缘协同: guides/multi-edge.md

  - 概念:
      - concepts/index.md
      - 系统架构: concepts/architecture.md
      - DNS 解析架构: concepts/dns-resolution.md
      - 调度域名与业务域名: concepts/domain-types.md
      - 节点隔离: concepts/node-isolation.md
      - 动态接入与故障切换: concepts/failover.md
      - 去中心化机制: concepts/decentralization.md

  - 参考:
      - reference/index.md
      - 初始化参数: reference/init-parameters.md
      - 代理配置: reference/proxy-config.md
      - 错误码: reference/errors.md
      - 各平台 API:
          - reference/platform-api/index.md
          - Android: reference/platform-api/android.md
          - iOS: reference/platform-api/ios.md
          - Flutter: reference/platform-api/flutter.md
          - Unity: reference/platform-api/unity.md

  - 资源:
      - resources/index.md
      - 排障: resources/troubleshooting.md
      - FAQ: resources/faq.md
      - 术语表: resources/glossary.md
      - 安全与合规: resources/security.md
      - SDK 版本说明: resources/release-notes.md
      - 文档更新日志: resources/changelog.md
```

---

## 三、文件迁移映射表

### 3.1 移动（保持内容，仅换路径）

| 当前路径 | 新路径 | 修改幅度 |
|---|---|---|
| `introduction/architecture.md` | `concepts/architecture.md` | 仅调整与白皮书重叠部分 |
| `introduction/dns.md` | `concepts/dns-resolution.md` | 无需大改 |
| `introduction/control-plane.md` | `internal/control-plane-design.md` | 移出公开 nav |
| `glossary.md` | `resources/glossary.md` | 仅校对术语 |
| `errors.md` | `reference/errors.md` | 无需大改 |
| `troubleshooting.md` | `resources/troubleshooting.md` | **删掉 §1 错误码表**，指向 reference/errors.md |
| `changelog.md` | `resources/changelog.md` | 无需大改 |
| `guides/platforms.md` | （删除） | 内容拆到各 platforms/*/index.md |

### 3.2 整合（4 平台信息合到 platforms/index.md 一张表）

| 当前来源 | 整合到 | 整合逻辑 |
|---|---|---|
| `guides/platforms.md` 全部 4 平台条目 | `platforms/index.md` | 平铺为支持矩阵 + axhttp 框架覆盖矩阵 + 4 平台 Demo 与 API ref 链接 |
| `troubleshooting.md` §4 平台特有问题 | **不动**（保留在 `resources/troubleshooting.md` §4） | platforms/index.md 末尾通过锚点链接跳过去 |
| 各平台安装片段（Gradle/CocoaPods/pubspec/Unity PM） | **不写**——交给 Demo README | 文档站只在矩阵里列"依赖引入方式"一栏的极简一行；详尽片段读 Demo README |

跨平台共性的"如何初始化 / 如何接入网络库 / 如何启用 EdgeDoH" **不拆**，继续在 `guides/initialize.md` / `guides/network-integration.md` / `guides/enable-edgedoh.md` 中用 tabs 切代码。

### 3.3 新建（白皮书内容下沉 + 缺失章节）

| 新文件 | 内容来源 | 一句话定位 |
|---|---|---|
| `platforms/index.md` | 整合 `guides/platforms.md` | 平台支持矩阵 + axhttp 框架覆盖矩阵 + Demo / API ref 跳转表（一页搞定） |
| `guides/proxy-modes.md` | 整合 reference/proxy-config.md + architecture §3 | 任务式："如何选 HTTP/SOCKS5/TCP" |
| `guides/multi-edge.md` | 白皮书"多边缘协同"章 | 任务式：地域/风险/灾备/成本四种策略 |
| `concepts/domain-types.md` | 白皮书"路由机制"章 | 调度域名 vs 业务域名的设计逻辑 |
| `concepts/node-isolation.md` | 白皮书"基于上下文的节点隔离"章 | 在 DNS 层隔离风险设备的工作原理 |
| `concepts/failover.md` | 白皮书"动态接入与故障切换"章 | SDK 侧 + 控制面侧双层容错 |
| `concepts/decentralization.md` | 白皮书"去中心化机制"章 | 多层兜底设计 |
| `resources/faq.md` | 新写 | 高频问题集中页 |
| `resources/security.md` | 新写 | AccessKey 管理、混淆建议、PII、数据驻留 |
| `resources/release-notes.md` | 链 GitHub Releases | SDK 版本变更（与文档 changelog 分开） |

---

## 四、`platforms/index.md` 单页内容骨架

一页搞定 4 平台的入口信息——**只放矩阵和跳转**，不写安装片段（交给 Demo README）、不写接入步骤（交给 Guides）、不写特有问题（指向 troubleshooting）。

```markdown
# 平台与框架

AxisNow SDK 支持 4 大端平台，跨平台接入步骤一致；本页是平台维度的快速索引。
完整接入流程请参见 [Quickstart](../introduction/quickstart.md) 与 [指南](../guides/index.md)。

## 1. 平台支持矩阵

| 平台 | 语言 | 最低版本 | 依赖引入（一句话） | Demo | API 参考 |
|---|---|---|---|---|---|
| Android | Java / Kotlin | API 21 (5.0) | `implementation files('libs/...aar')` | [🔗](https://github.com/.../android) | [🔗](../reference/platform-api/android.md) |
| iOS | Objective-C / Swift | iOS 12.0 | CocoaPods / SPM | [🔗](https://github.com/.../ios) | [🔗](../reference/platform-api/ios.md) |
| Flutter | Dart | Flutter 2.0+ | `axhttp: ^x.y.z` | [🔗](https://github.com/.../flutter) | [🔗](../reference/platform-api/flutter.md) |
| Unity | C# | Unity 2020 LTS+ | Package Manager / .unitypackage | [🔗](https://github.com/.../unity) | [🔗](../reference/platform-api/unity.md) |

> 完整的安装与构建说明见各 Demo 仓库的 README。

## 2. axhttp 框架覆盖

零侵入接入主流 HTTP 框架，业务层只换客户端实例。

| 平台 | OkHttp | Retrofit | HttpsURLConnection | URLSession | http | Dio |
|---|---|---|---|---|---|---|
| Android | ✅ Java/Kotlin | ✅ Java/Kotlin | ✅ Java | — | — | — |
| iOS | — | — | — | ✅ OC/Swift | — | — |
| Flutter | — | — | — | — | ✅ AxClient | ✅ AxHttpClient |
| Unity | — | — | — | — | — | — |

每个组合对应一个 Demo 子目录，统一收纳在 `agentsdk-demos` 仓库下。

## 3. 接入步骤

跨平台一致，按顺序：

1. [接入准备](../guides/prerequisites.md)
2. [初始化 SDK](../guides/initialize.md) ← 4 平台 tabs
3. [集成网络库](../guides/network-integration.md) ← 4 平台 tabs
4. [启用 EdgeDoH](../guides/enable-edgedoh.md)
5. [验证接入](../guides/verification.md)

## 4. 平台特有问题

每个平台在生产环境会遇到一些工程层面的细节问题（混淆、ATS、双端原生配置等）。
完整清单见 [排障 / §4 平台特有问题](../resources/troubleshooting.md#4-平台特有问题)，按平台分节：

- [Android](../resources/troubleshooting.md#41-android)（ProGuard / 多进程 / AAR / 权限）
- [iOS](../resources/troubleshooting.md#42-ios)（ATS / Bitcode / 必需系统库 / Deployment Target）
- [Flutter](../resources/troubleshooting.md#43-flutter)（PlatformException / MissingPluginException / 热重载）
- [Unity](../resources/troubleshooting.md#44-unity)（IL2CPP / 平台桥接）
```

**单页篇幅预期**：约 80-120 行，全部是矩阵、表格和外链；没有冗长的散文。维护成本：版本更新时只改对应矩阵单元格。

---

## 五、首页改造（index.md）

平台选择是 Platform-first 模式的关键 affordance。建议首页改成：

```markdown
# AxisNow SDK 开发者文档

[5 分钟跑通 →]  [查看白皮书 →]

## 选择你的平台

[Android]  [iOS]  [Flutter]  [Unity]
↓ 每张卡片直接进入对应 platforms/{platform}/get-started.md

## 任务索引

[启用 EdgeDoH]   [配置 DNS 路由]   [选择代理模式]   [验证接入]

## 站点资源

[GitHub]  [反馈]  [术语表]  [FAQ]
```

如果使用 Material for MkDocs，可以在顶部 nav 旁加 **Platform Switcher**（影响代码示例的 tab 默认值），是 Sentry/PostHog 的标志性体验。

---

## 六、实施顺序建议

### 阶段 1：纯重组（不写新内容，1 天）

只做文件移动 + nav 重写，不动现有内容：

1. 移动 `errors.md / troubleshooting.md / glossary.md / changelog.md` 到对应子目录
2. 移动 `introduction/architecture.md / dns.md` 到 `concepts/`
3. 把 `introduction/control-plane.md` 移到 `internal/`（或 nav 隐藏）
4. 删除 `guides/platforms.md`，新建 `platforms/index.md` 作为平台索引
5. 更新 `mkdocs.yml` nav 为方案 A 完整结构
6. 修复所有内部链接

**产出**：导航结构完成；除 `platforms/{P}/*` 外所有页面有内容；platforms 子树暂为 stub（"建设中"）。

### 阶段 2：填充 platforms/index.md + 扩展 guides（1-1.5 天）

1. **写 `platforms/index.md`**（约 100 行）：按 §4 骨架填 3 张表 + 跳转锚点：0.5 天
2. **补齐 `troubleshooting.md` §4**：当前 Unity 段缺失，需补 Unity 平台特有问题：0.25 天
3. **扩充跨平台 Guides 的 Unity tab**：把 `guides/initialize.md` / `network-integration.md` / `enable-edgedoh.md` 三篇的 tabs 补齐 Unity（当前只有 Android/iOS/Flutter）：0.5 天

**产出**：单页 platforms 导览可用 + Guides 跨 4 平台代码示例完整 + troubleshooting 4 平台齐全。

### 阶段 3：填充 concepts/ 与 resources/ 新章节（2-3 天）

把白皮书的 5 个独立章节抽出 + 写 FAQ / Security：

1. `concepts/domain-types.md`
2. `concepts/node-isolation.md`
3. `concepts/failover.md`
4. `concepts/decentralization.md`
5. `guides/multi-edge.md`
6. `resources/faq.md`（从 GitHub Issues / 支持渠道汇总，可分批补）
7. `resources/security.md`

**产出**：站点信息完整；白皮书与开发者文档形成对齐。

### 阶段 4：可选优化（视优先级）

- 顶部 Platform Switcher 控件
- 各平台自动生成 API doc 子站接入
- `resources/release-notes.md` 联动 GitHub Releases

---

## 七、与方案 B（最小改动）的取舍

| 维度 | 方案 A（二次修订版） | 方案 B |
|---|---|---|
| 工作量 | 中（约 4-6 工作日） | 小（约 1-2 工作日） |
| 文件数 | +10 | +2 |
| 用户体验对齐主流 SDK | ✅ 对齐（带 platforms 索引页） | ⚠️ 部分对齐 |
| 单平台开发者上手速度 | 快（platforms 单页矩阵 → Demo / guides） | 中（需要在 guides 内切 tab） |
| 内容重复风险 | **极低**（platforms 仅矩阵，安装交 Demo，特有问题交 troubleshooting） | 低 |
| 维护成本 | **低**（只需维护一张 platforms/index.md 矩阵 + guides tabs） | 中 |
| 适合的团队规模 | 1-2 人维护即可 | 1-2 人维护 |
| 适合的产品阶段 | 即将对外发布 | 内测 / Beta |

**建议**：

- 如果目标是面向**外部开发者**正式发布、希望对齐头部 SDK 产品形象 → 方案 A
- 如果当前主要是**内部 / 早期客户**使用，未来 3-6 个月不会有多平台同步迭代 → 先做方案 B，待用户数上来再升级到方案 A
- 也可以**先方案 B + 再升级方案 A**：方案 B 已经把术语统一、控制面文档移出、资源区独立——这些是方案 A 的前提条件，不会浪费

---

## 八、需要决策的几个问题

实施前建议先确认：

1. **品牌叫法最终选哪个？**——`AxisNow SDK` / `AgentSDK` / `AxisNow 移动应用 SDK`，决定全站统一替换的目标词
2. **白皮书是否同时进开发者站？**——目前是放在 `introduction/whitepaper.md`，也可独立成营销站，开发者站只保留精简版"What is X"
3. **Unity 是真实平台还是 placeholder？**——当前 unity.md 仅 7 行 stub，决定 `platforms/unity.md` 是写完整还是先放 stub
4. **`platform-api/*` 自动生成产物何时接入？**——决定 reference/platform-api/*.md 是保留 stub 还是先从 nav 隐藏
5. **是否启用 Material for MkDocs 的顶部 Platform Switcher？**——会影响所有代码示例的 tab 默认值联动
