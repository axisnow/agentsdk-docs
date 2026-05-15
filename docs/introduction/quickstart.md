# Quickstart

AgentSDK 接入分**三步**——本页用概念语言串一遍流程，**不涉及代码**。每一步都附"详细步骤"链接，跳到对应 Guide 后即可拿到完整代码示例（含 Android / iOS / Flutter 多平台 tabs）。

## 第 1 步：准备凭据与节点

接入前需要从产品控制台拿到两样东西：

- **AccessKey**（ID + Secret 一对）：SDK 的身份凭证，运行期不变
- **Edge 节点地址列表**：SDK 的第一跳入口，至少 1 条，推荐域名 + IP 混填以保证可用性

[详细步骤 → 接入准备](../guides/prerequisites.md)

## 第 2 步：初始化 SDK

在应用启动**最早时机**（Android `Application.onCreate` / iOS `didFinishLaunchingWithOptions` / Flutter 入口）调用 `initialize`，传入上一步拿到的凭据与节点地址。

- 返回 `0` 即成功，负数为错误码（见 [错误码](../errors.md)）
- 全局**只能成功初始化一次**，重复调用返回 `-102`

[详细步骤 → 初始化 SDK](../guides/initialize.md)

## 第 3 步：接入网络库

把业务请求接入 SDK 提供的本地代理。三种方式选其一：

- **推荐：axhttp 扩展（零侵入）** — 替换 OkHttp / URLSession / `http.Client` 实例即可；自动接管所有请求
- **HTTP / SOCKS5 代理** — 自定义网络栈或非 axhttp 框架时手动注入本地代理地址
- **TCP 代理** — Socket 编程、自定义协议场景

[详细步骤 → 集成网络库](../guides/network-integration.md)

---

## 下一步

跑通基础接入后建议按以下路径深入：

- 验证接入是否生效 → [验证接入](../guides/verification.md)
- 启用 EdgeDoH 防 DNS 劫持 → [启用 EdgeDoH](../guides/enable-edgedoh.md)
- 配置 DNS 路由策略（白名单 / 缓存 / 容错） → [配置 DNS 路由](../guides/dns-routing.md)
- 理解 SDK 工作机制 → [系统架构](architecture.md) · [DNS 配置](dns.md) · [控制面路由](control-plane.md)

## 找你的平台代码示例

各平台 Demo 仓库链接见 [平台与框架适配](../guides/platforms.md)，Android (Java) / iOS (Objective-C) / Flutter (Dart) / Unity (C#) + 7 种 HTTP 客户端框架都有可运行示例。
