# 自定义方法

此篇文章讨论如何在 API 设计中使用自定义方法。

自定义方法指五个标准方法之外的 API 方法。**应该（should）** 只有当标准方法不能完成需要的功能时才使用自定义方法。一般情况下，API 设计者 **应该（should）** 在可行的情况下选择标准方法。标准方法有更简洁和明确定义的语义并且被大多数开发者熟知，所以它们更易于使用并且不易出错。另一个优势是 API 平台对标准方法的支持更好，比如计费、错误处理、日志和监控。

自定义方法可以被关联到资源、集合或服务。它 **可以（may）** 接收任意请求并返回任意响应，并且也可以支持流式请求与响应。

## HTTP 映射

自定义方法 **应该（should）** 使用如下的通用映射方法：

```
https://service.name/v1/some/resource/name:customVerb
```

使用 `:` 代替 `/` 来分隔资源名与自定义动词是为了支持任意 `path` 参数，例如，*取消删除（undelete）* 文件能映射为 `POST /files/a/long/file/name:undelete`。

选择 HTTP 映射时 **必须（shall）** 遵守如下指南：

- 自定义方法 **应该（should）** 使用 HTTP `POST` 动词，因为 `POST` 有更加灵活的语义
- 自定义方法 **不应该（should not）** 使用 HTTP `PATCH`，但 **可以（may）** 使用其它 HTTP 动词。这种情况下，方法 **必须（must）** 遵循该动词的标准 HTTP 语义
- 注意，使用 HTTP `GET` 的自定义方法 **必须（must）** 是**幂等的并且不能有副作用**。例如，在资源上实现特殊查询的自定义方法应该使用 HTTP `GET`。
- 自定义方法中待操作的资源或集合名 **应该（should）** 映射到 URL `path` 参数
- URL path **必须（must）** 以冒号加上自定义动词做为后缀
- 如果自定义方法使用的 HTTP 动词允许 HTTP 请求体（`POST`、`PUT`、`PATCH`或自定义的 HTTP 动词），这些自定义方法的 HTTP 配置 **必须（must）** 使用 `body: "*"` 并且所有剩余的请求信息 **必须（shall）** 映射到 HTTP 请求体中
- 如果自定义方法使用的 HTTP 动词不允许 HTTP 请求体（`GET`、`DELETE`），这些方法的 HTTP 配置 **一定不要（must not）** 使用 `body`，所有剩余的请求信息 **必须（shall）** 映射到 HTTP `query` 参数中

警告：如果服务实现多个 API，**必须（must）** 小心地创建服务配置以避免 API 间的自定义动词冲突。

```
// 服务级别的自定义方法
rpc Watch(WatchRequest) returns (WatchResponse) {
  // 自定义方法映射到 HTTP POST，所有参数放到请求体中
  option (google.api.http) = {
    post: "/v1:watch"
    body: "*"
  };
}
// 集合级别的自定义方法
rpc ClearEvents(ClearEventsRequest) returns (ClearEventsResponse) {
  option (google.api.http) = {
    post: "/v3/events:clear"
    body: "*"
  };
}
// 资源级别的自定义方法
rpc CancelEvent(CancelEventRequest) returns (CancelEventResponse) {
  option (google.api.http) = {
    post: "/v3/{name=events/*}:cancel"
    body: "*"
  };
}
// 用于批量 get 的自定义方法
rpc BatchGetEvents(BatchGetEventsRequest) returns (BatchGetEventsResponse) {
  // 批量 get 方法映射到 HTTP GET
  option (google.api.http) = {
    get: "/v3/events:batchGet"
  };
}
```

## 用例

一些可选择自定义方法的其他情况：

- **重启虚拟机**： 设计的备选方案可以是“在重启资源集合中创建一个重启资源”（过于复杂）或者“虚拟机具有可变的状态，客户端能够将其从运行中改变为重启中”（会引入更多问题，比如是否有其他状态间的转变）。此外，重启是一个被熟知的概念，能够很好地转换为满足开发者需求的自定义方法
- **发送邮件**： 创建邮件信息并不一定要将它发送出去（草稿）。相对于备选方案（将消息移动到 Outbox 集合），自定义方法的优点是可以被 API 用户更多地发现并且更直接地理解它的概念
- **员工晋级**： 如果使用标准的 `update` 来实现，客户端必须重复进行管理流程的策略来保证正确的晋级

一些标准方法比自定义方法更好的例子：

- 使用不同的参数查询资源（使用标准的 `list` 方法和过滤）
- 简单的资源修改（使用带有字段掩码的标准 `update` 方法）
- 撤消通知（使用标准的 `delete` 方法）

## 通用自定义方法

常用的自定义方法名称列表如下。 API 设计者在引入自已的名称之前应该考虑这些名称，以便于跨 API 的一致性

| 方法名 | 自定义动词 | HTTP 动词 | 备注 |
|------|------|------|------|
| Canecl | :cancel | POST | 取消未完成的操作（构建、计算等）|
| BatchGet<复数名词> | :batchGet | POST | 批量取得多个资源（详情请查看 [List 的描述](standard-methods.md)）|
| Move | :move | GET | 将资源从一个父资源移动到另一个中 |
| Search | :search | GET | 用于获取不符合 List 语义的数据 |
| Undelete | :undelete | POST | 恢复以前删除的资源，推荐的保留期为30天 |