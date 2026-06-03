# 启用安全代理

安全代理（Secure Proxy）启用后，SDK 接管命中的业务流量，并根据目标边缘类型采用以下两种模式之一：

- **AxisNow 私有边缘**：与 AxisNow 私有边缘建立**加密隧道**模式，防止流量被外部嗅探。
- **Cloudflare 公共边缘**：在 HTTPS 请求中**注入 JWT Token**，进行客户端访问鉴权。**（暂未支持，规划中）**

其工作原理见 [白皮书 / 请求链路](../introduction/whitepaper.md#请求链路)，术语区分见 [术语表 / Secure Proxy 模式](../introduction/glossary.md)。

!!! tip "什么时候不需要启用"
    如果业务无需对流量做加密隧道或客户端鉴权防护，仅需基础调度与标准转发，**可跳过本节**——SDK **默认不启用安全代理**，业务流量仍可正常经 Edge 转发。

SDK 默认 `secureProxyEnabled = false`。要启用安全代理，在初始化时通过代理配置把 `secureProxyEnabled` 置为 `true` 即可。

!!! warning "前提：必须先在控制台添加域名配置"
    安全代理只对**控制台中已配置为资源的业务域名**生效。开启 `secureProxyEnabled` 后，SDK 仅对这些已配置域名的命中流量启用安全代理（当前为 AxisNow 私有边缘的加密隧道模式；Cloudflare 公共边缘的 JWT 鉴权暂未支持）；未在控制台配置的域名即便开了开关也不会走安全代理（按系统 DNS 直连源站）。请先完成 [接入准备 / 控制台添加资源配置](prerequisites.md#控制台添加资源配置可选)。

    **推荐**：为这些域名**同时启用 EdgeDoH**，由 Edge DoH 基于上下文返回最优数据面节点并防 DNS 劫持，与安全代理配合获得完整防护，见 [启用 EdgeDoH](enable-edgedoh.md)。

## SDK 侧：开启 secureProxyEnabled

在 `initialize` 的配置中加入代理配置，并把 `secureProxyEnabled` 设为 `true`：

=== "Android (Java)"

    ```java
    import com.axsecurity.sdk.service.AXService;
    import com.axsecurity.sdk.base.AXConfig;

    AXConfig config = new AXConfig.Builder()
        .accessKey("YOUR_ACCESS_KEY_ID", "YOUR_ACCESS_KEY_SECRET")
        .edgeNodes(new String[] { "edge.example.com", "203.0.113.1" })
        .proxy(new AXConfig.ProxyConfig.Builder()
            .secureProxyEnabled(true)
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
    config.edgeNodes       = @[ @"edge.example.com", @"203.0.113.1" ];

    AXProxyConfig *proxy = [[AXProxyConfig alloc] init];
    proxy.secureProxyEnabled = YES;
    config.proxy = proxy;

    [AXService initialize:config];
    ```

=== "iOS (Swift)"

    ```swift
    import AXSecurity

    let config = AXConfig()
    config.accessKeyID     = "YOUR_ACCESS_KEY_ID"
    config.accessKeySecret = "YOUR_ACCESS_KEY_SECRET"
    config.edgeNodes       = ["edge.example.com", "203.0.113.1"]

    let proxy = AXProxyConfig()
    proxy.secureProxyEnabled = true
    config.proxy = proxy

    AXService.initialize(config)
    ```

=== "Flutter (Dart)"

    ```dart
    import 'package:axsecurity_flutter_plugin/axsecurity_flutter_plugin.dart';
    import 'package:axsecurity_flutter_plugin/config.dart';

    final cfg = AxConfig(
      accessKeyId: 'YOUR_ACCESS_KEY_ID',
      accessKeySecret: 'YOUR_ACCESS_KEY_SECRET',
      edgeNodes: ['edge.example.com'],
      proxy: AxProxyConfig(secureProxyEnabled: true),
    );
    await AxService.initialize(config: cfg);
    ```

启用后无需改动业务接入代码：流量仍按 [集成网络库](network-integration.md) 中的方式经本地代理交给 SDK，是否走加密隧道由本开关与目标类型共同决定。

!!! note "安全代理与 EdgeDoH 相互独立"
    `secureProxyEnabled` 控制 **SDK 如何与边缘建立连接**（加密隧道），EdgeDoH 控制 **域名如何解析**（防 DNS 劫持）。两者正交，可单独开启、互不依赖。需要防 DNS 劫持解析时另见 [启用 EdgeDoH](enable-edgedoh.md)。

## 下一步

- 启用 EdgeDoH 防 DNS 劫持 → [启用 EdgeDoH](enable-edgedoh.md)
- 验证安全代理是否生效 → [验证接入](verification.md)
- 代理配置完整字段 → [参考 / 代理配置](../reference/proxy-config.md)
