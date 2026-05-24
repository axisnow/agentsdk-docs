# 指南

本节是**任务式 Guides**：从 5 分钟概念跑通到完整接入与配置，按顺序读完即可完成从零到验证生效的全过程。

## 接入路径

1. [**Quickstart**](quickstart.md) — 5 分钟概念流程，先建立整体心智模型（无代码版）
2. [接入准备](prerequisites.md) — AccessKey、Edge 节点地址、SDK 包准备
3. [初始化 SDK](initialize.md) — 在应用启动时完成初始化
4. [集成网络库](network-integration.md) — 把业务请求接入 SDK 本地代理（推荐 axhttp）
5. [启用 EdgeDoH](enable-edgedoh.md) — 防 DNS 劫持的基础配置
6. [配置 DNS 路由](dns-routing.md) — 白名单 / 豁免 / 缓存 / 容错的进阶配置
7. [验证接入](verification.md) — 通过控制台日志验证是否生效

## 也别错过

完成基础接入后，常见的进阶需求与对应入口：

- **理解 SDK 工作机制** → [简介 / 白皮书](../introduction/whitepaper.md)
- **遇到错误排查** → [参考 / 错误码](../reference/errors.md) · [资源 / 排障](../resources/troubleshooting.md)
- **查参数完整定义** → [参考 / 初始化参数](../reference/init-parameters.md) · [参考 / 代理配置](../reference/proxy-config.md)
- **查术语** → [简介 / 术语表](../introduction/glossary.md)

## 下一步

如果还没开始，从 [Quickstart](quickstart.md) 进入路径。
