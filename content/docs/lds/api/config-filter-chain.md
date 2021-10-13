---
title: "LDS API中的Filter Chain配置"
linkTitle: "Filter Chain配置"
weight: 464
date: 2021-10-08
description: >
  LDS API中的Filter Chain配置
---

## Filter Chain

过滤器链包裹着一组匹配判据，一个选项TLS上下文，一组过滤器，以及其他各种参数。

Listener Filter Chain的配置详细定义在 xDS API 中 Linsenter Components 的 proto 定义文件中，地址如下：

https://github.com/envoyproxy/envoy/blob/main/api/envoy/config/listener/v3/listener_components.proto#L201

### Json格式

Listerner Filter Chain配置的JSON格式如下所示：

```json
{
  "filter_chain_match": "{...}",
  "filters": [],
  "use_proxy_proto": "{...}",
  "transport_socket": "{...}",
  "transport_socket_connect_timeout": "{...}"
}
```

### proto格式

proto 文件定义如下:

```protobuf
message FilterChain {
  option (udpa.annotations.versioning).previous_message_type = "envoy.api.v2.listener.FilterChain";

  FilterChainMatch filter_chain_match = 1;

  repeated Filter filters = 3;

  google.protobuf.BoolValue use_proxy_proto = 4
      [deprecated = true, (envoy.annotations.deprecated_at_minor_version) = "3.0"];

  core.v3.Metadata metadata = 5;

  core.v3.TransportSocket transport_socket = 6;

  google.protobuf.Duration transport_socket_connect_timeout = 9;

  string name = 7;

  OnDemandConfiguration on_demand_configuration = 8;
}
```

### 字段说明

具体字段的说明：

| 字段                             | 格式                      | 说明                                                         |
| -------------------------------- | ------------------------- | ------------------------------------------------------------ |
| filter_chain_match               | FilterChainMatch          | 匹配连接到该过滤链时使用的判据。                             |
| filters                          | Filter                    | 一个单独的网络过滤器的列表，它构成了与监听器建立的连接的过滤器链。顺序很重要，因为过滤器会随着连接事件的发生按顺序处理。注意：如果过滤器列表为空，连接将默认关闭。 |
| use_proxy_proto                  | google.protobuf.BoolValue | 监听器是否应该在新连接上期待一个PROXY协议V1头。如果该选项被启用，监听器将假定连接的远程地址是头中指定的地址。包括AWS ELB在内的一些负载均衡器支持这个选项。如果该选项不存在或设置为false，Envoy将使用连接的物理对等地址作为远程地址。<br/><br/>这个字段已经废弃了。可以明确地添加PROXY协议监听器过滤器来代替。 |
| metadata                         | core.v3.Metadata          | 过滤链元数据                                                 |
| transport_socket                 | core.v3.TransportSocket   | 可选的自定义传输套接字实现，用于下游连接。要设置TLS，在typed_config中设置一个名称为envoy.transport_sockets.tls的传输套接字和DownstreamTlsContext。如果没有指定传输套接字的配置，新的连接将用纯文本建立。 |
| transport_socket_connect_timeout | google.protobuf.Duration  | 如果存在并且不为零，允许传入的连接完成任何传输套接字协商的时间量。如果这个时间在传输报告连接建立之前过期，连接将被立即关闭。 |

