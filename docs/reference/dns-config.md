# DNS 配置

DNS 配置控制 SDK 的域名解析行为，初始化时通过 `dns` 配置传入，所有参数均为**可选**——**不配置时所有域名走系统 DNS**。各平台具体字段名与代码 tabs 见 [指南 / 启用 EdgeDoH](../guides/enable-edgedoh.md)；解析路径背后的原理见 [白皮书 / AxisNow Edge DoH（AED）](../introduction/whitepaper.md#axisnow-edge-dohaed)。

!!! note "本页配置针对业务域名"
    下列白名单 / 豁免 / 缓存等配置都作用于**业务域名**（如 `api.app.com`）——决定 SDK 如何解析它们。
    SDK 自身寻址 Edge DoH 用的**调度域名**（在 `edgeNodes` 中配置）有独立的 3 层 EIP 兜底机制，不受本页配置影响。两类域名的区分详见 [白皮书 / 路由机制：业务域名与调度域名](../introduction/whitepaper.md#路由机制业务域名与调度域名)。

## DNS 路由：Edge DoH 白名单 + 黑名单豁免

通过两个**业务域名**列表控制对外解析路径：

- **Edge DoH 白名单**（`EdgeDohResolveDomains`）：命中（精确或 `*.suffix` 通配）的业务域名走 Edge DoH，未命中走系统 DNS。
- **Edge DoH 黑名单豁免**（`EdgeDohBypassDomains`）：优先级高于白名单；命中 bypass 的业务域名一律走系统 DNS，即便它同时命中白名单。

匹配优先级：`bypass` 命中 → 系统 DNS（结束）；否则 `resolve` 命中 → Edge DoH；否则 → 系统 DNS（默认）。

bypass 的存在是为了支持"`*.example.com` 全走 EdgeDoH 但 `login.example.com` 例外"这类场景，无需逐一枚举子域名白名单。

!!! warning "通配仅支持 `*.suffix` 一种形式"
    白名单 / 豁免的每个条目，要么是**精确域名**（`api.example.com`），要么是 **`*.` 开头的后缀通配**（`*.example.com`，匹配该后缀下的子域名）。**其他任何 `*` 写法都不支持**：

    - 裸 `*`（无法用单个条目匹配"全部域名"）
    - 中间 / 多段通配：`api.*.com`、`*.example.*`
    - 片段通配（`*` 不紧跟 `.`）：`api*.example.com`、`*example.com`

    不支持的写法不会按通配生效（会被当作普通字符的精确串，从而永远不命中）。如需覆盖某后缀下的所有子域名，请用 `*.example.com`。

## DNS 缓存控制

两条解析路径（EdgeDoH 与系统 DNS）共用同一份 SDK 缓存表，下列配置对两条路径**统一生效**：

| 参数 | 默认 | 范围 | 说明 |
|---|---|---|---|
| 缓存 TTL 下限 | 60 秒 | ≥ 5 秒 | 低于此值的 TTL 会被提升至此值；响应未携带 TTL 时也使用此值 |
| 缓存 TTL 上限 | 300 秒 | ≤ 600 秒 | 高于此值的 TTL 会被截断 |
| 否定缓存 TTL | 10 秒 | — | NXDOMAIN / NODATA 等否定响应的缓存时长 |

!!! note
    实际缓存时间满足：下限 ≤ 实际 TTL ≤ 上限。若上限低于下限，上限自动提升到下限。

## DNS 容错（fallback + 过期 IP 兜底）

两项独立的容错开关，**默认均启用**：

- **fallback 兜底**：命中 EdgeDoH 的域名解析失败时，hedged 并发到系统 DNS（1s 后启动并发请求，谁先成功用谁）。**仅对命中 EdgeDoH 的域名生效**——未走 EdgeDoH 的域名本身就在系统 DNS 上，无需 fallback。
- **过期 IP 兜底**：所有解析路径均失败时，允许返回已过期的缓存 IP，用于维持弱网连通性。两条路径都生效。

## 单次解析超时

单次解析端到端超时（**默认 10000 ms，最低 1000 ms**），包含重试与 hedged fallback。低于 1000 ms 会被自动提升至 1000 ms。EdgeDoH 与系统 DNS 路径共用此值。

## DNS 预解析名单

初始化时**后台异步预解析**的域名列表，结果提前写入缓存以消除首次请求的 DNS 延迟。预解析按路由规则自动分发到 EdgeDoH 或系统 DNS（**不必是 `EdgeDohResolveDomains` 子集**）。

- 建议只放启动后**一定会访问**的域名。
- 数量控制在 **10 个以内**。

## 典型配置组合范例

| 组合 | EdgeDoH 路由 | 豁免规则 | 适用场景 |
|------|--------------|----------|---------|
| 全量启用（推荐） | 业务域名加入白名单（如 `*.example.com`） | — | 生产环境默认，业务域名经 Edge DoH 解析后走 Secure Proxy 加密回源 |
| 域名分流 | 业务域名加入白名单 | 登录 / 内网域名加入豁免 | 大部分域名走 EdgeDoH，少数特殊域名豁免回系统 DNS |
