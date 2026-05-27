# 更新日志

文档站本身的更新日志。SDK 发行版本的变更说明请参阅 [GitHub Releases](https://github.com/axisnow/sdk-docs/releases)（建立后）。

## 2026-05-27

- **删除 `quickstart.md`，并入指南首页（`guides/index.md`）** — quickstart 与各 Guide 内容重复且已开始漂移（验证步骤滞后于实际流程），合并为单一来源；其独有的"SDK 本地代理机制"概念与"白皮书 2 步流程关系"并入指南首页，首页 CTA 与简介页入口重定向到指南首页
- **验证接入流程改版** — 从"控制台请求日志 / 流量观察"改为控制台「端点 / 设备」设备视图 + 设备路由详情
- **EdgeDoH 示例代码对齐 SDK API 变更** — `addEdgeDohResolveDomain` / `addEdgeDohBypassDomain` 改为 `edgeDohResolveDomains()` / `edgeDohBypassDomains()`（Android varargs / iOS 属性赋值），见 [sdk PR #387](https://github.com/cyberandao/sdk/pull/387)
- **平台与框架移除 Unity 行**（暂无 Demo / 支持）

## 2026-05-15

- **文档结构重设计为 Anthropic / Supabase 风格**
    - 顶级 nav 调整为 5 段：首页 / 简介 / 指南 / 参考 / 更新日志
    - 新增**指南**（Guides）顶级，下设 7 篇任务式 Guide：接入准备、初始化 SDK、集成网络库、启用 EdgeDoH、配置 DNS 路由、验证接入、平台与框架适配
    - 原顶级"概念与架构"整组并入 **简介**，作为"深入介绍"子组
    - 拆分原 `getting-started/integration-guide.md` 为独立 `quickstart.md` + 7 个 Guide 页面，单一来源
    - 首页改为 hero + 4 主任务卡 + 底部站点资源卡（GitHub / 反馈）
    - 各 Guide 内代码示例改用 `pymdownx.tabbed` 切 Android Java / iOS Objective-C / Flutter Dart 三 tab
- 目录变化：`getting-started/` → `introduction/`；`concepts/` 内容并入 `introduction/`

## 待发布

- 各平台 API 自动生成文档接入（Javadoc / DocC / Dartdoc / DocFX）
- Demo 仓库占位 URL 替换为真实地址
