# 概览

AgentSDK 是一款面向移动/原生应用的**应用网络接入 SDK**。它通过在客户端引入 **EdgeDoH 调度** 与 **多边缘代理转发**，让应用能够基于设备与网络上下文，在客户端侧实时选择最优接入路径，并将请求路由至公有 CDN 或自建边缘节点，从而获得更高的访问性能、可用性，以及更强的链路安全防护。

## 核心能力

- **Context-aware Routing（DoH 调度）**：基于设备、网络、风险等多维上下文，由 Edge DoH 动态返回最优节点地址，支持差异化分流、灰度发布、风险隔离等调度策略。
- **Multi-Edge Proxy（多边缘转发）**：统一的本地 Proxy 层将业务流量转发至多边缘数据平面（自建 AxisNow Edge 与公有 CDN 混部），实现多边缘协同与高可用访问。
- **链路加密 + 流量签名**：AxisNow 节点自动启用 TLS 隧道与签名认证；非 AxisNow 节点 HTTP/S 自动加签、TCP 透明转发。
- **设备/资源信任校验**：结合设备指纹、网络环境、风险评级，对终端请求做可信判定，拦截风险设备、仿冒应用等异常访问。

完整工作流程图与术语定义参见 [系统架构](architecture.md) 与 [术语表](../glossary.md)。

## 产品边界

为避免定位误解，以下能力**不在 SDK 提供范围**内：

- **不是公共递归 DoH** —— 面向特定应用的接入方，不是通用 DNS 服务
- **不是 VPN / 全局代理 / 系统级协议栈劫持** —— 只接管应用自身发起的网络请求
- **不是终端安全套件** —— 聚焦网络访问层的安全与可用性，不扩展为终端防护方案

## 支持的平台与框架

- **平台 SDK**：Android (Java) · iOS (Objective-C) · Flutter (Dart) · Unity (C#)
- **HTTP 客户端扩展（axhttp）**：Android × {OkHttp / Retrofit / HttpsURLConnection} × {Java / Kotlin}、iOS × URLSession × {Objective-C / Swift}
- **Flutter** 插件内置 `AxClient` / `AxHttpClient`

每个组合的最低版本要求、Demo 仓库链接见 [平台与框架适配](../guides/platforms.md)。

## 核心 SDK 与 axhttp 的分层关系

```
┌──────────────────────────────────────────────┐
│            应用层（App Code）                  │
├──────────────────────────────────────────────┤
│   axhttp 扩展（可选）                          │
│   OkHttp / Retrofit / URLSession / AxClient   │
├──────────────────────────────────────────────┤
│   核心 SDK（必选）                             │
│   本地代理 · 加密隧道 · DNS 解析 · 认证        │
└──────────────────────────────────────────────┘
```

- 所有平台都需要**先集成核心 SDK**
- 若应用使用主流 HTTP 框架（OkHttp / URLSession / http 包等），**推荐使用对应的 axhttp 扩展**以简化接入
- 若需要代理**非 HTTP 协议**或有自定义需求，直接使用核心 SDK 的代理接口即可

## 下一步

- [Quickstart](quickstart.md) — 3 步概念流程，5 分钟知道接入要做什么
- [指南](../guides/index.md) — 7 篇任务式 Guides，从接入准备到验证全流程
- [系统架构](architecture.md) — 深入了解 SDK 内部组件与工作机制
- [DNS 配置](dns.md) — EdgeDoH 路由规则与缓存调优
