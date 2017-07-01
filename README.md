## 介绍

这是一篇网络 API 的通用设计指南，它从2014年开始被 Google 使用，并且指导我们设计了 [Cloud API](https://cloud.google.com/apis/docs/overview) 和 其它 [Google API](https://github.com/googleapis/googleapis)。我们将此指南分享出来希望能让人们更便捷地合作。

[Google Cloud Endpoints](https://cloud.google.com/endpoints/docs/grpc) 的开发者在设计 gRPC API 时可能会发现这篇指南特别有用，我们强烈推荐这类开发者使用这些设计原则，但我们不会要求非 Google 员工必须遵守。

此指南适用于 [REST API](https://zh.wikipedia.org/wiki/REST) 和 [RPC API](https://zh.wikipedia.org/wiki/RPC)，特别是 gRPC API，gRPC API 使用 [Protocol Buffers](http://tailnode.tk/2017/04/google-api-design-guide/proto3/) 来定义API接口，使用 [API Service Configuration](https://github.com/googleapis/googleapis) 来配置 API 服务，包括 HTTP 映射(HTTP mapping)、日志和监控。HTTP 映射功能被 Google API 和 Cloud Endpoints gRPC API 用来进行 JSON/HTTP 与 Protocol Buffers/RPC 的转换。

新的风格和设计被采纳和核准后将会被添加进来，所以此指南会持续更新。并且API设计的艺术与技术会一直进步，所以此指南永远不会“完结”。

## 约定

这些关键词**MUST**”、**MUST NOT**、**REQUIRED**、**SHALL**、**SHALL NOT**、**SHOULD**、**SHOULD NOT**、**RECOMMEND**、**MAY**、**OPTIONAL** 的解释请参考 [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) 或[中文解释](rfc2119.md)（翻译时我会保留这些单词并且加粗显示）。