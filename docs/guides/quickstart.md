# Quickstart

AxisNow SDK 接入分为 **4 个步骤**——本页用概念语言串一遍流程，**不涉及代码**。每步附"详细步骤"链接，跳到对应 Guide 后即可拿到完整代码示例（含 Android / iOS / Flutter 多平台 tabs）。

!!! note "与白皮书 2 步流程的关系"
    [白皮书 / 客户端集成](../introduction/whitepaper.md#客户端集成quickstart) 把接入压缩成 2 步（init + 接入网络请求）——那是"最少必需"。本页的 4 步在此之上加了**第 3 步「启用 EdgeDoH」**（最常见的场景配置）和**第 4 步「验证接入」**（确认生效），以串起从零到上生产的完整路径。

## SDK 的本地代理机制

SDK 启动后会在本地启动 **HTTP 代理**和 **SOCKS5 代理**服务。你需要把业务请求发到 SDK 的本地代理地址上，有两种方式：

- **使用 axhttp 扩展（零侵入）** —— 主流 HTTP 框架（OkHttp / Retrofit / URLSession / `http` 包等）的封装包，直接替换客户端实例，无需感知代理地址
- **上层应用自行设置代理** —— 应用主动获取本地代理地址，把流量转发到 SDK 的代理服务

axhttp 与核心 SDK 的分层关系见 [指南 / 平台与框架 / 核心 SDK 与 axhttp 分层关系](platforms.md#核心-sdk-与-axhttp-分层关系)。

## 开始前：接入准备

从控制台拿到以下两样东西：

- **AccessKey ID / Secret**：SDK 的身份凭证对，初始化时传入，运行期不变
- **Edge 节点地址列表**（`edgeNodes`）：**调度域名 + 预置 EIP** 的混填——SDK 通过它寻址 Edge DoH 服务（详见 [白皮书 / 调度域名](../introduction/whitepaper.md#调度域名)）。至少 1 条，推荐域名 + IP 混填以保证可用性

可选项：在控制台为**业务域名**（如 `api.app.com`）配置 **资源**（域名 + 源站 + 路由 + 安全策略，详见 [术语表](../introduction/glossary.md#5-凭证与配置)）与 **Edge DoH 路由规则**——前者决定哪些域名走 SDK 接管，后者决定 Edge DoH 基于上下文如何返回最优数据面节点。

[详细步骤 → 接入准备](prerequisites.md)

---

## 第 1 步：初始化 SDK

在应用启动**最早时机**调用 SDK 初始化接口（Android `Application.onCreate` / iOS `didFinishLaunchingWithOptions` / Flutter 应用入口），确保在发起任何网络请求之前完成。

初始化时传入 **AccessKey** 与 **Edge 节点地址列表**；可选传入 **DNS 配置** 等。

- 返回 `0` 即成功，负数为错误码（见 [错误码](../reference/errors.md)）
- 全局**仅允许成功初始化一次**，重复调用返回 `-102`
- 初始化失败后再次调用返回缓存错误，需**重启进程**重试

[详细步骤 → 初始化 SDK](initialize.md)

## 第 2 步：应用代理到网络库

初始化完成后，把业务请求接入 SDK 本地代理。SDK 对外提供两种代理模式：

| 模式 | 适用场景 |
|------|---------|
| **HTTP 代理** | OkHttp / URLSession / Dio 等 HTTP 客户端 |
| **SOCKS5 代理** | 非 HTTP 协议、通用代理需求 |

- 主流 HTTP 框架推荐用对应的 **axhttp 扩展**：零侵入，业务层只换客户端实例
- 不在 axhttp 覆盖范围 / 非 HTTP 协议 → 调用核心 SDK 的代理接口手动注入

[详细步骤 → 集成网络库](network-integration.md)

## 第 3 步：常见场景配置

最常见的场景是**启用 Edge DoH 防 DNS 劫持**（**推荐**）：

- **控制台侧**：确认 Edge DoH 路由规则已配置
- **SDK 侧**：把要保护的**业务域名**加入 `EdgeDohResolveDomains` 白名单（精确或 `*.suffix` 通配）

不启用 Edge DoH 也无妨——SDK 默认走系统 DNS，业务流量仍可经 Secure Proxy（Private Edge）或 Standard Proxy（Public Edge）转发。

[详细步骤 → 启用 EdgeDoH](enable-edgedoh.md) · [进阶 → 配置 DNS 路由（缓存 / 容错 / 预解析）](dns-routing.md)

## 第 4 步：验证接入

SDK 客户端**不对外打印调试日志**，移动应用沙盒内也无法直接跑 `curl`——**接入是否生效以控制台请求日志为准**。

1. 让应用发起若干次受 SDK 接管的网络请求
2. 进控制台**请求日志 / 流量观察**页，按 AccessKey 或时间范围过滤
3. 逐项确认 4 个维度：

    - **请求是否到达**：日志能否查到对应客户端的记录
    - **DNS 解析路径**：命中白名单的业务域名应标注为 Edge DoH 解析、解析返回的数据面节点符合预期
    - **端到端接入模式**：目标为 Private Edge 应走 **Secure Proxy**（加密隧道）；目标为 Public Edge 应走 **Standard Proxy**（HTTPS + JWT）
    - **回源情况**：状态码符合预期，4xx/5xx 异常对照源站日志排查

任一项不通过 → 对照 [错误码](../reference/errors.md) 与 [排障](../resources/troubleshooting.md) 定位问题。

[详细步骤 → 验证接入](verification.md)

---

## 找你的平台代码示例

各平台 Demo 仓库（Android Java / iOS Objective-C / Flutter Dart + 7 种 HTTP 客户端框架）见 [平台与框架](platforms.md)。

## 下一步

- **完整接入路径** → [指南](index.md)（7 篇任务式 Guides 串起从接入准备到验证全流程）
- **理解 SDK 工作机制** → [白皮书](../introduction/whitepaper.md)
- **查参数 / API** → [参考 / 初始化参数](../reference/init-parameters.md) · [参考 / 代理配置](../reference/proxy-config.md)
