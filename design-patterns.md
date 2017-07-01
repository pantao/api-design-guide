# 设计模式

## 空响应体

标准的 `Delete` 方法 **必须（must）** 返回 `google.protobuf.Empty` 来实现全局一致性。它还可以防止客户端依赖于在重试期间不可用的附加元数据。因为随着时间推移对于自定义方法， 对于自定义方法，它们 **必须（must）** 具有自己的 `XxxResponse` 消息，即使它们是空的，因为功能很可能随着时间的推移而增加，并且需要返回附加数据。

## 范围字段

表示范围的字段 **应该（should）** 使用符合命名约定的半开半闭区间 `[start_xxx, end_xxx)`，例如 `[start_key, end_key)` 或 `[start_time, end_time)`。C++ STL 和 Java 标准库经常使用半开半闭的语义。API **应该（should）** 避免使用表示区间的其他方法，例如 (`index`, `count`) 或 [`first`, `last`]。

## 资源标签

在面向资源的 API 中，资源结构由 API 定义。为了允许客户端向资源附加少量且简单的元数据（例如将一台虚拟机资源标记为数据库服务器），API **应该（should）** 使用在 `google.api.LabelDescriptor` 中描述的资源标签设计模式。

API **应该（should）** 在资源定义中添加字段 `map<string, string> labels`：

```
message Book {
  string name = 1;
  map<string, string> labels = 2;
}
```

## 耗时操作

如果一个 API 方法需要花费较长时间运行，可以将其设计成向客户端返回一个表示长时间运行的资源，客户端可通过这个资源来获取操作的执行进度并取得执行结果。[Operation](https://github.com/googleapis/googleapis/blob/master/google/longrunning/operations.proto) 定义了耗时操作的标准接口。**不能（must not）** 为 API 使用自定义的耗时操作接口以免打破一致性。

资源 **必须（must）** 作为响应消息直接返回，并且对资源操作的结果 **应该（should）** 反应在 API 中。例如：当创建资源时这个资源 **应该（should）** 显示在 `LIST` 和 `GET` 方法中，并且 **应该（should）** 指示出这个资源还没有准备好。如果方法不需要长期执行，当操作完成时 `Operation.response` 字段应该包含直接返回的消息。

## 列表分页

即使结果集很小，可 LIST 的集合也 **应该（should）** 支持分页。

理由：尽管向已有的 API 添加分页功能从 API 的视角来看是纯粹的增加功能，但它实际会改变行为。不知道有分页功能的已有客户端会错误地将取到的第一页数据当成全部数据。

为了在 `List` 方法中支持分页， API **应该（shall）**：

- 在 `List` 方法的请求信息中定义一个 `string` 字段 `page_token`。客户端通过这个字段来请求指定的某一页。
- 在 `List` 方法的请求信息中定义一个 `int32` 字段 `page_size`。客户端通过这个字段来指定返回结果的最大数量。服务端可以进一步限制在单个页面中返回的最大结果数量。`page_size` 是 `0` 时，将由服务端决定返回结果的数量。
- 在 `List` 方法的响应信息中定义一个 `string` 字段 `next_page_token`。这个字段表示取得下一页的页码。空字符串表示没有更多数据了。

为了取得下一页的结果，客户端 **应该（shall）** 将响应中的 `next_page_token` 传入下次的请求：

```
rpc ListBooks(ListBooksRequest) returns (ListBooksResponse);
message ListBooksRequest {
  string name = 1;
  int32 page_size = 2;
  string page_token = 3;
}
message ListBooksResponse {
  repeated Book books = 1;
  string next_page_token = 2;
}
```

当客户端在 `query` 参数传入除 `page token` 之外的参数时，如果 `query` 参数与 `page token` 不一致，服务 **必须（must）** 拒绝此请求。

`page token` 的内容 **应该（should）** 是对 web 安全的 `BASE64` 编码后的 `protocol buffer`，这样就不会有兼容性问题。`page token` 中存在敏感信息时，**应该（should）** 将其加密。服务端 **必须（must）** 通过以下方法来防止通过篡改 `page token` 来获取敏感信息的问题：

- 根据后续请求指定 `query` 参数
- 在 `page token` 中仅引用服务端的状态
- 在 `page token` 中加密并签名 `query` 参数，并且在每次调用中对这些参数进行验证和鉴权

分页功能也 **可以（may）** 在响应中通过名为 `total_size` 类型为 `int32` 的字段来提供查询资源的总数量。

## 列出子集合

API 有时需要客户端对子集合进行 `List`/`Search` 操作。例如一个 API 有书架集合，每个书架有书的集合，客户端想要在所有书架中搜索一本书。这种情况下推荐在子集合上使用标准的 `List`，并且为父集合指定通配符 `"-"`。例如：

```
GET https://library.googleapis.com/v1/shelves/-/books?filter=xxx
```

注意：使用 `"-"` 而非 `"*"` 是为了避免 URL 转义。

## 从子集合中取得唯一资源

有时子集合中的资源具有在其父集合内唯一的标识符，在这种情况下通过 `Get` 来取得某资源而不需要知道它的父集合可能是有用的。在这种情况下，建议使用标准 `Get`，并为资源唯一的所有父集合指定通配符 `"-"`。例如：

111
GET https://library.googleapis.com/v1/shelves/-/books/{id}
111

响应 **必须（must）** 使用资源的带有父集合标识符的规范名称。例如上面的请求应该返回名称类似 `shelves/shelf713/books/book8141` 的资源，而不是 `shelves/-/books/book8141`。

## 排序
如果 API 方法允许客户端指定列表结果的排序顺序，请求消息中 应该（should） 包含如下字段：

```
string order_by = ...;
```

这个值 **应该（should）** 遵循 SQL 语法：用逗号分隔的字段列表。例如：`"foo,bar"`。默认升序排列。**应该（should）** 给字段添加后缀 `" desc"` 来表示降序。例如：`"foo desc,bar"`。

多余的空格可以忽略，`"foo,bar desc"`和 `" foo , bar desc "`是相等的。

## 请求校验

如果 API 方法有副作用，并且需要仅验证请求而不产生副作用，请求消息 **应该（should）** 包含一个字段：

```
bool validate_only = ...;
```

当此字段设置为 `true` 时，服务端 **一定不要（must not）** 执行任何有副作用的操作，而是对请求进行校验。

校验成功时 **一定（must）** 要返回 `google.rpc.Code.OK`，并且使用相同请求信息的完整请求 **不应该（should not）** 返回 `google.rpc.Code.INVALID_ARGUMENT`。注意，可能因为其他错误（比如 `google.rpc.Code.ALREADY_EXISTS` 或竞态条件）此请求还是会失败。

## 请求重入

对于网络 API，幂等是很重要的，因为当有网络异常时它们能够安全地进行重试。然而一些 API 并不容易实现幂等性，例如需要避免不必要重复的创建资源操作。对于这类情况，请求信息 应该（should） 包含一个唯一 ID（例如 UUID），这样服务端能够通过此 ID 来检测重复，保证请求只被处理一次。

```
// 服务端用于检测重复请求的唯一 ID
// 此字段应该命名为 `request_id`
string request_id = ...;
```

因为客户端很可能没有接收到之前的响应，所以当检测到重复请求后，服务端 应该（should） 返回之前成功的响应。

## 枚举默认值

每个枚举定义 **必须（must）** 以 `0` 值开始，用于当枚举值没有明确指定时。API **必须（must）** 在文档中说明如何处理 `0` 值。

如果有通用的默认行为，**应该（should）** 使用枚举值 `0`。API 应该在文档中说明期待的行为。

如果没有通用的默认行为，枚举值 `0` **应该（should）** 命名为 `ENUM_TYPE_UNSPECIFIED` 并且和错误 `INVALID_ARGUMENT` 一起使用。

```
enum Isolation {
  // 未指定
  ISOLATION_UNSPECIFIED = 0;
  // 快照读。如果所有读写都不能在并发事务中逻辑地序列化，则会发生冲突
  SERIALIZABLE = 1;
  // 快照读。并发事务向同一行写入时导致冲突
  SNAPSHOT = 2;
  ...
}

// 当未指定时，服务器将使用 SNAPSHOT 或更高的隔离级别
Isolation level = 1;
```

一个惯用名称 **可以（may）** 用于 `0` 值，例如，`google.rpc.Code.OK` 是指定不存在错误的惯用方法。在这种情况下，`OK` 与枚举类型中的 `UNSPECIFIED` 在语义上是相等的。

在存在本质上合理和安全的默认情况下，**可以（may）** 使用 `0` 值。例如，在资源视图枚举中 `BASIC` 是 `0` 值。

## 语法句法

在某些 API 设计中，有必要为某些数据格式定义简单的语法，例如可接受的文本输入。为了在不同 API 中提供一致的开发体验和减少学习曲线，API 设计者 **必须（must）** 使用 `ISO 14977` 扩展的 `Backus-Naur` 表格（EBNF）句法来定义这些语法。

```
Production  = name "=" [ Expression ] ";" ;
Expression  = Alternative { "|" Alternative } ;
Alternative = Term { Term } ;
Term        = name | TOKEN | Group | Option | Repetition ;
Group       = "(" Expression ")" ;
Option      = "[" Expression "]" ;
Repetition  = "{" Expression "}" ;
```

注意：`TOKEN` 表示在语法之外定义的终端。

## 整数类型

在API 设计中，**不应该（should not）** 使用像 `uint32` 和 `fixed32` 这种无符号整型，这是因为一些重要的编程语言和系统（例如 `Java`, `JavaScript` 和 `OpenAPI`）不能很好地支持它们并且更容易导致溢出的问题。另一个问题是，不同的 API 很可能对同一个资源使用不匹配的有符号和无符号类型。

在大小和时间这种负数没有意义的类型中 **可以（may）** 使用且仅使用 `-1` 来表示特定的意义，例如到在文件结尾（`EOF`）、无穷的时间、无资源限额或未知的年纪。当这样使用负数时，**必须（must）** 在文档中明确说明以防止混淆。API 生成器也应该在文档中记录隐式默认值 `0` 表示的行为。

## 部分响应

客户端有时只需要响应信息中的特定子集。一些 API 平台提供了对部分响应的原生支持。Google API 平台通过响应字段掩码来提供支持。对于任一 REST API 调用，有一个隐式的系统 `query` 参数 `$fields`，它是 `google.protobuf.FieldMask` 的 `JSON` 表示。在返回给客户端之前，响应消息会被 `$fields` 字段过滤。此行为是在 API 平台自动执行的。

```
GET https://library.googleapis.com/v1/shelves?$fields=name
```

## 资源视图

为了减少网络流量，允许客户端限制服务器在其响应中返回的资源的哪些部分是有用的，返回资源的视图而不是全部资源表示。API 中的资源视图是通过向请求添加参数来实现的，该参数允许客户端在响应中指定要接收资源的哪个视图。

此参数：

- **应该（should）** 是枚举类型
- **必须（must）** 命名为 `view`

枚举中的每个值定义了资源的哪部分（字段）在响应中会被返回。文档中 **应该（should）** 明确记录每个 `view` 值会返回什么。

```
package google.example.library.v1;
service Library {
  rpc ListBooks(ListBooksRequest) returns (ListBooksResponse) {
    option (google.api.http) = {
      get: "/v1/{name=shelves/*}/books"
    }
  };
}
enum BookView {
  // 响应中只包含作者、标题、ISBN 和唯一的图书 ID。这是默认值。
  BASIC = 0;
  // 返回所有信息，包括书中的内容
  FULL = 1;
}
message ListBooksRequest {
  string name = 1;
  // 指定返回图书资源的哪些部分
  BookView view = 2;
}
```

对应的 URL：

```
GET https://library.googleapis.com/v1/shelves/shelf1/books?view=BASIC
```

可以在 [标准方法](standard-methods.md) 一章中查看更多关于方法定义、请求和响应的内容。

## ETag

`ETag` 是一个不透明的标识符，允许客户端进行条件请求。为了支持 `ETag`，API **应该（should）** 在资源定义中包含一个字符串字段 `etag`，它的语义 **必须(must)** 与 ETag的常用用法相匹配。通常，etag 包含由服务器计算出的资源指纹。更多详细信息，请参阅[维基百科](https://en.wikipedia.org/wiki/HTTP_ETag)和 [RFC 7232](https://tools.ietf.org/html/rfc7232#section-2.3)。

`ETags` 可以强验证或弱验证，其中弱验证的ETag 以 `W /` 为前缀。在这种情况下，强验证意味着具有相同 ETag 的两个资源具有相同的内容和相同的额外字段（`Content-Type`）。这意味着强验证的 `ETag` 允许缓存稍后组装的部分响应。

相反，具有相同弱验证 `ETag` 值的资源意味着这些表示在语义上是等效的，但不一定每字节都相同，因此不适合于字节范围请求的响应缓存。

```
// 强验证的 ETag（包含引号）
"1a2f3e4d5b6c7c"
// 弱验证的 ETag（包含前缀和引号）
W/"1a2b3c4d5ef"
```

## 输出字段

API 可能希望将由客户端提供的字段和只由服务端在特定资源上返回的字段进行区分。对于仅输出的字段，**必须（shall）** 记录字段属性。

请注意，如果客户端在请求中设置了仅输出（`output only`）字段，或者客户端使用仅输出字段指定了一个 `google.protobuf.FieldMask`，则服务器 **必须（must）** 接受该请求而不能出错。这意味着服务器 必须（must） 忽略仅输出字段的存在及其任何指示。这个建议的原因是因为客户端通常会将服务器返回的资源重用为另一个请求的输入，例如一个获取到的 `Book` 将在 `UPDATE ` 方法中被再次使用。如果要验证仅输出字段，客户端需要做清除输出字段的额外工作。

```
message Book {
  string name = 1;
  // 只用做输出
  Timestamp create_time = 2;
}
```