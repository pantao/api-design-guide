# 标准方法

此章节定义标准方法 `List`、`Get`、`Create`、`Update` 和 `Delete`。标准方法存在的意义是广泛的 API 中许多 API 方法具有非常相似的语义，通过将这些类似的 API 融合到标准方法中，我们可以显著降低复杂性并提高一致性。以 [Google APIs](https://github.com/googleapis/googleapis) 为例，超过 70% 是标准方法，这让它们更加易于学习和使用。

下表描述了如何将它们映射为 REST 方法，也称为 CRUD 方法：

| 方法    | HTTP 映射   | HTTP 请求体  | HTTP 响应体  |
|--------|-------------|-------------|-------------|
| `List` | GET <集合 URL> | 空 | 资源[1]列表 |
| `Get`  | GET <资源 URL> | 空 | 资源[1] |
| `Create` | POST <集合 URL> | 资源 | 资源[1] |
| `Update` | PUT 或 PATCH <资源 URL> | 资源 | 资源[1] |
| `Delete` | DELETE <资源 URL> | 空 | 空[2] |

[1] `List`、`Get`、`Create` 和 `Update` 方法支持字段掩码时，返回的资源 **可能（MAY）** 只包含部分数据。在某些情况下，API 平台会对所有方法原生支持字段掩码。

[2] 不立即删除资源（比如通过更新标志位或会执行时间较长的删除操作）的 `Delete` 方法返回的响应 **应该（SHOULD）** 包含长时间运行的操作或被修改的资源。

标准方法也 **可以（MAY）** 为不能在一个 API 调用周期完成的请求返回长期运行的操作。

下面章节详细描述了每一个标准方法。这些例子展示了在 `.proto` 文件中定义的方法，其中包含用于 HTTP 映射的特殊注释。你可以在 [Google APIs](https://github.com/googleapis/googleapis) 项目中找到许多使用标准方法的例子。

## List

`List` 方法接收资源名和零个或多个其它参数做为输入，返回符合输入的资源列表。它也通常被用来搜索资源。

`List` 适合取得没有缓存且大小有限的来自单个集合的数据。对于更广泛的情况，**应该（should）** 使用[自定义方法](custom-methods.md)。

应该使用自定义的 `BatchGet` 方法来实现批量获取（例如接收多个资源 ID然后返回对应的资源），而不是使用 `List`。但如果已存在能够提供同样功能的 `List` 方法，你 **可以（may）** 为此目的重用 `List` 方法。如果你在使用自定义的 `BatchGet` 方法，**应该（should）** 将它映射成 `HTTP GET`。

适用的常见模式：[分页](design-patterns.md)、[结果排序](design-patterns.md)

适用的命名约定：[过滤字段](naming-conventions.md)、[结果字段](naming-conventions.md)

HTTP 映射：

- `List` 方法 **必须（must）** 使用 `HTTP GET` 动词。
- **应该（should）** 把要列出集合的资源名字放在 URL path 参数中。如果集合名映射到 URL path 中，URL 模版的最后一段（[集合 ID](resource-names.md]） **必须（must）** 是常量。
- 所有其他的请求信息字段 **必须（shall）** 映射到 URL 的 `query` 参数中。
- 没有请求体，API 配置中 **一定不能（must not）** 定义 `body`。
- 响应体 **应该（should）`` 包含资源列表和可选的元数据。

```
// 列出指定书架上的所有图书
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
  // List 方法映射为 HTTP GET
  option (google.api.http) = {
    // `parent` 获取父资源名，例如 "shelves/shelf1"
    get: "/v1/{parent=shelves/*}/books"
  };
}
message ListBooksRequest {
  // 父资源名称，例如 "shelves/shelf1".
  string parent = 1;
  // 返回值的最大条数
  int32 page_size = 2;
  // 从上一个 List 请求返回的 next_page_token 值（如果存在）
  string page_token = 3;
}
message ListBooksResponse {
  // 字段名应该匹配方法名中的名词 "books"，根据请求中的 page_size 字段，将会返回最大数量的条目
  repeated Book books = 1;
  // 用于取得下一页结果的值，没有时为空
  string next_page_token = 2;
}
```

## Get

`GET` 方法接收资源名，零个或多个参数，返回指定的资源。

HTTP 映射：

- `Get` 方法 **必须（must）** 使用 `HTTP GET` 动词。
- 表示资源名的请求信息字段 **应该（should）** 映射到 URL path 参数中。
- 所有其他的请求信息字段 **必须（shall）** 映射到 URL 的 `query` 参数中。
- 没有请求体，API 配置中**一定不能（must not）** 定义 `body`。
- 返回的资源 **必须（shall）** 填充整个响应体。

```
// 取得指定的 book
rpc GetBook(GetBookRequest) returns (Book) {
  // Get 映射为 HTTP GET。资源名映射到 URL 中。没有请求体
  option (google.api.http) = {
    // 注意 URL 中用于获取资源名的模板变量，例如 "shelves/shelf1/books/book2"
    get: "/v1/{name=shelves/*/books/*}"
  };
}
message GetBookRequest {
  // 此字段包含被请求资源的名字，例如："shelves/shelf1/books/book2"
  string name = 1;
}
```

## Create

`Create` 方法接收一个集合名、零个或多个参数，在指定集合中创建一个新的资源并将其返回。

如果 API 支持创建资源，它 **应该（should）** 在所有可被创建的资源上具有 `Create` 方法 。

HTTP 映射：

- `Create` 方法 **必须（must）** 使用 `HTTP POST` 动词。
- 请求消息中 **应该（should）** 含有名为 `parent` 的字段来接收新建资源的父资源名。
- 所有其他的请求信息字段 **必须（shall）** 映射到 URL 的 `query` 参数中。
- 请求 **可以（may）** 包含名为 `<resource>_id` 的字段名来允许调用者选择一个客户端分配的 ID。这个字段 **必须（must）** 映射到 URL `query` 参数中。
- 包含资源的请求信息字段 **应该（should）** 映射到请求体中。如果 `Create` 方法使用了 HTTP 配置中的 `body` 字段，那么 **必须（must）** 使用 `body: "<resource_field>"` 这种格式。
- 返回的资源 **必须（shall）** 填充到整个响应体。

如果 `Create` 方法支持客户端指定资源名，当资源名已存在时请求 **应该（should）** 失败 （**推荐（recommended）** 使用 `google.rpc.Code.ALREADY_EXISTS`））或者服务端分配另外的名字，并且文档中应该明确指出创建的资源名可能会与传入的名字不同。

```
rpc CreateBook(CreateBookRequest) returns (Book) {
  // Create 映射为 HTTP POST，URL path 做为集合名称
  // HTTP 请求体中包含资源
  option (google.api.http) = {
    // 通过 `parent` 获取父资源名，例如 "shelves/1"
    post: "/v1/{parent=shelves/*}/books"
    body: "book"
  };
}
message CreateBookRequest {
  // 将被创建的 book 的父资源名
  string parent = 1;
  // book 使用的 ID
  string book_id = 3;
  // 资源 book 将被创建，字段名应该与方法名中的名词相匹配
  Book book = 2;
}
rpc CreateShelf(CreateShelfRequest) returns (Shelf) {
  option (google.api.http) = {
    post: "/v1/shelves"
    body: "shelf"
  };
}
message CreateShelfRequest {
  Shelf shelf = 1;
}
```

## Update

`Update` 方法接收包含资源和零个或多个参数的请求，更新指定的资源和属性并返回更新后的资源。

可修改的资源属性 **应该（should）** 使用 `Update` 方法来更新，除非属性中包含[资源名或父资源](resource-names.md)。任何重命名或移动资源的操作 **一定不要（must not）** 通过 `Update 执行`，**必须（shall）** 通过自定义方法处理。

HTTP 映射：

- 标准的 `Update` 方法 **应该（should）** 支持资源的部分更新，使用带有名为 `update_mask` 的 `FieldMask` 字段的 HTTP 动词 `PATCH` 来执行操作。
- **应该（should）** 使用[自定义方法](custom-methods.md)来实现更高级的 `Update` 方法，例如追加重复的字段。
- 如果 `Update` 方法仅支持资源的完整更新，则 **必须（must）** 使用 HTTP 动词 `PUT` 实现。然而并不鼓励这样做，因为当添加新的资源字段时会有后向兼容的问题。
- 被修改资源的名称字段 **必须（must）** 映射到 URL `path` 参数中。此字段也 **可以（may）** 加在资源信息中。
- 包含资源的请求信息 **必须（must）** 映射到请求体中。
- 所有其他请求信息 **必须（must）** 映射到 URL `query` 参数中。
- 返回响应中的资源信息 **必须（must）** 是被修改的资源。
- 如果 API 接收客户端分配的资源名，那么服务端 **可以（may）** 允许客户端指定一个不存在的资源名来创建新的资源。否则，`Update` 方法 **应该（should）** 因为不存在的资源名而失败。当资源名不存在是唯一错误时，**应该（should）** 使用错误码 `NOT_FOUND`。

即使 API 的 `Update` 方法能够新建资源，它也 **应该（should）** 提供 `Create` 方法。这是因为只有 `Update` 方法能够新建资源会让人迷惑。

```
rpc UpdateBook(UpdateBookRequest) returns (Book) {
  // Update 映射为 HTTP PATCH。资源名映射到 URL path 参数中
  // 资源包含在 HTTP 请求体中
  option (google.api.http) = {
    // 注意用于获取待更新 book 的资源名的 URL 模版变量
    patch: "/v1/{book.name=shelves/*/books/*}"
    body: "book"
  };
}
message UpdateBookRequest {
  // 替换服务端上的 book 资源
  Book book = 1;
  // 用于资源更新的掩码
  // `FieldMask` 的定义请参考 https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#fieldmask
  FieldMask update_mask = 2;
}
```

## Delete

`Delete` 方法接收资源名和零个或多个参数，然后删除或准备删除指定资源。`Delete` 方法 **应该（should）** 返回 `google.protobuf.Empty`。

注意，API **不应该（should not）** 依赖 `Delete` 方法返回的任何信息，因为它不能重复调用。

HTTP 映射：

- `Delete` 方法 **必须（must）** 使用 HTTP `DELETE` 动词
- 表示资源名的请求信息字段 **应该（should）** 映射到 URL `path` 参数中
- 所有其他的请求信息字段 **必须（shall）** 映射到 URL 的 `query` 参数中
- 没有请求体，API 配置 **一定不要（must not）** 定义 `body`
- `Delete` 方法直接删除资源时，**应该（should）** 返回空的响应
- `Delete` 方法初始化一个长时间的操作时，**应该（should）** 返回这个操作
- `Delete` 方法将资源标记为被删除时，**应该（should）** 返回更新后的资源

`Delete` 方法的调用在效果上应该是幂等的，但并不需要有相同的返回值。任意次 `Delete` 请求的结果 **应该（should）** 是资源被删除，但只有第一次请求应该返回正确，后续的请求应该返回 `google.rpc.Code.NOT_FOUND`。

```
rpc DeleteBook(DeleteBookRequest) returns (google.protobuf.Empty) {
  // Delete 映射为 HTTP DELETE。资源名映射到 URL path 参数中
  // 没有请求体
  option (google.api.http) = {
    // 注意 URL 模板变量获取待删除资源的名称，例如 "shelves/shelf1/books/book2"
    delete: "/v1/{name=shelves/*/books/*}"
  };
}
message DeleteBookRequest {
  // 待删除的资源名称，例如 "shelves/shelf1/books/book2"
  string name = 1;
}
```