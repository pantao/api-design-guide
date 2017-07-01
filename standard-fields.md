# 标准字段

本节介绍了在需要类似概念时应使用的一组标准消息字段定义，这将确保相同的概念在不同的 API 中具有相同的名称和语义。

名称 | 类型 | 描述
-|-|-
name | string | name 字段应该包含相对资源名称
parent | string | 对于资源定义和 `List`/`Create` 请求，`parent` 字段应该包含父资源的[相对资源名称](resource-names.md)
create_time | Timestamp | 资源创建的时间戳
update_time | Timestamp | 资源最后更新的时间戳（当执行 create/patch/delete 操作时更新）
delete_time | Timestamp | 资源删除的时间戳，只有支持恢复时有效
time_zone | string | 时区名，必须是一个像 “America/Los_Angeles” 的 [IANA TZ](http://www.iana.org/time-zones) 名字。更多信息请参考 [https://en.wikipedia.org/wiki/List_of_tz_database_time_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
region_code | string | 国家地区编码（CLDR），更多信息请参考 [http://www.unicode.org/reports/tr35/#unicode_region_subtag](http://www.unicode.org/reports/tr35/#unicode_region_subtag)
language_code | string | BCP-47 语言编码例如“en-US”或“sr-Latn”，更多信息请参考 [http://www.unicode.org/reports/tr35/#Unicode_locale_identifier](http://www.unicode.org/reports/tr35/#Unicode_locale_identifier)
display_name | string | 资源的显示名称
title | string | 资源的官方名称，例如公司名称。它应该被看做是 `display_name` 的正式名称
description | string | 资源的描述信息
filter | string | List 方法的标准过滤字段
query | string | 在 [search](custom-methods.md) 方法中使用，功能与 `filter` 相同
page_token | string | List 请求中的分页字段
page_size | int32 | List 请求中一页的大小
total_size | int32 | 资源总数量
next_page_token | string | List 响应中的下一页，用于下一次请求的 `page_token` 字段
resume_token | string | 用于恢复流请求的 `opaque token`
labels | map | 表示云资源标签
deleted | bool | 如果资源支持恢复，它必须具有 `deleted` 标签来标记资源是否被删除
show_deleted | bool | 如果资源支持恢复，对应的 `List` 方法必须有 `show_deleted` 字段来表示是否展示被删除的资源
update_mask | FieldMask | 用于在 `Update` 请求中对资源进行部分更新
validate_only | bool | 为 `true` 时表示请求只验证而不会执行