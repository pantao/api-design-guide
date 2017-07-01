# 资源名称

在面向资源的 API 中，资源是命名实体，资源名称是其标识符。每个资源 **必须（ MUST ）** 有唯一的资源名称。资源名称由资源自己的 ID，任一父资源的 ID 及其 API 服务名称组成。下面我们将看一看资源 ID 和资源名是如何构成的。

gRPC API 应该为资源名使用无协议（scheme-less）的 [URI](https://tools.ietf.org/html/rfc3986)。它们通常遵循 REST URL 惯例并且其行为与网络文件路径非常相似。它们能非常容易地映射到 REST API：详细内容查看[标准方法](standard-methods.md)。

集合是一种特殊类型的资源，它包含了相同类型子资源的列表。例如，目录是文件资源的集合。集合的资源 ID 叫做集合 ID。

资源名称由集合 ID 和资源 ID 按层次组织形成，并以斜杠（`/`）分隔。如果资源包含子资源，子资源名称的格式是父资源名称后面加上子资源 ID，同样地使用斜杠分隔。

例 1：一个存储服务具有 `buckets` 集合，每个 `bucket` 具有 `objects` 集合：

| API 服务名                | 集合ID    | 资源ID     | 集合ID    | 资源ID      |
|--------------------------|----------|------------|----------|------------|
| //storage.googleapis.com | /buckets | /bucket-id | /objects | /object-id |

例 2：一个具有 `users` 集合的邮件服务，每个用户具有 `settings` 子资源， `settings` 子资源具有 `customFrom` 和另外的子资源：

| API 服务名                | 集合ID    | 资源ID            | 集合ID     | 资源ID       |
|--------------------------|----------|-------------------|-----------|-------------|
| //mail.googleapis.com    | /users   | /name@example.com | /settings | /customFrom |

API 设计者可以为资源和集合 ID 选择任何可接受的值，只要它们在资源层次结构中是唯一的即可。你可以在下面找到有关选择适当资源和集合 ID 的更多指南。

## 完整资源名

无协议（scheme-less） [URI](http://tools.ietf.org/html/rfc3986) 由[兼容 DNS](http://tools.ietf.org/html/rfc1035) 的 API 服务名和资源路径组成。资源路径也称为相对资源名。例如：

```
"//library.googleapis.com/shelves/shelf1/books/book2"
```

API 服务名用于客户端定位 API 服务端点，如果只为内部服务，它可以（may）是假的 DNS 名。如果 API 服务名在上下文中显而易见的话则会经常使用相对资源名。

## 相对资源名

没有斜杠（`/`）开头的 [URI 路径](http://tools.ietf.org/html/rfc3986#appendix-A)标识了 API 服务中的资源。例如：

```
"shelves/shelf1/books/book2"
```

## 资源 ID

使用非空的 [URI 段](http://tools.ietf.org/html/rfc3986#appendix-A)标识其父资源中的资源。请看上面的例子。

资源名称后面跟随的资源 ID **可以（MAY）** 具有不只一个 URI 段，例如：

| 集合ID | 资源ID               |
|-------|----------------------|
| files | /source/py/parser.py |

如果可以的话，API 服务应该（should）使用 URL 友好的资源 ID。资源 ID 必须（must） 明确地记录在文档中，不管它们是由客户端还是服务端分配的。例如，文件名一般由客户端分配，而邮件信息 ID 一般由服务端分配。

## 集合 ID

使用非空的 [URI 段](http://tools.ietf.org/html/rfc3986#appendix-A) 标识其父资源中的资源集合。请看上面的例子。

因为集合 ID 经常出现在生成的客户端库中，它们 **必须（MUST）** 符合以下要求：

- **必须（MUST）** 是合法的 C/C++ 标识符
- **必须（MUST）** 是复数形式的首字母小写的驼峰命名
- **必须（MUST）** 使用清晰简明的英语词汇
- **应该（SHOULD）** 避免或限定过于笼统的术语。例如：`RowValue` 优于 `Value`。除非明确定义，否则 **应该（SHOULD）** 避免使用如下术语：
    - Element
    - Entry
    - Instance
    - Item
    - Object
    - Resource
    - Type
    - Value

## 资源名 vs URL

完整的资源名类似普通的 URL，但它们并不相同。同样的资源能够通过不同版本或不同协议的 API 来暴露出去。完整的资源名并没有指定这些信息，所以必须将它映射到特定的协议和 API 版本上才能直接地使用。

为了通过 REST API 使用完整的资源名，**必须（MUST）** 使用如下方法将其映射为 REST URL：在服务名前添加 HTTPS 协议、在资源路径前添加 API 主版本号、将资源路径进行 URL 转义。例如：

```
// 这是日历事件的资源名
"//calendar.googleapis.com/users/john smith/events/123"
// 这是对应的 HTTP URL
"https://calendar.googleapis.com/v3/users/john%20smith/events/123"
```

## 资源名做为字符串

除非有向后兼容的问题，Google API **必须（MUST）** 使用字符串来表示资源名。资源名 **应该(SHOULD)** 像普通文件路径那样处理，并且不支持[百分号编码](https://zh.wikipedia.org/zh-hans/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81)。

对于资源定义，第一个字段 **应该(SHOULD)** 是资源名称的字符串字段，它 **应该(SHOULD)** 叫作 name。

注意：像 `display_name`、`first_name`、`last_name`、`full_name` 这种与名字相关的字段 **应该(SHOULD)** 给出定义来避免混乱。

例子：

```
service LibraryService {
  rpc GetBook(GetBookRequest) returns (Book) {
    option (google.api.http) = {
      get: "/v1/{name=shelves/*/books/*}"
    };
  };
  rpc CreateBook(CreateBookRequest) returns (Book) {
    option (google.api.http) = {
      post: "/v1/{parent=shelves/*}/books"
      body: "book"
    };
  };
}
message Book {
  // book 的资源名。必须是"shelves/*/books/*"的格式
  // 例如："shelves/shelf1/books/book2".
  string name = 1;
  // ... 其他属性
}
message GetBookRequest {
  // 一个 book 的资源名。例如："shelves/shelf1/books/book2"
  string name = 1;
}
message CreateBookRequest {
  // 创建 book 的父资源名
  // 例如："shelves/shelf1"
  string parent = 1;
  // 被创建的 book 资源。客户端一定不能（must not）设置 `Book.name` 字段
  Book book = 2;
}
```

提示：为了资源名称的一致性，开始的斜杠 **一定不能（MUST NOT）** 被 URL 模版变量捕获。例如，**必须（MUST）** 使用 URL 模版 `/v1/{name=shelves/*/books/*}` 而不是 `/v1{name=/shelves/*/books/*}`。