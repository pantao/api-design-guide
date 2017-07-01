# 目录结构

通常使用 `.proto` 文件定义 API，使用 `.yaml` 文件做为配置。

每个 API 服务 **必须（must）** 有一个 API 目录来存放定义文件和构建脚本。

API 目录 **应该（should）** 遵循如下的标准结构：

- API 目录
- 配置文件
    - `{service}.yaml`：主服务的配置文件，`google.api.Service` 的 YAML 格式
    - `prod.yaml`：产品环境配置文件
    - `staging.yaml`：Staging 环境配置文件
    - `test.yaml`：测试环境配置文件
    - `local.yaml`：本地环境配置文件
- 接口定义
    - `v[0-9]*/*`：每一个子目录包含 API 的一个主版本，主要存放原型文件和构建脚本
    - `{subapi}/v[0-9]*/*`：每一个 `{subapi}` 目录包含子 API 的接口定义。每个子 API 可以有它独立的主版本号
    - `type/*`： 包含类型定义的原型文件，包括这些：在不同 API 间共享的类型、不同 API 版本间共享的类型或 API 与服务实现间共享的类型。一旦发布，`type/*` 中定义的类型 **不应该（should not）** 有破坏兼容性的修改。