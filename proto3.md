# 使用 proto3

这一章讨论在 API 设计中如何使用 Protocol Buffer。为了简化开发体验并提高运行效率，gPRC API **应该（should）** 在 API 定义时使用 Protocol Buffers 第 3 版（proto3）。

[Protocol Buffer](https://github.com/google/protobuf) 是一个为了定义数据结构和编程接口的语言独立平台独立的简单的接口定义语言（IDL）。它支持二进制和文本格式，并且能够在不同的平台不同的协议中使用。

proto3 是 Protocol Buffer 的最新版本，与 proto2 比有如下改变：

- 原始字段（primitive fields）不再支持 `hasField`。未设置的原始字段有语言相关的默认值。
    - 消息字段仍然可用，可以使用编译器生成的 `hasField` 方法或与 null 进行比较或与由具体实现定义的哨兵值比较。
- 不再支持用户自定义的字段默认值
- 枚举定义 **必须（must）** 以 `0` 开始
- 不再支持 `required` 字段
- 不再支持扩展（extensions），请使用 `google.protobuf.Any`
- 由于向后兼容性和运行时兼容性的原因，`google/protobuf/descriptor.proto` 特殊例外
- 删除了组语法（group）

删除这些特性是为了让 API 的设计更加简洁可靠和提高性能。例如在记录日志前经常需要过滤一些敏感字段，但当字段是 required 时，这种操作是不可能的。

查看 [Protocol Buffers](https://developers.google.com/protocol-buffers/) 获取更多信息。