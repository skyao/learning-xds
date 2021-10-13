---
title: "LDS API中的Filter配置"
linkTitle: "Network Filter配置"
weight: 467
date: 2021-10-08
description: >
  LDS API中的Filter配置
---

Listener Filter的配置详细定义在 xDS API 中 Linsenter Components 的 proto 定义文件中，地址如下：

https://github.com/envoyproxy/envoy/blob/main/api/envoy/config/listener/v3/listener_components.proto#L28

### Json格式

Listerner Filter配置的JSON格式如下所示：

```json
{
  "name": "...",
  "typed_config": "{...}"
}
```

### proto格式

proto 文件定义如下:

```protobuf
message Filter {

  string name = 1 [(validate.rules).string = {min_len: 1}];

  oneof config_type {

    google.protobuf.Any typed_config = 4;

    core.v3.ExtensionConfigSource config_discovery = 5;
  }
}
```

### Listener字段说明

具体字段的说明：

| 字段             | 格式                          | 说明                                                         |
| ---------------- | ----------------------------- | ------------------------------------------------------------ |
| name             | string                        | 要实例化的过滤器的名称。该名称必须符合支持的过滤器           |
| typed_config     | google.protobuf.Any           | 过滤器的具体配置，取决于正在实例化的过滤器。参见支持的过滤器的进一步的文档。 |
| config_discovery | core.v3.ExtensionConfigSource | 扩展配置发现服务的配置源指定器。在失败和没有默认配置的情况下，监听器会关闭连接。 |

如果是通过 typed_config 来配置 filter，则 name 必须是已经被支持的 filter，具体目前支持的filter列表见：

https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/network_filters#config-network-filters

