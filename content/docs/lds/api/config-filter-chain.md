---
title: "LDS API中的Filter Chain配置"
linkTitle: "Filter Chain配置"
weight: 424
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

## FilterChainMatch

指定为 Listener 选择特定过滤链的匹配判据。

为了选择一个过滤链，它的所有判据必须被传入的连接满足，其属性由网络堆栈和/或监听器过滤器设置。

以下顺序适用:

- 目的地端口。
- 目的地IP地址。
- 服务器名称（例如：TLS协议的SNI）。
- 传输协议。
- 应用协议（例如：TLS协议的ALPN）。
- 直接连接的源IP地址（这只有在使用覆盖源地址的监听器过滤器时才会与源IP地址不同，如代理协议监听器过滤器）。
- 源类型（例如，任何，本地或外部网络）。
- 源IP地址。
- 源端口

对于允许范围或通配符的标准，将使用任何配置的过滤链中与传入连接相匹配的最具体的值（例如，对于SNI www.example.com，最具体的匹配将是www.example.com，然后是*.example.com，然后是*.com，然后是任何没有server_names要求的过滤链）。

用另一种方式来推理过滤链的匹配。假设存在N个过滤链。使用上述8个步骤对过滤链集进行修剪。在每个步骤中，与属性最具体匹配的过滤链继续进入下一个步骤。监听器保证在所有的步骤之后最多剩下1个过滤链。

例如：

对于目的端口，指定传入流量的目的端口的过滤链是最具体的匹配。如果没有一个过滤链指定确切的目的端口，那么不指定端口的过滤链是最具体的匹配。指定错误端口的过滤链永远不可能成为最具体的匹配。

Listener Filter Chain Match的配置详细定义在 xDS API 中 Linsenter Components 的 proto 定义文件中，地址如下：

https://github.com/envoyproxy/envoy/blob/main/api/envoy/config/listener/v3/listener_components.proto#L97

### Json格式

Listerner Filter Chain Match配置的JSON格式如下所示：

```json
{
  "destination_port": "{...}",
  "prefix_ranges": [],
  "direct_source_prefix_ranges": [],
  "source_type": "...",
  "source_prefix_ranges": [],
  "source_ports": [],
  "server_names": [],
  "transport_protocol": "...",
  "application_protocols": []
}
```

### proto格式

proto 文件定义如下:

```protobuf
message FilterChainMatch {
  option (udpa.annotations.versioning).previous_message_type =
      "envoy.api.v2.listener.FilterChainMatch";

  google.protobuf.UInt32Value destination_port = 8 [(validate.rules).uint32 = {lte: 65535 gte: 1}];

  repeated core.v3.CidrRange prefix_ranges = 3;

  string address_suffix = 4;

  google.protobuf.UInt32Value suffix_len = 5;

  repeated core.v3.CidrRange direct_source_prefix_ranges = 13;

  ConnectionSourceType source_type = 12 [(validate.rules).enum = {defined_only: true}];

  repeated core.v3.CidrRange source_prefix_ranges = 6;

  repeated uint32 source_ports = 7
      [(validate.rules).repeated = {items {uint32 {lte: 65535 gte: 1}}}];

  repeated string server_names = 11;

  string transport_protocol = 9;

  repeated string application_protocols = 10;
}
```

### 字段说明

具体字段的说明：

| 字段                        | 格式                        | 说明                                                         |
| --------------------------- | --------------------------- | ------------------------------------------------------------ |
| destination_port            | google.protobuf.UInt32Value | 当监听器上设置use_original_dst时，在确定过滤链匹配时要考虑的可选目标端口。 |
| prefix_ranges               | core.v3.CidrRange[]         | 如果非空，则是一个IP地址和前缀长度，以便在监听器被绑定到0.0.0.0/::或指定use_original_dst时匹配地址。 |
| direct_source_prefix_ranges | core.v3.CidrRange[]         | 如果下游连接的直接连接源IP地址包含在至少一个指定的子网中，则满足该条件。如果没有指定参数或者列表为空，直接连接的源IP地址将被忽略。 |
| source_type                 | ConnectionSourceType        | 指定连接源IP匹配类型。可以是任何，本地或外部网络。           |
| source_prefix_ranges        | core.v3.CidrRange[]         | 如果下游连接的源IP地址包含在至少一个指定的子网中，则满足该条件。如果没有指定参数或列表为空，则源IP地址被忽略。 |
| source_ports                | uint32[]                    | 如果下游连接的源端口包含在至少一个指定的端口中，则满足该条件。如果没有指定该参数，源端口将被忽略。 |
| server_names                | string[]                    | 如果非空，则是一个服务器名称的列表（例如TLS协议的SNI），在确定过滤器链匹配时要考虑。这些值将与新连接的服务器名称进行比较，当被一个监听器过滤器检测到时。<br/><br/>服务器名称将与所有通配符域名进行匹配，即www.example.com，然后是*.example.com，然后是*.com。<br/><br/>注意，不支持部分通配符，像*w.example.com这样的值是无效的。 |
| transport_protocol          | string                      | 如果非空，在确定过滤器链匹配时要考虑的传输协议。这个值将与新连接的传输协议进行比较，当它被一个监听器过滤器检测到时。<br/><br/>建议的值包括:<br/><br/>- raw_buffer - 默认值，在没有检测到传输协议时使用。<br/><br/>- tls - 当检测到TLS协议时由envoy.filters.listener.tls_inspector设置。 |
| application_protocols       | string[]                    | 如果非空，则是一个应用协议的列表（例如，TLS协议的ALPN），在确定过滤器链的匹配时要考虑。这些值将与新连接的应用协议进行比较，当被一个监听器过滤器检测到时。<br/><br/>建议的值包括。<br/><br/>- http/1.1 - 由 envoy. filters.listener.tls_inspector 设置。<br/><br/>- h2 - 由 envoy.filters.listener.tls_inspector 设置。 |

