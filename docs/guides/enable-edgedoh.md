# 启用 EdgeDoH

EdgeDoH 是控制面节点提供的 DNS-over-HTTPS 解析服务，可基于设备 / 网络 / 风险上下文动态返回最优节点地址，**防 DNS 劫持、抗污染、抗 DDoS**。

!!! tip "什么时候不需要启用"
    如果只需要加密隧道、不需要防 DNS 劫持，**可跳过本节**——SDK 默认走系统 DNS，业务流量仍可经 SecureProxy 加密转发。

从当前版本起 SDK **默认不启用 EdgeDoH**：未配置白名单时所有域名走系统 DNS。要启用差异化路由与防劫持解析，需要完成控制台与 SDK 两侧配置。

## 控制台侧：确认 DoH 规则已配置

EdgeDoH 的差异化路由依赖控制台的 DoH 规则。

- 如果在 [接入准备 / 控制台添加 DoH 路由规则](prerequisites.md#控制台添加-doh-路由规则可选) 已配置完成 → 跳过此步
- 未配置 → 参考 **控制台配置指南**（产品控制台文档）为需要保护的域名配置 DoH 规则

## SDK 侧：把域名加入 EdgeDoH 白名单

在初始化时通过 DNS 配置中的 `edgedoh_resolve_domains` 传入需要走 EdgeDoH 的域名（精确或 `*.suffix` 通配）：

=== "Android (Java)"

    ```java
    AXServiceConfig config = new AXServiceConfig();
    // ... AccessKey、edgeAddresses 等基础配置 ...

    AXDnsConfig dns = new AXDnsConfig();
    dns.setEdgedohResolveDomains(Arrays.asList("*.example.com"));
    dns.setEdgedohBypassDomains(Arrays.asList("login.example.com"));  // 可选豁免
    config.setDnsConfig(dns);

    AXService.initialize(this, config);
    ```

=== "iOS (Objective-C)"

    ```objc
    AXServiceConfig *config = [[AXServiceConfig alloc] init];
    // ... AccessKey、edgeAddresses 等基础配置 ...

    AXDnsConfig *dns = [[AXDnsConfig alloc] init];
    dns.edgedohResolveDomains = @[@"*.example.com"];
    dns.edgedohBypassDomains = @[@"login.example.com"];  // 可选豁免
    config.dnsConfig = dns;

    [AXService initializeWithConfig:config];
    ```

=== "Flutter (Dart)"

    ```dart
    final config = AxServiceConfig(
      accessKeyID: 'xxx',
      accessKeySecret: 'yyy',
      edgeAddresses: ['doh.example.com'],
      dnsConfig: AxDnsConfig(
        edgedohResolveDomains: ['*.example.com'],
        edgedohBypassDomains: ['login.example.com'],
      ),
    );
    await AxService.initialize(config);
    ```

**匹配优先级**：`edgedoh_bypass_domains` 命中 → 系统 DNS（结束）；否则 `edgedoh_resolve_domains` 命中 → EdgeDoH；否则 → 系统 DNS（默认）。

`bypass` 的存在是为了支持 "`*.example.com` 全走 EdgeDoH 但 `login.example.com` 例外" 这类场景，无需逐一枚举子域名白名单。

## 下一步

基础白名单启用后，可深入配置：

- **DNS 缓存、容错、预解析等高级选项** → [配置 DNS 路由](dns-routing.md)
- **DNS 解析路径背后的原理** → [DNS 配置](../introduction/dns.md)
- **如何验证 EdgeDoH 是否生效** → [验证接入](verification.md)
