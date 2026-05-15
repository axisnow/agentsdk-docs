# 各平台 API

各平台 SDK 包暴露的 API 文档，由各平台 doc generator 生成（Android Javadoc/Dokka、iOS DocC/Jazzy、Flutter Dartdoc、Unity DocFX 产物）。

!!! warning "建设中"
    本节当前为占位章节，待 SDK 各平台 API 文档生成管线接入后，会以 HTML 子目录形式发布到本站对应路径：

    - Android：`/reference/platform-api/android/`
    - iOS：`/reference/platform-api/ios/`
    - Flutter：`/reference/platform-api/flutter/`
    - Unity：`/reference/platform-api/unity/`

在此之前，可参考：

- [API 概念](../index.md)（跨平台一致的参数语义与接口职责）
- [支持的平台与框架](../../guides/platforms.md)（各平台 Demo 仓库链接，含可运行示例代码）

## 与「API 概念」的区别

| | API 概念 | 各平台 API |
|---|---|---|
| 内容 | 跨平台一致的参数语义、默认值、约束 | 方法签名、字段名、返回类型（按平台分类） |
| 来源 | 手写 Markdown | 由各平台 doc generator 自动生成 |
| 抽象层 | 「做什么」 | 「叫什么、参数类型是什么」 |
| 阅读时机 | 接入前理解概念 | 写代码时查 API 签名 |
