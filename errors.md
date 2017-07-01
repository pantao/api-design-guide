# 错误处理

本章简单介绍 Google API 的错误模型以及开发人员如何正确生成和处理错误的一般指导。

Google API 使用了简单的协议无关的错误模型，这允许我们在不同 API，不同协议（例如 gRPC 或 HTTP），不同的错误上下文（如同步、批量处理、工作流错误）中提供相同的使用体验。

## 错误模型

错误模型由 [google.rpc.Status](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto) 定义。如下所示：

```
package google.rpc;
message Status {
  // 容易被客户端处理的简单错误码，实际的错误码定义在`google.rpc.Code`
  int32 code = 1;
  // 易于开发者阅读的错误信息，它应该解释错误并且提供可行的解决方法
  string message = 2;
  // 附加的错误信息，客户端能够通过它处理错误，例如重试的等待时间或者帮助页面链接
  repeated google.protobuf.Any details = 3;
}
```

因为大部分 Google API 使用面向资源的设计，所以错误处理通过在大量资源上使用一小组标准错误也遵循了相同的设计原则。例如，服务端使用一个标准的 `google.rpc.Code.NOT_FOUND` 错误码加上特定的资源名来表示“未找到”错误，而不是定义不同种类的“未找到”错误。更少的错误状态减少了文档的复杂性，在客户端的库中提供更好的习惯性映射，在不限制可包含信息的情况下减少了客户端逻辑的复杂性。

## 错误码

Google API **必须（must）** 使用 `google.rpc.Code` 中定义的标准错误码。单独的 API **应该（should）** 避免定义附加错误码，因为开发者非常不喜欢为大量错误码编写处理逻辑。作为参考，每个 API 处理平均 3 个错误码意味着大部分程序逻辑在进行错误处理，这并不是好的开发体验。

## 错误消息

错误消息应该帮助用户轻松并快速地 **理解并解决** API 错误。通常情况请参考如下规则：

- 不要假设用户非常了解你的 API。用户可能是客户端开发者、运维人员、IT 人员或者 app 的普通用户。
- 不要假设用户了解服务实现的细节或熟悉错误上下文（例如日志分析）。
- 如果可能的话，应构建错误消息，以便技术用户（但不一定是 API 的开发人员）可以对错误进行响应并更正。
- 保持错误信息的简短。如果需要的话，提供链接以便迷惑的用户能够提出问题得到反馈或得到更多信息。否则，请使用 details 字段来扩展错误消息。

## 错误详情

Google API 为错误详情定义了一组标准错误负载，可以去 [google/rpc/error_details.proto](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto) 查看。这里包含了 API 中最常见的错误，例如达到资源限额和错误的输入参数。与错误码相同，错误详情也应该使用标准的负载。

只有在能够帮助程序代码处理错误时才可以为错误详情引入新的类型。如果错误信息只能够由人（非代码）处理，应当让开发者依赖错误消息的内容来手动处理，而不是引入新的错误详情类型。如果新的类型被引入，一定要为它们进行显式的注册。

这里有一些 `error_details` 负载的示例：

- `RetryInfo` 描述了当客户端能够重试请求时，可能返回 `Code.UNAVAILABLE` 或 `Code.ABORTED`
- `QuotaFailure` 描述了配额检查失败的原因，可能返回 `Code.RESOURCE_EXHAUSTED`
- `BadRequest` 描述了客户端的非法请求，可能返回 `Code.INVALID_ARGUMENT`

## HTTP 映射

虽然 `proto3` 有原生的`JSON` 编码，但 Google 的 API 平台使用如下的 JSON 格式进行错误响应，以允许向后兼容：

```
{
  "error": {
    "code": 401,
    "message": "Request had invalid credentials.",
    "status": "UNAUTHENTICATED",
    "details": [{
      "@type": "type.googleapis.com/google.rpc.RetryInfo",
      ...
    }]
  }
}
```

字段 | 描述
error | 为了向后兼容 Google API 客户端库添加的额外层
code | Status.code 映射为 HTTP 状态码
message | 对应 Status.message
status | 对应 Status.code
details | 对应 Status.details

## RPC 映射

不同的 RPC 协议用不同的方法映射到错误模型（error model）。对于 [gRPC](http://grpc.io/)，生成的代码和所有语言的运行库都原生支持错误模型。你可以去 gRPC 的 API 文档中查看详情。

## 客户端库的映射

Google 客户端库可能会选择按照不同的惯例来对不同语言进行不同的错误处理。例如，库 [google-cloud-go](https://github.com/GoogleCloudPlatform/google-cloud-go) 会返回 `google.rpc.Status` 的实例，而 [google-cloud-java](https://github.com/GoogleCloudPlatform/google-cloud-java) 则会抛出异常。

## 错误信息本地化

`google.rpc.Status` 中的 `message` 字段是面向开发者的，**必须（must）** 是英语。

如果需要向用户提供错误信息，请使用 `google.rpc.LocalizedMessage` 作为详情字段。`google.rpc.LocalizedMessage` 可以被本地化，但请保证 `google.rpc.Status` 中是英文。

API 服务应该默认使用认证用户的 `locale` 或 `HTTP Accept-Language` 头来决定本地化语言。

## 错误处理

下表包含了所有在 [google.rpc.Code](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto) 中定义的 gRPC 错误代码和产生原因的简单描述。可以通过查看返回状态码的描述并修改对应的代码来处理错误。

HTTP | RPC | Description
-|-|-
200 | OK | 没有错误
400 | INVALID_ARGUMENT | 客户端使用了错误的参数，通过 error message 和 error details 查看更多信息
400 | FAILED_PRECONDITION | 当前的系统状态不能执行请求，例如删除非空目录
400 | OUT_OF_RANGE | 客户端指定无效范围
401 | UNAUTHENTICATED | 由于缺少、无效或过期的 OAuth 令牌，请求未通过身份验证
403 | PERMISSION_DENIED | 客户端没有足够的权限，这可能是因为 OAuth 令牌没有正确的范围，客户端没有权限或者 API 还没有开放
404 | NOT_FOUND | 指定的资源不存在，或者由于未公开的原因（如白名单）请求被拒绝
409 | ABORTED | 并发冲突，例如读写冲突
409 | ALREADY_EXISTS | 客户端试图创建的资源已经存在
429 | RESOURCE_EXHAUSTED | 超过资源限额或频率限制,客户端应该通过 google.rpc.QuotaFailure 查看更多信息
499 | CANCELLED | 客户端取消请求
500 | DATA_LOSS | 不可恢复的数据丢失或损坏，客户端应该将此错误报告给用户
500 | UNKNOWN | 服务端未知错误，一般是 BUG
500 | INTERNAL | 服务端内部错误，一般是 BUG
501 | NOT_IMPLEMENTED | 服务端未实现此 API
503 | UNAVAILABLE | 服务端不可用，一般是服务端挂了
504 | DEADLINE_EXCEEDED | 请求超过最后期限，如果重复发生，请考虑减少请求的复杂性

## 错误重试

当发生 `500`，`503` 和 `504` 错误时客户端 **应该（should）** 以指数级增长的间隔来重试请求。除非文档中进行了说明，最小的重试间隔应该是 1 秒。对于 `429` 错误，客户端应该以最小 30 秒的间隔重试。对于其他错误，重试操作可能并不可行，请先确保请求是幂等的并查看错误消息以获得指引。

## 错误传播

如果 API 服务依赖于其他服务，则不应盲目地将这些服务中的错误传播给客户端。翻译错误时，有如下建议：

- 隐藏实现细节和机密信息
- 调整负责该错误的一方。例如，应把从其他服务接收到 `INVALID ARGUMENT` 错误转换为 `INTERNAL` 返回给调用者。

## 生成错误

\服务端产生的错误应该包含足够多的信息来帮助客户端开发者理解和解决问题。同时也要小心用户数据的安全和隐私，因为错误经常会被作为日志记录下来并被其他人查看，所以应避免在错误信息和错误详情中暴露敏感信息。例如，错误信息“Client IP address is not on whitelist 128.0.0.0/8”将用户不可访问的服务端策略暴露出去了。

为了生成合适的错误，你首先应该熟悉 `google.rpc.Code` 来为每种错误条件选择最合适的错误。服务端程序可以并行检查多个错误条件，然后返回第一个。

下表列出了每一种错误码和对应的错误信息示例。

HTTP | RPC | 错误信息示例
-|-|-
400 | INVALID_ARGUMENT | Request field x.y.z is xxx, expected one of [yyy, zzz].
400 | FAILED_PRECONDITION | Resource xxx is a non-empty directory, so it cannot be deleted.
400 | OUT_OF_RANGE | Parameter ‘age’ is out of range [0, 125].
401 | UNAUTHENTICATED | Invalid authentication credentials.
403 | PERMISSION_DENIED | Permission ‘xxx’ denied on file ‘yyy’.
404 | NOT_FOUND | Resource ‘xxx’ not found.
409 | ABORTED | Couldn’t acquire lock on resource ‘xxx’.
409 | ALREADY_EXISTS | Resource ‘xxx’ already exists.
429 | RESOURCE_EXHAUSTED | Quota limit ‘xxx’ exceeded.
499 | CANCELLED | Request cancelled by the client.
500 | DATA_LOSS | 请看提示
500 | UNKNOWN | 请看提示
500 | INTERNAL | 请看提示
501 | NOT_IMPLEMENTED | Method ‘xxx’ not implemented.
503 | UNAVAILABLE | 请看提示
504 | DEADLINE_EXCEEDED | 请看提示

提示：因为客户端不能修复服务端的错误，生成额外的错误详情并没有用处。为了避免通过 error condition 泄露敏感信息，推荐不要生成任何 error message 并且只生成 `google.rpc.DebugInfo` 错误详情。`DebugInfo` 只能用于服务端日志，不要发送给客户端。

`google.rpc` 定义了一组标准错误负载，它们优先于自定义的错误负载。下表列出了每个错误代码及其匹配的标准错误负载。

HTTP | RPC | 推荐的错误详情
-|-|-
400 | INVALID_ARGUMENT | google.rpc.BadRequest
400 | FAILED_PRECONDITION | google.rpc.PreconditionFailure
400 | OUT_OF_RANGE | google.rpc.BadRequest
401 | UNAUTHENTICATED | 
403 | PERMISSION_DENIED | 
404 | NOT_FOUND | 
409 | ABORTED | 
409 | ALREADY_EXISTS | 
429 | RESOURCE_EXHAUSTED | google.rpc.QuotaFailure
499 | CANCELLED | 
500 | DATA_LOSS | 
500 | UNKNOWN | 
500 | INTERNAL | 
501 | NOT_IMPLEMENTED | 
503 | UNAVAILABLE | 
504 | DEADLINE_EXCEEDED