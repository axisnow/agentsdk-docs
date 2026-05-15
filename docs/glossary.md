# 术语表

下列术语来自 AgentSDK 自有定义，与公网常见叫法可能不完全对应。详细机制请到对应章节查阅。

| 术语 | 含义 |
|------|------|
| **AxisNow Cloud** | 中心化的设备 / 资源 / 策略编排平台，下发路由规则与拨测结果。SDK **不与之直连**，所有交互经控制面节点中转。 |
| **控制面节点** | 客户自部署的边缘节点，承担寻址、配置下发、API 中继、日志聚合。Edge DoH 服务运行在它之上。详见 [控制面路由](introduction/control-plane.md)。 |
| **数据面节点** | 承载业务流量代理转发与链路加密 / 签名的边缘节点。可在 AxisNow / 第三方（Cloudflare、AWS 等）/ 自部署之间混部。 |
| **Edge 节点** | 控制面节点与数据面节点的统称。在不强调控制面 / 数据面区别的语境里使用。 |
| **Edge DoH** | 控制面节点对外暴露的 DNS-over-HTTPS 解析服务，以 AccessKey 签名认证，提供防劫持、防污染的域名解析。配置详见 [DNS 配置](introduction/dns.md)。 |
| **EdgeDoH 白名单** | `edgedoh_resolve_domains` 中列出的域名（精确或 `*.suffix` 通配）走 EdgeDoH，未命中走系统 DNS。|
| **EdgeDoH 豁免** | `edgedoh_bypass_domains` 中列出的域名一律走系统 DNS，优先级高于白名单。 |
| **SecureProxy（安全代理）** | SDK 端侧的安全代理能力，含两类防护：**加密通信隧道**（本地代理 → 数据面节点的 TLS 链路）与**基于可信设备环境的访问控制**（由 Edge 侧依据控制台策略执行）。 |
| **本地代理** | SDK 在设备本地监听的 TCP / HTTP / SOCKS5 端口，业务请求经本地代理转发到 Edge 节点。 |
| **AccessKey** | 控制台下发的 ID + Secret 身份凭证对。SDK 初始化时传入，用于 Edge DoH 签名认证与控制面消息验签；凭证缺失或非法时初始化返回错误码 `-112` / `-113` / `-114`，详见 [错误码](errors.md)。 |
| **资源** | 控制台为业务域名定义的代理对象，关联源站、路由规则、访问策略。**未配资源的域名不走 SDK 代理**，按本地 DNS 直连源站。 |
| **设备上下文** | SDK 上报给控制面的运行时元数据（设备信息、网络环境、风险评级、目标域名等），是 Edge DoH 做差异化路由的输入。 |
| **EIP** | Elastic IP，控制面节点的固定入口 IP。`edgeAddresses` 中以纯 IP 形式预埋的部分作为冷启动入口与最后兜底。 |
| **兜底域名** | `edgeAddresses` 中以域名形式预埋的部分（如 `doh.acme.com`），通过公共 DoH 解析得到控制面 IP，作为内置 EIP 失联时的回退入口。 |
| **拨测** | 控制面节点对数据面节点的健康度探测，结果用于 DoH 调度时优选可用节点。 |
| **axhttp** | AgentSDK 之上的 HTTP 客户端扩展层，零侵入封装主流 HTTP 框架（OkHttp / Retrofit / URLSession 等），见 [支持的平台与框架](guides/platforms.md#http-客户端扩展axhttp)。 |
