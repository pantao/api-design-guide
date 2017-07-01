# 文件结构

gRPC API **应该（should）** 使用 [proto3](proto3.md) 在 `.proto` 文件中定义。

文件中 **必须（must）** 将高层级和更重要的定义放在其他项目之前。原型文件 **应该（should）** 参照下面的顺序：

- 版权和许可协议（如果需要）
- 按照 `syntax`, `package`, `option` 和 `import` 的顺序写的原型
- API 概述文档
- 以引入顺序降序排列的 `service` 定义
- 资源 `message` 定义，父资源 **必须（must）* 在子资源之前定义
- RPC 请求和响应的 `message` 定义，与对应方法的顺序相同。每个请求信息 - 必须（must）-  在对应响应信息之前（如果有的话）

如果一个原型文件包含了所有的 API，那么它应该以 API 名字命名：

API | Proto
-|-
Library | library.proto
Calendar | calendar.proto

如果需要的话，可以将一个大 `.proto` 文件中的服务、资源消息、请求/响应消息移动到单独的文件中。

推荐在单独的文件中保存单独的服务和对应的请求响应消息，将文件命名为 `<enclosed service name>.proto`。对于只有资源的原型文件，可将它命名为 `resources.proto`。

## proto 文件名

proto 文件 **应该（should）** 以小写字母下划线分隔的名字命名，并且 **一定（must）** 要使用 `.proto` 做为后缀。例如：`service_controller.proto`。

## proto option

为了在不同 API 中生成一致的客户端库，API 开发者 **必须（must）** 在 `.proto` 文件中使用一致的 proto option。参照本指南的 API 定义 **必须（must）** 使用如下的 option：

```
syntax = "proto3";
// 包名要以公司名开始，以主版本号结束
package google.abc.xyz.v1;
// 这个 option 表示 namespace 在 C# 中使用。默认使用包名的帕斯卡命名法，当包名由一个
// 单词的当包名由一个单词的字段组成时这样是没问题的。
// 例如，包名 "google.shopping.pets.v1" 应该使用 "Google.Shopping.Pets.V1" 格式的
// C# namespace。
// 但是如果有多个单词组成的字段时，这个 option 就需要避免只有首个单词大写。
// 例如，Google Pet Store API 的包名可以是 "google.shopping.petstore.v1"，意味着它
// 对应的 C# namespace 是 "Google.Shopping.Petstore.V1"。因为 option 应该将它改为
// "Google.Shopping.PetStore.V1"。
// 对于 C#/.NET 命名格式的更多信息，请查看[Framework Design Guidelines](https://msdn
// .microsoft.com/en-us/library/ms229043)
option csharp_namespace = "Google.Abc.Xyz.V1";
// 这个 option 允许 proto 编译器在包名而不是外部类中生成 Java 代码。通过减少一层名字和
// 保持与其他大部分不支持外部类相同提高了开发者的使用体验。
option java_multiple_files = true;
// Java 外部类应该以文件名的首字母大写的驼峰命名法命名。这个类只用于保存 proto 的描述符，
// 所以开发者不需要直接使用它。
option java_outer_classname = "XyzProto";
// Java 包名必须是添加了合适前缀的 proto 包名
option java_package = "com.google.abc.xyz.v1";
// 从包中生成 Objective-C 符号的前缀。最短 3 个字符、全大写并且惯例是使用包名的缩写。
// 一些较短但足够唯一不会与其他冲突的名字之后可能会被使用。'GPB' 被 protocol buffer
// 保留使用。
option objc_class_prefix = "GABCX";
```