# 配置 DNS 路由

[启用 EdgeDoH](enable-edgedoh.md) 教了最基本的白名单/豁免用法。本页是**进阶配置**：缓存、容错、预解析、典型配置组合。

所有字段的完整定义见 [参考 / 初始化参数 / DNS 配置](../reference/init-parameters.md#dns-配置)。背后原理见 [白皮书 / AxisNow Edge DoH（AED）](../introduction/whitepaper.md#axisnow-edge-dohaed)。

!!! note "本页配置针对业务域名"
    本页的白名单/豁免/缓存等配置都作用于**业务域名**（如 `api.app.com`）——决定 SDK 如何解析它们。
    SDK 自身寻址 Edge DoH 用的**调度域名**（在 `edgeNodes` 中配置）有独立的 3 层 EIP 兜底机制，不受本页配置影响。两类域名的区分详见 [白皮书 / 路由机制：业务域名与调度域名](../introduction/whitepaper.md#路由机制业务域名与调度域名)。

## 1. 白名单与豁免（进阶）

```text
EdgeDohResolveDomains    → 命中的业务域名走 Edge DoH
EdgeDohBypassDomains     → 命中一律回系统 DNS（优先级高于白名单）
```

> 各平台具体方法名：Android `addEdgeDohResolveDomain(String)` / iOS `-addEdgeDohResolveDomain:` / Flutter `AxDnsConfig(edgeDohResolveDomains: [...])`。bypass 同理。

匹配优先级：**bypass → resolve → 默认（系统 DNS）**。

典型用法：

- `*.example.com` 全走 EdgeDoH，**但 `login.example.com` 例外**（业务策略要求走系统 DNS）：把后者加入 `EdgeDohBypassDomains`
- **只保护特定域名**（如 `api.example.com`、`cdn.example.com`），其他全走系统：精确加入 `EdgeDohResolveDomains`

## 2. DNS 缓存控制

两条解析路径（EdgeDoH 与系统 DNS）共用同一份 SDK 缓存表，下列配置对两条路径**统一生效**：

| 参数 | 默认 | 范围 | 说明 |
|---|---|---|---|
| 缓存 TTL 下限 | 60 秒 | ≥ 5 秒 | 低于此值的 TTL 会被提升至此值；响应未携带 TTL 时也使用此值 |
| 缓存 TTL 上限 | 300 秒 | ≤ 600 秒 | 高于此值的 TTL 会被截断 |
| 否定缓存 TTL | 10 秒 | — | NXDOMAIN / NODATA 等否定响应的缓存时长 |

!!! note
    实际缓存时间满足：下限 ≤ 实际 TTL ≤ 上限。若上限低于下限，上限自动提升到下限。

## 3. DNS 容错（fallback + 过期 IP 兜底）

两项独立的容错开关，**默认均启用**：

- **fallback 兜底**：命中 EdgeDoH 的域名解析失败时，hedged 并发到系统 DNS（1s 后启动并发请求，谁先成功用谁）。**仅对命中 EdgeDoH 的域名生效**——未走 EdgeDoH 的域名本身就在系统 DNS 上，无需 fallback
- **过期 IP 兜底**：所有解析路径均失败时，允许返回已过期的缓存 IP，用于维持弱网连通性。两条路径都生效

## 4. 单次解析超时

单次解析端到端超时（**默认 10000 ms，最低 1000 ms**），包含重试与 hedged fallback。低于 1000 ms 会被自动提升至 1000 ms。EdgeDoH 与系统 DNS 路径共用此值。

## 5. DNS 预解析名单

初始化时**后台异步预解析**的域名列表，结果提前写入缓存以消除首次请求的 DNS 延迟。预解析按路由规则自动分发到 EdgeDoH 或系统 DNS（**不必是 `EdgeDohResolveDomains` 子集**）。

- 建议只放启动后**一定会访问**的域名
- 数量控制在 **10 个以内**

## 6. 典型配置组合范例

| 组合 | EdgeDoH 路由 | 豁免规则 | 适用场景 |
|------|--------------|----------|---------|
| 全量启用（推荐） | 业务域名加入白名单（如 `*.example.com`） | — | 生产环境默认，业务域名经 Edge DoH 解析后走 Secure Proxy 加密回源 |
| 域名分流 | 业务域名加入白名单 | 登录 / 内网域名加入豁免 | 大部分域名走 EdgeDoH，少数特殊域名豁免回系统 DNS |

## 下一步

配置完成后，到控制台验证生效情况 → [验证接入](verification.md)。
