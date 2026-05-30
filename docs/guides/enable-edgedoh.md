# 启用 EdgeDoH

Edge DoH 是运行在 **Private Edge** 上的 DNS-over-HTTPS 解析服务，可基于设备 / 网络 / 风险上下文动态返回最优数据面节点地址，**防 DNS 劫持、抗污染、抗 DDoS**。

!!! tip "什么时候不需要启用"
    如果只需要加密隧道、不需要防 DNS 劫持，**可跳过本节**——SDK 默认走系统 DNS，业务流量仍可经 Secure Proxy（Private Edge）或 Standard Proxy（Public Edge）转发。

从当前版本起 SDK **默认不启用 EdgeDoH**：未配置白名单时所有域名走系统 DNS。要启用差异化路由与防劫持解析，需要完成控制台与 SDK 两侧配置。

## 控制台侧：确认 DoH 规则已配置

EdgeDoH 的差异化路由依赖控制台的 DoH 规则。

- 如果在 [接入准备 / 控制台添加 Edge DoH 路由规则](prerequisites.md#控制台添加-edge-doh-路由规则可选) 已配置完成 → 跳过此步
- 未配置 → 参考 **控制台配置指南**（产品控制台文档）为需要保护的域名配置 DoH 规则

## SDK 侧：把业务域名加入 EdgeDoH 白名单

在初始化时通过 DNS 配置加入需要走 EdgeDoH 的**业务域名**（catch-all `*`、精确或 `*.suffix` 通配）：

=== "Android (Java)"

    ```java
    import com.axsecurity.sdk.service.AXService;
    import com.axsecurity.sdk.base.AXConfig;

    AXConfig config = new AXConfig.Builder()
        .accessKey("YOUR_ACCESS_KEY_ID", "YOUR_ACCESS_KEY_SECRET")
        .edgeNodes(new String[] { "doh.example.com", "203.0.113.1" })
        .dns(new AXConfig.DnsConfig.Builder()
            .edgeDohResolveDomains("*.example.com")
            .edgeDohBypassDomains("login.example.com")  // 可选豁免
            .build())
        .build();

    AXService.initialize(this.getApplicationContext(), config);
    ```

=== "iOS (Objective-C)"

    ```objc
    #import <AXSecurity/axsecurity.h>

    AXConfig *config = [[AXConfig alloc] init];
    config.accessKeyID     = @"YOUR_ACCESS_KEY_ID";
    config.accessKeySecret = @"YOUR_ACCESS_KEY_SECRET";
    config.edgeNodes       = @[@"doh.example.com", @"203.0.113.1"];

    AXDNSConfig *dns = [[AXDNSConfig alloc] init];
    dns.edgeDohResolveDomains = @[ @"*.example.com" ];
    dns.edgeDohBypassDomains  = @[ @"login.example.com" ];  // 可选豁免
    config.dns = dns;

    [AXService initialize:config];
    ```

=== "Flutter (Dart)"

    ```dart
    import 'package:axsecurity_flutter_plugin/axsecurity_flutter_plugin.dart';
    import 'package:axsecurity_flutter_plugin/config.dart';

    final cfg = AxConfig(
      accessKeyId: 'YOUR_ACCESS_KEY_ID',
      accessKeySecret: 'YOUR_ACCESS_KEY_SECRET',
      edgeNodes: ['doh.example.com'],
      dns: AxDnsConfig(
        edgeDohResolveDomains: ['*.example.com'],
        edgeDohBypassDomains: ['login.example.com'],  // 可选豁免
      ),
    );
    await AxService.initialize(config: cfg);
    ```

**匹配优先级**：`EdgeDohBypassDomains` 命中 → 不走 EdgeDoH（结束）；否则 `EdgeDohResolveDomains` 命中 → 走 EdgeDoH；否则 → 不走 EdgeDoH（默认）。

`bypass` 的存在是为了支持 "`*.example.com` 全走 EdgeDoH 但 `login.example.com` 例外" 这类场景，无需逐一枚举子域名白名单。把白名单配成 catch-all `["*"]`、再用 bypass 列出例外，即"除黑名单外全部域名走 EdgeDoH"。

!!! tip "让所有域名都走 EdgeDoH"
    把白名单配成单独的 catch-all `*`（如 `.edgeDohResolveDomains("*")`）即匹配**所有域名**，含裸域名。注意 `*.example.com` 这类后缀通配**不匹配裸后缀**（匹配 `api.example.com`，不匹配 `example.com`）——要连裸域名一起覆盖必须用 `*`。

!!! warning "通配支持 catch-all `*` 与 `*.suffix`"
    每个条目要么是 catch-all `*`、精确域名，要么是 `*.` 开头的后缀通配（如 `*.example.com`）。中间通配（`api.*.com`）、片段通配（`api*.example.com`）等**其他 `*` 写法都不支持**。完整规则见 [参考 / DNS 配置](../reference/dns-config.md)。

基础白名单启用后，可深入配置：

- **DNS 缓存、容错、预解析等高级选项** → [参考 / DNS 配置](../reference/dns-config.md)
- **DNS 解析路径背后的原理** → [白皮书 / AxisNow Edge DoH（AED）](../introduction/whitepaper.md#axisnow-edge-dohaed)
- **如何验证 EdgeDoH 是否生效** → [验证接入](verification.md)
