# 术语表

本表与 [白皮书](whitepaper.md) 定义对齐。SDK 文档其他章节如出现不一致，以本表为准。

带 ⚠️ 标记的条目是 SDK 文档早期使用的术语，已被白皮书的精确定义取代——保留是为了兼容历史引用。

---

## 1. 体系组件

| 术语 | 含义 |
|------|------|
| **AxisNow Cloud（控制面）** | 中心化的策略编排、配置分发与全局观测中心。**不直接参与业务流量转发**。负责域名/源站/边缘节点管理、路由与安全策略下发、SDK 配置、节点健康与可观测数据汇聚、对 Data Plane 节点的拨测。SDK **不与之直连**，所有交互经 Edge DoH 中转。 |
| **Multi-Edge 网络** | SDK 请求进入业务系统前的边缘执行层，由 **Private Edge** + **Public Edge** 共同组成，承担流量接入、安全防护与回源转发。 |
| **Private Edge** | AxisNow 自建边缘，承担**调度面 + 数据面**双重职责：<br>① **调度面**——运行 Edge DoH 服务，做上下文路由决策与 SDK 接入点发现；<br>② **数据面**——TLS 加密链路接入、可信校验、WAF/Bot 防护、智能回源。 |
| **Public Edge** | 第三方公有 CDN（Cloudflare、AWS CloudFront 等）。**仅承担数据面**职责，**不运行 Edge DoH**——其流量调度完全由 Private Edge 上的 Edge DoH 决定。通过 JWT Token 校验实现访问控制。 |
| **Data Plane（数据面）** | Multi-Edge 网络中**承担流量转发**的层。是 Private Edge 数据面角色 + Public Edge 全部职责的合称。 |
| **AxisNow SDK Client** | 集成在移动 App / 桌面客户端 / IoT 设备中的轻量级 SDK。承担本地决策与可信接入：DNS 拦截、Proxy 转发、可信校验、动态选路、Edge 接入。 |
| **Edge DoH（AED）** | 基于 DNS-over-HTTPS 的解析服务，运行在**每个 Private Edge 节点**上。具备**分布式、高可用、基于上下文策略解析**三大特性。SDK 通过 Edge DoH 获取业务域名的最优数据面地址与控制面下发的配置。详见 [白皮书 / AxisNow Edge DoH（AED）](whitepaper.md#axisnow-edge-dohaed)。 |
| ⚠️ **控制面节点**（旧术语） | SDK 文档早期术语，**等价于 Private Edge**。**不要**与 AxisNow Cloud（白皮书中的"控制面"）混淆——两者是完全不同的实体。 |
| ⚠️ **数据面节点**（旧术语） | SDK 文档早期术语，泛指 Data Plane 中任意一个边缘节点（可能是 Private Edge 的数据面角色，也可能是 Public Edge）。 |
| **Edge 节点** | 对 Private Edge 与 Public Edge 的统称。在不区分两类边缘的语境里使用。 |

---

## 2. 域名分类

白皮书把 SDK 处理的域名严格分成两类，可用性诉求完全不同：

| 术语 | 含义 |
|------|------|
| **调度域名** | 让 SDK 找到 Edge DoH 服务的域名（如 `doh.acme.com`）。**先于任何业务请求被解析**，是系统启动前提，对极致可达性有诉求。在 SDK 初始化参数 `edgeAddresses` 中与预置 EIP 并列配置。 |
| **业务域名** | 应用实际访问的目标域名（如 `api.app.com`），由 SDK 接管 DNS 解析与流量转发。命中 `edgedoh_resolve_domains` 白名单的域名走 Edge DoH 解析，其他不走 Edge DoH。 |

---

## 3. 端到端接入模式

数据面节点接收 SDK 请求时，根据目标类型走不同的接入模式：

| 术语 | 含义 |
|------|------|
| **Secure Proxy 模式** | 目标为 **Private Edge** 时启用。SDK 与之建立**加密链路（TLS 隧道）**；Private Edge 终止隧道并完成请求解封装与鉴权。安全等级高，支持精细化访问控制。 |
| **Standard Proxy 模式** | 目标为 **Public Edge** 时启用。SDK 在标准 HTTPS 请求中**注入 JWT Token**；Public Edge 校验 Token 有效性，确认请求来自合法 SDK 客户端。侧重全球覆盖与标准加速。 |
| ⚠️ **SecureProxy**（旧术语） | SDK 文档早期的上位词，泛指"业务流量经 SDK 安全代理"。按白皮书规范应**拆为 Secure Proxy / Standard Proxy** 两种端到端模式。 |
| **本地代理** | SDK 在设备本地监听的 HTTP / SOCKS5 / TCP 端口，业务请求经本地代理转发到 Edge 节点。<br>**注**：本地代理类型（HTTP / SOCKS5 / TCP）与端到端接入模式（Secure / Standard Proxy）是**正交概念**——前者描述应用如何把流量交给 SDK，后者描述 SDK 如何与目标边缘建立连接。 |

---

## 4. DNS 与寻址

| 术语 | 含义 |
|------|------|
| **EIP** | Edge DoH 服务的对外入口 IP。SDK 持有的 EIP 来源有三类（详见白皮书 § AED 完整请求流程）：<br>① **预置 EIP**——App SDK 内置的兜底地址（`edgeAddresses` 中纯 IP 部分）<br>② **兜底解析 EIP**——通过预配置的兜底域名（公共 DoH / LocalDNS 解析）拿到<br>③ **策略分配 EIP**——Edge DoH 基于设备/网络上下文动态下发的最优 EIP（运行期主用入口） |
| **兜底域名** | `edgeAddresses` 中以域名形式预埋的部分（如 `doh.acme.com`）。通过公共 DoH 或 LocalDNS 解析得到 EIP，作为预置 EIP 失联时的二级兜底。对应上述"兜底解析 EIP"的获取路径。 |
| **EdgeDoH 白名单** | `edgedoh_resolve_domains` 中列出的**业务域名**（catch-all `*`、精确或 `*.suffix` 通配）走 Edge DoH 解析，未命中则不走 Edge DoH。配成 `["*"]` 即所有域名全走 Edge DoH。 |
| **EdgeDoH 豁免** | `edgedoh_bypass_domains` 中列出的业务域名一律不走 Edge DoH，**优先级高于白名单**。用于"`*.example.com` 全走 EdgeDoH 但 `login.example.com` 例外"这类场景。 |

---

## 5. 凭证与配置

| 术语 | 含义 |
|------|------|
| **AccessKey** | 控制台下发的 ID + Secret 身份凭证对。SDK 初始化时传入，用于 Edge DoH 签名认证与控制面消息验签。凭证缺失或非法时初始化返回错误码 `-112` / `-113` / `-114`，详见 [错误码](../reference/errors.md)。 |
| **资源** | 控制台为**业务域名**定义的代理对象，关联**源站、路由策略、安全策略**——是 SDK 文档对白皮书中"域名 + 源站 + 路由策略 + 安全策略"组合的统称。**未配资源的域名不走 SDK 代理**，按本地 DNS 直连源站。<br>**注**：白皮书未单独使用 "资源" 一词，而是分别讨论上述 4 项。本表沿用 SDK 文档的"资源"是便于实操层面引用控制台对象。 |
| **设备上下文** | SDK 上报给 Edge DoH 的运行时元数据：设备信息、网络环境、风险评级、目标域名等。Edge DoH 基于这些做差异化路由（不同上下文 → 不同节点池）。 |

---

## 6. 运维与可观测

| 术语 | 含义 |
|------|------|
| **拨测** | **AxisNow Cloud** 对 Data Plane 节点的健康度探测，结果用于 Edge DoH 调度时剔除不可用节点。<br>**注**：旧版 glossary 描述为"控制面节点对数据面节点的拨测"不准确——拨测由云端发起，是全局监控，不是边缘节点之间的探测。 |
| **axhttp** | AxisNow SDK 之上的 HTTP 客户端扩展层，零侵入封装主流 HTTP 框架（OkHttp / Retrofit / URLSession 等），见 [指南 / 支持的平台与框架 / axhttp 框架覆盖](../guides/platforms.md#axhttp-框架覆盖)。 |
