# 词汇表

## 网络 API（Networked APIs）

通过计算机网络中运行的应用程序接口。它们使用包括 HTTP 在内的网络协议进行通信，并且生产和消费 API 的往往是不同的组织。

## Google API

Google 服务的网络 API。大部分在 `googleapis.com` 域名上。不包括客户端库和 SDK 等其他类型的 API。

## API 接口（API Interface）

一个 Protocol Buffer 服务的定义。在大多数编程语言中它被映射到一个接口。一个 API 接口可以被多个 API 服务实现。

## API 版本（API Version）

一个 API 接口或多个定义在一起的 API 接口的版本。API 版本通常以字符串表示（例如 “v1”）并且以 API 请求和 Protocol Buffer 的包名表示。

## API 方法（API Method）

API 接口中的一个单独操作。在 Protocol Buffer 中以 `rpc` 定义，并且在大多数编程语言中映射到 API 接口中的一个函数。

## API 请求（API Request）

一个单独的 API 方法调用。它经常用作计费、记录、监控和速率限制的单位。

## API 服务（API Service）

一人部署了暴露出网络端点的 API 接口的实现。API 服务以 [RFC 1035 DNS](https://www.ietf.org/rfc/rfc1035.txt) 格式的服务名（例如 `calendar.googleapis.com`）进行标识。

## API 端点（API Endpoint）

指向用于 API 服务处理实际 API 请求的网络地址。例如 `pubsub.googleapis.com` 和 `content-pubsub.googleapis.com`。

## API 产品（API Product）

一个 API 服务加上相关的组件（服务声明、文档、客户端库和服务支持），组合起来以产品的形式提供给用户。例如 Google Calendar API。注意：人们有时会简单地使用 API 表示 API 产品。

## API 服务定义（API Service Definition）

API 接口的定义（.proto 文件）和 API 服务配置（.yaml 文件）一起定义了API 服务

## API 消费者（API Consumer）

消费 API 服务的实体。对于 Google API，API 消费者一般是拥有客户端程序或服务端资源的 Google 项目。

## API 生产者（API Producer）

产生 API 服务的实体。对于 Google API，API 生产者一般是拥有 API 服务的 Google 项目。

## API 后端（API Backend）

为 API 服务实现了业务逻辑的一组服务和相关的基础设施。

## API 前端（API Frontend）

通过 API 服务提供通用功能的一组服务和相关的基础设施，例如负载均衡器和认证服务器。注意：API 前端和后端可能距离很近也可能很远。有时它们可能会编译成一个二进制文件并运行在一个进程中。