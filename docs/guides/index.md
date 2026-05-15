# 指南

本节是**任务式 Guides**：每篇解决一个具体的接入或配置问题，按推荐顺序串起一条完整接入路径。如果你刚开始接入，建议从 [接入准备](prerequisites.md) 顺序读下去。

## 接入路径

按顺序读完，即可完成从零到验证生效的全过程：

1. [接入准备](prerequisites.md) — AccessKey、Edge 节点地址、SDK 包准备
2. [初始化 SDK](initialize.md) — 在应用启动时完成初始化
3. [集成网络库](network-integration.md) — 把业务请求接入 SDK 本地代理（推荐 axhttp）
4. [启用 EdgeDoH](enable-edgedoh.md) — 防 DNS 劫持的基础配置
5. [配置 DNS 路由](dns-routing.md) — 白名单 / 豁免 / 缓存 / 容错的进阶配置
6. [验证接入](verification.md) — 通过控制台日志验证是否生效
7. [平台与框架适配](platforms.md) — 各平台 / 框架的 Demo 仓库索引

## 也别错过

完成基础接入后，常见的进阶需求与对应入口：

- **理解 SDK 工作机制** → [系统架构](../introduction/architecture.md) · [DNS 配置](../introduction/dns.md) · [控制面路由](../introduction/control-plane.md)
- **遇到错误排查** → [错误码](../errors.md) · [排障](../troubleshooting.md)
- **查参数完整定义** → [参考 / API 概念](../reference/index.md)

## 下一步

如果还没开始，从 [接入准备](prerequisites.md) 进入路径。
