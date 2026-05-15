# 集成网络库

初始化完成后，把业务请求接入 SDK 的**本地代理**。SDK 对外提供三种代理模式，按业务网络库选其一：

| 模式 | API（概念名） | 适用场景 | 接入方式 |
|------|----------------|---------|---------|
| **HTTP 代理** | `getLocalHTTPProxy()` | OkHttp / URLSession / Dio 等 HTTP 客户端 | 作为 HTTP Proxy 注入客户端 |
| **SOCKS5 代理** | `getLocalSocks5Proxy()` | 非 HTTP 协议、通用代理需求 | 作为 SOCKS5 Proxy 注入 |
| **TCP 代理** | `getLocalTCPProxy(host, port)` | Socket 编程、自定义协议、精细控制特定 host:port | 用返回的本地地址替换原始连接目标 |

完整接口定义见 [参考 / 代理配置](../reference/proxy-config.md)；底层机制见 [系统架构 § 代理模式](../introduction/architecture.md#3-代理模式)。

## 推荐：使用 axhttp 扩展（零侵入）

如果业务使用主流 HTTP 框架（OkHttp / Retrofit / URLSession / `http` 包等），**直接用对应的 axhttp 扩展**——它封装了上述代理调用，业务层只需替换客户端实例：

=== "Android (Java + OkHttp)"

    ```java
    import com.axsecurity.axhttp.okhttp.AXHTTPService;

    // 直接取 SDK 托管的 OkHttpClient
    OkHttpClient client = AXHTTPService.getOkHttpClient();

    Request request = new Request.Builder()
        .url("https://api.example.com/data")
        .build();

    try (Response response = client.newCall(request).execute()) {
        // 正常使用
    }
    ```

    如需自定义超时、拦截器、SSL/TLS：

    ```java
    AXHTTPService.setOkHttpClientBuilder(
        new OkHttpClient.Builder()
            .connectTimeout(5, TimeUnit.SECONDS)
            .readTimeout(10, TimeUnit.SECONDS)
    );
    OkHttpClient client = AXHTTPService.getOkHttpClient();
    ```

=== "iOS (Objective-C + URLSession)"

    ```objc
    NSURLSession *session = [AXURLSession sharedSession];
    NSURL *url = [NSURL URLWithString:@"https://api.example.com/data"];
    [[session dataTaskWithURL:url
              completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        // 正常使用
    }] resume];
    ```

=== "Flutter (Dart + http)"

    ```dart
    import 'package:axhttp/axhttp.dart';

    final client = AxClient();
    final response = await client.get(Uri.parse('https://api.example.com/data'));
    // 正常使用
    ```

各平台完整支持矩阵与 Demo 仓库见 [平台与框架适配](platforms.md)。

## 手动调用代理 API

如果框架不在 axhttp 覆盖范围内，或需要非 HTTP 协议代理，直接调用核心 SDK 的代理接口：

=== "Android (Java)"

    ```java
    // HTTP 代理
    AXLocalProxy proxy = AXService.getLocalHTTPProxy();
    Proxy javaProxy = new Proxy(Proxy.Type.HTTP,
            new InetSocketAddress(proxy.getIp(), proxy.getPort()));
    // 注入到 HTTP 客户端

    // SOCKS5 代理
    AXLocalProxy socks5 = AXService.getLocalSocks5Proxy();
    Proxy socksProxy = new Proxy(Proxy.Type.SOCKS,
            new InetSocketAddress(socks5.getIp(), socks5.getPort()));

    // TCP 代理（按目标 host:port 注册）
    AXLocalProxy tcp = AXService.getLocalTCPProxy("backend.example.com", 443);
    // 用 tcp.getIp() + tcp.getPort() 替换原始连接目标
    ```

=== "iOS (Objective-C)"

    ```objc
    AXLocalProxy *proxy = [AXService getLocalHTTPProxy];
    NSURLSessionConfiguration *cfg = [NSURLSessionConfiguration defaultSessionConfiguration];
    cfg.connectionProxyDictionary = @{
        @"HTTPEnable": @YES,
        @"HTTPProxy": proxy.ip,
        @"HTTPPort": @(proxy.port),
    };
    ```

=== "Flutter (Dart)"

    ```dart
    final proxy = await AxService.getLocalHTTPProxy();
    final httpClient = HttpClient();
    httpClient.findProxy = (uri) => 'PROXY ${proxy.ip}:${proxy.port}';
    ```

## 调用约束

- 所有代理接口在 `initialize` 成功（返回 `0`）后才可调用，否则返回未就绪
- 返回的本地地址在 SDK 生命周期内**保持不变**，可缓存到业务层重复使用
- TCP 代理按目标 host:port 注册——同一 host:port 重复调用返回同一本地端口

## 下一步

业务请求接入完成后，启用 EdgeDoH 防 DNS 劫持 → [启用 EdgeDoH](enable-edgedoh.md)。
