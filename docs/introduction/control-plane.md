# 控制面路由

> **受众**: SDK 开发者、各平台 Wrapper 开发者
> **状态**: 设计文档

## 1. 概述

控制面路由负责为 SDK 的控制面请求选取可用的 Edge 节点地址。控制面请求包括：

- **DoH 查询** — 通过 Edge 节点的 DoH 服务解析域名
- **资源配置拉取** — 从控制中心获取租户的资源配置列表
- **日志 / Metrics 上报** — 向 Edge 节点上报运行数据

SDK 维护三类 EIP 地址池，每次发起控制面请求时按优先级从中选取一个可用 IP。

## 2. 三类地址池

| 地址池 | 简称 | 解析源 | DNS 引擎 | 可用时机 |
|--------|------|--------|----------|----------|
| **内置 IP 池** | staticIPs | 初始化参数中的 IP 地址 | — (无需解析) | 始终可用 |
| **公共 DNS 解析池** | domainIPs | 初始化参数中的域名 | 公共 HTTP DNS / System DNS | 首次解析成功后可用 |
| **Edge DoH 解析池** | routingIPs | 初始化参数中的域名 | Edge DoH (Context-Aware) | 首次调度成功后可用 |

> **公共 DNS 解析池和 Edge DoH 解析池的解析源是同一组内置域名**（`EdgeAddresses` 中的域名部分）。两者的区别仅在于 DNS 引擎不同：公共 DNS 解析池走公共 HTTP DNS 或系统 DNS，作为通用兜底；Edge DoH 解析池走 Edge 节点的 DoH 服务，能基于设备上下文返回最优地址，优先级更高。

### 2.1 内置 IP 池（staticIPs）

用户初始化 SDK 时通过 `InitialConfig.EdgeAddresses` 传入的 **IP 地址**。

```json
{
  "edge_addresses": ["1.2.3.4", "5.6.7.8", "edge.example.com"]
}
```

- `EdgeAddresses` 中的纯 IP 地址会被提取为静态 IP 池
- 编译时确定，不依赖任何网络请求，**始终可用**
- 作为最基础的兜底保障，即使所有 DNS 和调度均不可用，仍能尝试连接

**获取方式**：`edgediscovery.NewEdgeServiceDiscovery()` 在初始化时通过 `net.ParseIP()` 从 `EdgeAddresses` 中分离出 IP 地址存入 `staticIPs`。

### 2.2 公共 DNS 解析池（domainIPs）

对 `EdgeAddresses` 中的**域名**（非 IP 部分），通过**公共 HTTP DNS 或系统 DNS** 解析获得的 IP。

- 解析源：`EdgeAddresses` 中的域名，与 Edge DoH 解析池共享同一组域名
- DNS 引擎：公共 HTTP DNS (`httpdns`)，失败时 fallback 到系统 DNS (`systemdns`)
- 如果 `EdgeAddresses` 中没有域名，则此池为空（无此能力）
- 如果有多个域名，SDK 会解析所有域名并合并结果
- 内置域名推荐通过 DNS Routing 来控制其外网解析结果
- 作为 Edge DoH 解析池的**兜底**，当 Edge DoH 不可用时提供域名解析能力

**获取方式**：SDK 启动后，`routing.StartPreloadLoop()` 中的 `dnsResolver` 定期通过公共 HTTP DNS 解析内置域名，结果通过 `routing.GetIPv4ByHostFromCache()` / `routing.GetIPv4ByHost()` 提供给地址池发现。

### 2.3 Edge DoH 解析池（routingIPs）

对 `EdgeAddresses` 中的**域名**（与公共 DNS 解析池相同的一组域名），通过 Edge 节点的 **Context-Aware DoH 插件**解析获得的 IP。Edge 节点会根据客户端 IP、地理位置、设备信息等上下文条件，从配置的地址策略中选举最优地址返回。

- 解析源：`EdgeAddresses` 中的域名，与公共 DNS 解析池共享同一组域名
- DNS 引擎：Edge DoH (`edgedoh`)，基于设备上下文的智能解析
- 需要至少一次成功的 DoH 请求，**首次启动时不存在**（此时由公共 DNS 解析池兜底）
- 请求中携带 `routing_purpose: control` 标识控制面用途
- 响应中的 `is_eip` 字段标识该地址是否为 Edge EIP
- 同一域名的 Edge DoH 解析结果**优先级高于**公共 DNS 解析结果（更贴近当前网络环境）

**获取方式**：`routing.StartPreloadLoop()` 中的 `edgeResolver` 定期向 Edge 节点发起 DoH 解析请求，携带设备上下文（tenant、device、app 信息等），结果通过 `routing.GetIPsForHostFromCache()` 提供给地址池发现。

### 2.4 两个解析池的关系

SDK 启动后，`routing.StartPreloadLoop()` 为同一组内置域名启动两个独立的后台 Resolver：

| Resolver | DNS 引擎 | 结果流向 | 对应函数 |
|----------|----------|----------|----------|
| `edgeResolver` | Edge DoH (`edgedoh`) | Edge DoH 解析池 (tier 0) | `routing.GetIPsForHostFromCache()` |
| `dnsResolver` | 公共 HTTP DNS (`httpdns`) + fallback `systemdns` | 公共 DNS 解析池 (tier 2) | `routing.GetIPv4ByHostFromCache()` / `routing.GetIPv4ByHost()` |

两个 Resolver 独立运行、互不依赖。Edge DoH 解析池的结果优先级更高（tier 0），公共 DNS 解析池作为兜底（tier 2）。

### 2.5 池内排序

每个池内的 IP 都会基于以下信号做优先级排序（IP1 > IP2 > IP3）：

1. **本地出口 + GeoIP/ISP 库** — 优先选择网络距离近的 IP（`GeoBalancer`）
2. **全链路 Health-Check** — 优先选择健康的 IP

排序逻辑全异步执行。若排序尚未完成（如首次启动），则使用池内默认顺序（`OrderBalancer`）。

## 3. 选取优先级

三类地址池在 `EdgeServiceDiscovery.Resolve()` 中被组织为 4 个 tier：

| Tier | 对应地址池 | 内容 | 代码 |
|------|-----------|------|------|
| 0 | Edge DoH 解析池 | Routing DoH 缓存中解析到的 IP | `routingCacheFn(domain)` |
| 1 | 内置 IP 池 | GeoIP 最优静态 IP（或首个静态 IP） | `selectBalancer(staticIPs).Next()` |
| 2 | 公共 DNS 解析池 | 域名 DNS 缓存或主动解析的 IP | `resolveDomainIPs()` |
| 3 | 内置 IP 池 | 剩余静态 IP | `staticIPs[1:]` |

`AdaptiveBalancer.next()` 按 tier 顺序迭代，每个 tier 内部通过 `GeoBalancer`（有 GeoIP 数据时）或 `OrderBalancer`（无 GeoIP 时）选取 IP，跳过已尝试失败的地址。

#### Tier 1 选取规则（内置 IP 池）

Tier 1 从全部静态 IP 中选出**一个**最优 IP，选取逻辑取决于本地 GeoIP 能力：

| 条件 | 选取方式 | 说明 |
|------|----------|------|
| 设备外网 IP 已知 **且** 本地 GeoIP 库可用 | `GeoBalancer.PickNearest()` | 根据设备 IP 的地理位置，从所有 staticIPs 中选取网络距离最近的 IP |
| 设备外网 IP 未知 **或** GeoIP 库不可用 | `OrderBalancer`（即 `staticIPs[0]`） | 退化为按初始化顺序选取第一个静态 IP |

- **设备外网 IP**：通过 `base.GetDeviceExternalIP()` 获取，首次启动时可能尚未获取到
- **GeoIP 库**：通过 `geoip.GetMMDBReader()` 判断是否已加载 MMDB 数据文件
- 两个条件**同时满足**才走 GeoIP 选取路径，任一不满足则使用默认顺序

### 3.1 初次安装（Edge DoH 解析池尚不存在）

此时 tier 0 为空，实际可用的是 tier 1 → tier 2 → tier 3：

```
内置IP.最优IP (tier 1, GeoIP 选取或首个)
  │
  ├─ 连通 → 使用
  │
  └─ 不通 → 公共DNS解析池是否有解析结果？
              │
              ├─ 有 → 依次尝试 公共DNS.IP1, IP2, IP3 (tier 2)
              │        └─ 全部不通 → 内置IP.IP2, IP3 ... (tier 3)
              │
              └─ 无 → 内置IP.IP2 (tier 3)
                       └─ 不通 → 等待域名解析完成后重试
```

核心原则：**只要公共 DNS 解析池中有地址，就优先使用公共 DNS 解析池的地址**（域名解析结果通常比静态 IP 更贴近当前网络环境）。

### 3.2 后续启动（Edge DoH 解析池已存在）

调度成功后，tier 0 有值，整体顺序变为 tier 0 → tier 1 → tier 2 → tier 3：

```
EdgeDoH.IP1 (tier 0，最高优先)
  │
  ├─ 连通 → 使用
  │
  └─ 不通 → 快速重试 EdgeDoH.IP2, IP3
              │
              └─ 全部不通 → 回退到「初次安装」逻辑
                            (内置IP.最优IP → 公共DNS解析池 → 内置IP 剩余)
```

### 3.3 调度失败 / 尚无调度

若 Edge DoH 解析池始终为空（调度请求未成功），则等价于初次安装的选取逻辑。

## 4. 域名解析的延迟加载

`resolveDomainIPs()`（tier 2）采用延迟策略，避免在静态 IP 可用时阻塞等待 DNS：

1. 优先查 DNS 缓存（`dnsCacheFn`），有缓存立即返回
2. 缓存未命中时，仅当**所有静态 IP 都已尝试失败**才同步等待 DNS 解析（`dnsResolveFn`）
3. 若仍有未尝试的静态 IP，tier 2 返回空，让流程先走 tier 3 的剩余静态 IP

## 5. Health-Check 与排序

`AdaptiveBalancer.Next()` 在选出候选 IP 后，会通过 `netstats.HealthCheckIsHealthy()` 检查健康状态。不健康的 IP 会被跳过并继续尝试下一个。当所有 tier 的所有 IP 均不可用时，重置健康检查状态并返回失败。

## 6. 初始化流程

```
InitializeSDK(config)
  │
  ├─ routing.Init(store, config.EdgeAddresses)
  │    └─ 从 EdgeAddresses 中提取域名 → routingDomains
  │
  ├─ edgediscovery.NewEdgeServiceDiscovery(config.EdgeAddresses, ...)
  │    ├─ net.ParseIP() 分离 IP → staticIPs (内置 IP 池)
  │    ├─ 剩余部分 → domains (公共 DNS 解析池 & Edge DoH 解析池的共同解析源)
  │    └─ 注入回调函数：
  │         ├─ routingCacheFn  = routing.GetIPsForHostFromCache  (tier 0)
  │         ├─ dnsCacheFn      = routing.GetIPv4ByHostFromCache  (tier 2 缓存)
  │         └─ dnsResolveFn    = routing.GetIPv4ByHost           (tier 2 主动解析)
  │
  ├─ adaptivebalancer.NewAdaptiveBalancer(discovery)
  │    └─ 关联到 sdkContext.LoadBalancer
  │
  ├─ edgedoh.NewResolver(sdkContext)
  │    └─ 创建 Edge DoH 解析器
  │
  └─ routing.StartPreloadLoop(sdkContext, dohResolver)
       ├─ edgeResolver: Edge DoH 定期解析 routingDomains → Edge DoH 解析池 (tier 0)
       └─ dnsResolver:  HTTP DNS 定期解析 routingDomains → 公共 DNS 解析池 (tier 2)
```

## 7. 关键文件索引

| 文件 | 职责 |
|------|------|
| `pkg/sdk/sdk.go` | SDK 初始化入口，组装各组件 |
| `pkg/loadbalance/edgediscovery/edge_discovery.go` | 4-tier 地址池发现与组织 |
| `pkg/loadbalance/adaptivebalancer/adaptivebalancer.go` | 跨 tier 选取与 Health-Check 过滤 |
| `pkg/loadbalance/geobalancer/` | 基于 GeoIP 的 tier 内排序 |
| `pkg/loadbalance/orderbalancer/` | 默认顺序的 tier 内排序 |
| `pkg/controller/routing/routing.go` | Routing DoH + DNS 缓存管理，公共 DNS 解析池和 Edge DoH 解析池的数据来源 |
| `pkg/dns/edgedoh/` | Edge DoH 解析引擎（Edge DoH 解析池的底层实现） |
| `pkg/dns/resolver/resolver.go` | DNS 路由（EdgeDoH 白名单 / 豁免名单）与缓存 |
| `pkg/netstats/` | Health-Check 状态管理 |
