# 版本控制

这一章是网络 API 的版本控制指南。因为一个 API 服务 可以（may） 提供多个 [API 接口](glossary.md)，[API 版本](glossary.md)策略应用在 API 接口上而不是 [API 服务](glossary.md)。为了方便，下面的 API 表示 API 接口。

网络 API **应该（should）** 使用 Semantic Versioning。对于版本号 `MAJOR.MINOR.PATCH`：

- 有不兼容的升级时，增加 `MAJOR`
- 添加了能向后兼容的新功能时，增加 `MINOR`
- 修改了能向后兼容的 BUG 时，增加 `PATCH`

根据 API 版本的不同，`major` 版本号使用不同的规则：

- 对于 version 1(v1)，major 部分 **应该（should）** 加上 proto 的包名字，例如 `google.pubsub.v1`。如果包名包含稳定的类型并且接口不会有不兼容的改变，major 部分 可以（may） 忽略版本号，例如：`google.protobuf` 和 `google.longrunning`。
- 对于除 v1 外的所有版本，major 版本号 **必须（must）** 加上 proto 的包名字。例如 `google.pubsub.v2`。

对于 pre-GA 的发布（例如 alpha 和 beta），推荐在版本号中添加后缀，后缀 **应该（should）** 以 pre-release 的版本名（例如 alpha、beta）和可选的 pre-release 版本号组成。

版本进度的例子：

Version | Proto Package | Description
-|-|-
v1alpha | v1alpha1 | v1 alpha 发布
v1beta1 | v1beta1 | v1 beta 第一次发布
v1beta2 | v1beta2 | v1 beta 第二次发布
v1test | v1test | 带有假数据的内部测试版
v1 | v1 | major 的版本是 v1，可正式使用
v1.1beta1 | v1p1beta1 | 对 v1 版的首次小版本（minor）修改的 beta 发布
v1.1 | v1 | 小版本升级到 v1.1
v2beta1 | v2beta1 | v2 beta 第一次发布
v2 | v2 | major 的版本是 v2，可正式使用

minor 和 patch 的版本号 **应该（should）** 表现在 API 配置和文档中，**一定不要（must not）** 写在 proto 的包名中。

**注意**：Google API 平台目前没有原生支持 minor 和 patch。对于每一个 major 版本只有一套文件和客户端的库。API 作者需要通过文档和发布日志手动记录 minor 和 patch。

新的 major 版本号 **一定不要（must not）** 依赖 相同 API 之前的 major 版本。在了解相关联的依赖性和稳定性风险后，API 可以（may） 依赖其他 API。一个稳定的 API 版本 **必须（must）** 只依赖其他 API 的最新稳定版本。

在一段时间内，相同 API 的不同版本 **必须（must）** 在单个客户端中同时工作。这样才能帮助客户端从旧版 API 平滑迁移到新版 API。

只有当没有依赖后旧版本 API 才能被删除。

被多个 API 共享的通用稳定的数据类型（例如日期和时间） **应该（should）** 定义在单独的 proto 包中。如果有必要进行不兼容的修改，则 **必须（must）** 引入新的类型或包含新 major 版本的包。

## 向后兼容

定义什么是向后兼容的修改是比较困难的。

下面列出了一些，但如果你有任何疑问，点击这里查看详情。

### 保持向后兼容的修改

- 向 API 服务中添加 API 接口
- 向 API 接口中添加方法
- 向方法添加 HTTP 绑定
- 向请求信息添加字段
- 向响应信息添加字段
- 向枚举添加值
- 添加只输出（output-only）的资源字段

### 破坏向后兼容的修改

- 删除/重命名服务、接口、字段名、方法或枚举值
- 修改 HTTP 绑定
- 修改字段类型
- 修改资源名的格式
- 修改已有请求的可见性（visible behavior）
- 在 HTTP 定义中修改 URL 格式
- 在资源消息中添加读/写字段