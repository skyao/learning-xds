---
title: "LDS API中的Listener Filter配置"
linkTitle: "Listener Filter配置"
weight: 463
date: 2021-10-09
description: >
  LDS API中的Listener Filter配置
---

Listener Filter的配置详细定义在 xDS API 中 Linsenter Components 的 proto 定义文件中，地址如下：

https://github.com/envoyproxy/envoy/blob/main/api/envoy/config/listener/v3/listener_components.proto#L336

### Json格式

Listerner Filter Chain配置的JSON格式如下所示：

```json
{
  "name": "...",
  "typed_config": "{...}",
  "filter_disabled": "{...}"
}
```

### proto格式

proto 文件定义如下:

```protobuf
message ListenerFilter {
  option (udpa.annotations.versioning).previous_message_type =
      "envoy.api.v2.listener.ListenerFilter";

  string name = 1 [(validate.rules).string = {min_len: 1}];

  oneof config_type {
    google.protobuf.Any typed_config = 3;
  }

  ListenerFilterChainMatchPredicate filter_disabled = 4;
}
```

### 字段说明

具体字段的说明：

| 字段            | 格式                              | 说明                                                         |
| --------------- | --------------------------------- | ------------------------------------------------------------ |
| name            | string                            | 要实例化的过滤器的名称。该名称必须符合支持的过滤器。         |
| typed_config    | google.protobuf.Any               | 过滤器的具体配置，取决于正在实例化的过滤器。参见支持的过滤器的进一步文件。 |
| filter_disabled | ListenerFilterChainMatchPredicate | 用于禁用过滤器的可选匹配谓词。当这个字段为空时，过滤器被启用。更多例子见ListenerFilterChainMatchPredicate。 |

如果是通过 typed_config 来配置 filter，则 name 必须是已经被支持的 filter，具体目前支持的filter列表见：

https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/listener_filters#config-listener-filters
