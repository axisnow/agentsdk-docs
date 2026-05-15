# Quickstart

AgentSDK 接入分为 **4 个步骤**——本页用概念语言串一遍流程，**不涉及代码**。每步附"详细步骤"链接，跳到对应 Guide 后即可拿到完整代码示例（含 Android / iOS / Flutter 多平台 tabs）。

## SDK 的本地代理机制

SDK 启动后会在本地启动 **HTTP 代理**和 **SOCKS5 代理**服务。你需要把业务请求发到 SDK 的本地代理地址上，有两种方式：

- **使用 axhttp 扩展（零侵入）** —— 主流 HTTP 框架（OkHttp / Retrofit / URLSession / `http` 包等）的封装包，直接替换客户端实例，无需感知代理地址
- **上层应用自行设置代理** —— 应用主动获取本地代理地址，把流量转发到 SDK 的代理服务

axhttp 与核心 SDK 的分层关系见 [概览](overview.md#核心-sdk-与-axhttp-的分层关系)。

## 开始前：接入准备

从控制台拿到以下两样东西：

- **AccessKey ID / Secret**：SDK 的身份凭证对，初始化时传入，运行期不变
- **Edge 节点地址列表**：SDK 的第一跳入口（DoH 解析与加密隧道），至少 1 条，推荐域名 + IP 混填以保证可用性

可选项：在控制台为业务域名配置 **资源** 与 **DoH 路由规则**——前者决定哪些域名走 SecureProxy，后者决定 Edge DoH 调度策略。

[详细步骤 → 接入准备](../guides/prerequisites.md)

---

## 第 1 步：初始化 SDK

在应用启动**最早时机**调用 SDK 初始化接口（Android `Application.onCreate` / iOS `didFinishLaunchingWithOptions` / Flutter 应用入口），确保在发起任何网络请求之前完成。

初始化时传入 **AccessKey** 与 **Edge 节点地址列表**；可选传入 **DNS 配置**、**加密隧道开关** 等。

- 返回 `0` 即成功，负数为错误码（见 [错误码](../errors.md)）
- 全局**仅允许成功初始化一次**，重复调用返回 `-102`
- 初始化失败后再次调用返回缓存错误，需**重启进程**重试

[详细步骤 → 初始化 SDK](../guides/initialize.md)

## 第 2 步：应用代理到网络库

初始化完成后，把业务请求接入 SDK 本地代理。SDK 对外提供三种代理模式：

| 模式 | 适用场景 |
|------|---------|
| **HTTP 代理** | OkHttp / URLSession / Dio 等 HTTP 客户端 |
| **SOCKS5 代理** | 非 HTTP 协议、通用代理需求 |
| **TCP 代理** | Socket 编程、自定义协议、精细控制特定 host:port |

- 主流 HTTP 框架推荐用对应的 **axhttp 扩展**：零侵入，业务层只换客户端实例
- 不在 axhttp 覆盖范围 / 非 HTTP 协议 → 调用核心 SDK 的代理接口手动注入

[详细步骤 → 集成网络库](../guides/network-integration.md)

## 第 3 步：常见场景配置

最常见的场景是**启用 EdgeDoH 防 DNS 劫持**（**推荐**）：

- **控制台侧**：确认 DoH 路由规则已配置
- **SDK 侧**：把要保护的域名加入 `edgedoh_resolve_domains` 白名单（精确或 `*.suffix` 通配）

不启用 EdgeDoH 也无妨——SDK 默认走系统 DNS，业务流量仍可经 SecureProxy 加密转发。

[详细步骤 → 启用 EdgeDoH](../guides/enable-edgedoh.md) · [进阶 → 配置 DNS 路由（缓存 / 容错 / 预解析）](../guides/dns-routing.md)

## 第 4 步：验证接入

SDK 客户端**不对外打印调试日志**，移动应用沙盒内也无法直接跑 `curl`——**接入是否生效以控制台请求日志为准**。

1. 让应用发起若干次受 SDK 接管的网络请求
2. 进控制台**请求日志 / 流量观察**页，按 AccessKey 或时间范围过滤
3. 逐项确认 4 个维度：

    - **请求是否到达**：日志能否查到对应客户端的记录
    - **DNS 解析路径**：命中白名单的域名应标注为 EdgeDoH 解析、Edge 节点符合预期
    - **是否走 SecureProxy**：加密通道还是明文，与 **加密隧道开关** 一致
    - **回源情况**：状态码符合预期，4xx/5xx 异常对照源站日志排查

任一项不通过 → 对照 [错误码](../errors.md) 与 [排障](../troubleshooting.md) 定位问题。

[详细步骤 → 验证接入](../guides/verification.md)

---

## 找你的平台代码示例

各平台 Demo 仓库（Android Java / iOS Objective-C / Flutter Dart / Unity C# + 7 种 HTTP 客户端框架）见 [平台与框架适配](../guides/platforms.md)。

## 下一步

- **完整接入路径** → [指南](../guides/index.md)（7 篇任务式 Guides 串起从接入准备到验证全流程）
- **理解 SDK 工作机制** → [白皮书](whitepaper.md) · [系统架构](architecture.md) · [DNS 配置](dns.md) · [控制面路由](control-plane.md)
- **查参数 / API** → [参考 / 初始化参数](../reference/init-parameters.md) · [参考 / 代理配置](../reference/proxy-config.md)
