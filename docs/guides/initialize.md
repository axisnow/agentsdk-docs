# 初始化 SDK

在应用启动**最早时机**调用 SDK 初始化接口，确保在发起任何网络请求之前完成。各平台触发位置：

- **Android**：`Application.onCreate`
- **iOS**：`-application:didFinishLaunchingWithOptions:`
- **Flutter**：应用入口 `main()` 异步初始化

初始化时需要传入 **AccessKey**、**Edge 节点地址列表**，可选传入 **DNS 配置**、**加密隧道开关** 等——完整参数语义、默认值、约束见 [参考 / 初始化参数](../reference/init-parameters.md)。

## 调用契约

- 返回 `0` 表示成功，负数为错误码（见 [错误码](../errors.md)）
- 全局**仅允许成功初始化一次**，重复调用返回 `-102`
- 初始化失败后再次调用返回缓存的错误，需**重启进程**重试

## 代码示例

!!! note
    下列代码为接口语义示意，真实 API 签名以各平台 Demo 为准——见 [平台与框架适配](platforms.md)。

=== "Android (Java)"

    ```java
    public class App extends Application {
        @Override
        public void onCreate() {
            super.onCreate();
            AXServiceConfig config = new AXServiceConfig();
            config.setAccessKeyID("YOUR_ACCESS_KEY_ID");
            config.setAccessKeySecret("YOUR_ACCESS_KEY_SECRET");
            config.setEdgeAddresses(Arrays.asList("doh.example.com", "203.0.113.1"));
            int ret = AXService.initialize(this, config);
            if (ret != 0) {
                Log.e("AXService", "init failed: " + ret);
            }
        }
    }
    ```

=== "iOS (Objective-C)"

    ```objc
    - (BOOL)application:(UIApplication *)application
        didFinishLaunchingWithOptions:(NSDictionary *)options {
        AXServiceConfig *config = [[AXServiceConfig alloc] init];
        config.accessKeyID = @"YOUR_ACCESS_KEY_ID";
        config.accessKeySecret = @"YOUR_ACCESS_KEY_SECRET";
        config.edgeAddresses = @[@"doh.example.com", @"203.0.113.1"];
        int ret = [AXService initializeWithConfig:config];
        if (ret != 0) {
            NSLog(@"AXService init failed: %d", ret);
        }
        return YES;
    }
    ```

=== "Flutter (Dart)"

    ```dart
    Future<void> main() async {
      WidgetsFlutterBinding.ensureInitialized();
      final config = AxServiceConfig(
        accessKeyID: 'YOUR_ACCESS_KEY_ID',
        accessKeySecret: 'YOUR_ACCESS_KEY_SECRET',
        edgeAddresses: ['doh.example.com', '203.0.113.1'],
      );
      final ret = await AxService.initialize(config);
      if (ret != 0) {
        debugPrint('AxService init failed: $ret');
      }
      runApp(const MyApp());
    }
    ```

=== "Unity (C#)"

    ```
    待补充——Unity 平台 API 文档建设中，请参考 Unity Demo 仓库
    ```

## 常见初始化错误

| 错误码 | 含义 | 排查方向 |
|---|---|---|
| `-102` | 重复初始化 | 检查是否多次调用 `initialize` |
| `-111` | 配置 JSON 非法 | 检查字段类型与必填项 |
| `-112` / `-113` / `-114` | AccessKey 缺失或非法 | 重新从控制台复制完整凭证 |
| `-115` | EdgeAddresses 为空 | 至少配置一个有效的 Edge 节点地址 |

完整错误码定义见 [错误码](../errors.md)。

## 下一步

初始化成功后，把业务请求接入 SDK 本地代理 → [集成网络库](network-integration.md)。
