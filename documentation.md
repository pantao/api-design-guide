# 文档

这一章是为 API 添加内部文档的指南。大部分 API 有概述、教程和更高级别的参考文档（此指南不讨论）。API 名、资源名和方法名的信息请查看[命名约定](name-conventions.md)。

## 注释格式

在 `.proto` 文件中使用 `//` 来添加注释。

```
// Creates a shelf in the library, and returns the new Shelf.
rpc CreateShelf(CreateShelfRequest) returns (Shelf) {
  option (google.api.http) = { post: "/v1/shelves" body: "shelf" };
}
```

## 在服务配置中添加注释

作为在 `.proto` 文件中添加注释的替代方法，你可以在 API 的 YAML 服务配置文件中添加注释。如果两个文件中都记录了相同的元素，则该文件中的文档将优先于 `.proto` 中的文档

```
documentation:
  summary: Gets and lists social activities
  overview: A simple example service that lets you get and list possible social activities
  rules:
  - selector: google.social.Social.GetActivity
    description: Gets a social activity. If the activity does not exist, returns Code.NOT_FOUND.
...
```

当在一个 `.proto` 文件中有多个服务并且要提供特定于服务的文档时，你就需要上面的方法。可以在 YAML 中添加 `overview` 详细记录 API 的描述信息。但是，一般推荐将注释添加到 `.proto`。

与 `.proto` 注释一样，可以在 YAML 文件的注释中使用 `Markdown` 来提供其他格式。

## API 描述

API 描述是一个描述此 API 能做什么，以动词开始的句子。在 .proto 文件中，API 描述以注释的方法添加到对应的 service 上，例如：

```
// Manages books and shelves in a simple digital library.
service LibraryService {
...
}
```

API 描述的其他示例：

- Shares updates, photos, videos, and more with your friends around the world.
- Accesses a cloud-hosted machine learning service that makes it easy to build smart apps that respond to streams of data.

## 资源描述

顾名思义资源描述用来描述资源代表的什么东西。在 .proto 文件中，资源描述作为注释添加到对应消息上，例如：

```
// A book resource in the Library API.
message Book {
  ...
}
```

资源描述的其他示例：

- A task on the user’s to-do list. Each task has a unique priority.
- An event on the user’s calendar.

## 字段和参数描述

用于描述字段和参数的定义，一些示例：

- The number of topics in this series.
- The accuracy of the latitude and longitude coordinates, in meters. Must be non-negative.
- Flag governing whether attachment URL values are returned for submission resources in this series. The default value for series.insert is true.
- The container for voting information. Present only when voting information is recorded.
- Not currently used or deprecated.

字段和参数描述:

- 必须清楚地描述边界条件（描述哪些值有效哪些值无效。API 用户会尽最大的可能来误用服务，而且不能阅读底层代码来弄清楚）
- 必须指定默认值和默认行为
- 当是字符串时，要描述语法、允许的字符和需要的编码格式，例如
    - 1-255 characters in the set [A-a0-9]
    - A valid URL path string starting with / that follows the RFC 2332 conventions. Max length is 500 characters.
- 如果可以的话，提供示例
- 当字段 **必须**、**仅输入**、**仅输出** 时，必须在字段描述的开始进行说明。默认情况下所有字段和参数都是可选的。例如：

```
message Table {
  // Required. The resource name of the table.
  string name = 1;
  // Input only. Whether to dry run the table creation.
  bool dryrun = 2;
  // Output only. The timestamp when the table was created. Assigned by
  // the server.
  Timestamp create_time = 3;
  // The display name of the table.
  string display_name = 4;
}
```

## 方法描述

方法描述用来说明方法的作用和操作的资源。通常以第三人称的现在时[动词](https://developers.google.com/internal/style/reference-verbs) 开始。示例：

- Lists calendar events for the authenticated user.
- Updates a calendar event with the data included in the request.
- Deletes a location record from the authenticated user’s location history.
- Creates or updates a location record in the authenticated user’s location history using the data included in the request. If a location resource already exists with the same timestamp value, the data provided overwrites the existing data.

## 注意事项

确保描述简洁完整并且容易被理解。不能仅做重述，例如 `series.insert` 方法的描述不能简单地写成 “Inserts a series”。命名应该包含丰富的信息，大多数读者会阅读描述，因为他们需要比名字本身更多的信息。如果不知道在描述中要写什么，试着回答下面的问题：

- 它是什么
- 成功或失败后会导致什么。什么会导致失败。
- 是幂等的吗
- 单位是什么（例如：米、度、像素）
- 接收参数的范围
- 副作用是什么
- 如何使用
- 常见错误是什么
- 是否总是存在（例如：”Container for voting information. Present only when voting information is recorded.”）
- 是否有默认值

## 惯例

本节列出了文本描述和文档的一些使用惯例。使用 `ID` 表示标识符（而不使用 `Id` 或 `id`）。使用 `JSON` 表示数据格式（而不使用 `Json` 或 `json`）。所有字段/参数使用 `code font` 的格式，字符串要用引号括起来。

- ID
- JSON
- RPC
- REST
- `property_name` 或 `"string_literal"`
- `true` / `false`

## 语言风格

和[命名约定](name-conventions.md)一样，在写注释时推荐使用简洁一致的词语和风格，让母语非英语的读者容易理解。因此要避免行话、俚语、复杂的隐喻、流行文化或其他不容易理解的内容。使用友好专业的风格直接与读者“交谈”，并尽可能保持注释的简洁。请记住，大部分读者只想知道如何使用 API，而不是阅读你的文档！