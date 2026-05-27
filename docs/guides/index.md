# 指南

本节是**任务式 Guides**：从接入准备到验证生效，按顺序读完即可完成从零到上生产的全过程。每篇 Guide 都附 Android / iOS / Flutter 多平台代码示例。

!!! note "与白皮书 2 步流程的关系"
    [白皮书 / 客户端集成](../introduction/whitepaper.md#客户端集成quickstart) 把接入压缩成 2 步（init + 接入网络请求）——那是"最少必需"。本节路径在此之上加了**「启用 EdgeDoH」**（最常见的场景配置）和**「验证接入」**（确认生效），串起从零到上生产的完整路径。

## 开始前：SDK 的本地代理机制

SDK 启动后会在本地启动 **HTTP 代理**和 **SOCKS5 代理**服务。你需要把业务请求发到 SDK 的本地代理地址上，有两种方式：

- **使用 axhttp 扩展（零侵入）** —— 主流 HTTP 框架（OkHttp / Retrofit / URLSession / `http` 包等）的封装包，直接替换客户端实例，无需感知代理地址
- **上层应用自行设置代理** —— 应用主动获取本地代理地址，把流量转发到 SDK 的代理服务

axhttp 与核心 SDK 的分层关系见 [支持的平台与框架 / 核心 SDK 与 axhttp 分层关系](platforms.md#核心-sdk-与-axhttp-分层关系)。

## 接入路径

1. [接入准备](prerequisites.md) — AccessKey、Edge 节点地址、SDK 包准备
2. [初始化 SDK](initialize.md) — 在应用启动时完成初始化
3. [集成网络库](network-integration.md) — 把业务请求接入 SDK 本地代理（推荐 axhttp）
4. [启用 EdgeDoH](enable-edgedoh.md) — 防 DNS 劫持的基础配置
5. [验证接入](verification.md) — 通过控制台设备视图验证是否生效

## 也别错过

完成基础接入后，常见的进阶需求与对应入口：

- **理解 SDK 工作机制** → [简介 / 白皮书](../introduction/whitepaper.md)
- **遇到错误排查** → [参考 / 错误码](../reference/errors.md) · [资源 / 排障](../resources/troubleshooting.md)
- **DNS 高级调优**（白名单 / 豁免 / 缓存 / 容错 / 预解析） → [参考 / DNS 配置](../reference/dns-config.md)
- **查参数完整定义** → [参考 / 初始化参数](../reference/init-parameters.md) · [参考 / 代理配置](../reference/proxy-config.md)
- **找平台代码示例** → [支持的平台与框架](platforms.md)（Android / iOS / Flutter + 7 种 HTTP 客户端框架）
- **查术语** → [简介 / 术语表](../introduction/glossary.md)

## 下一步

从 [接入准备](prerequisites.md) 进入接入路径。
