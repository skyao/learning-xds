---
title: "LDS API中的Listener配置"
linkTitle: "Listener配置"
weight: 462
date: 2021-09-28
description: >
  LDS API中的Listener配置
---


## Listener配置

Listener 的配置详细定义在 xDS API 中 Linsenter 的 proto 定义文件中，地址如下：

https://github.com/envoyproxy/envoy/blob/main/api/envoy/config/listener/v3/listener.proto#L39

### Json格式

Listerner配置的JSON格式如下所示：

```json
{
  "name": "...",
  "address": "{...}",
  "stat_prefix": "...",
  "filter_chains": [],
  "use_original_dst": "{...}",
  "default_filter_chain": "{...}",
  "per_connection_buffer_limit_bytes": "{...}",
  "metadata": "{...}",
  "drain_type": "...",
  "listener_filters": [],
  "listener_filters_timeout": "{...}",
  "continue_on_listener_filters_timeout": "...",
  "transparent": "{...}",
  "freebind": "{...}",
  "socket_options": [],
  "tcp_fast_open_queue_length": "{...}",
  "traffic_direction": "...",
  "udp_listener_config": "{...}",
  "api_listener": "{...}",
  "connection_balance_config": "{...}",
  "reuse_port": "...",
  "enable_reuse_port": "{...}",
  "access_log": [],
  "tcp_backlog_size": "{...}",
  "bind_to_port": "{...}"
}
```

### proto格式

proto 文件定义如下:

```protobuf
message Listener {
  option (udpa.annotations.versioning).previous_message_type = "envoy.api.v2.Listener";

  string name = 1;

  core.v3.Address address = 2 [(validate.rules).message = {required: true}];

  string stat_prefix = 28;

  repeated FilterChain filter_chains = 3;

  google.protobuf.BoolValue use_original_dst = 4;

  FilterChain default_filter_chain = 25;

  google.protobuf.UInt32Value per_connection_buffer_limit_bytes = 5
      [(udpa.annotations.security).configure_for_untrusted_downstream = true];

  core.v3.Metadata metadata = 6;

  DrainType drain_type = 8;

  repeated ListenerFilter listener_filters = 9;

  google.protobuf.Duration listener_filters_timeout = 15;

  bool continue_on_listener_filters_timeout = 17;

  google.protobuf.BoolValue transparent = 10;

  google.protobuf.BoolValue freebind = 11;

  repeated core.v3.SocketOption socket_options = 13;

  google.protobuf.UInt32Value tcp_fast_open_queue_length = 12;

  core.v3.TrafficDirection traffic_direction = 16;

  UdpListenerConfig udp_listener_config = 18;

  ApiListener api_listener = 19;

  ConnectionBalanceConfig connection_balance_config = 20;

  google.protobuf.BoolValue enable_reuse_port = 29;

  repeated accesslog.v3.AccessLog access_log = 22;

  google.protobuf.UInt32Value tcp_backlog_size = 24;

  google.protobuf.BoolValue bind_to_port = 26;
}
```

### Listener字段说明

具体字段的说明：

| 字段                                 | 格式                        | 说明                                                         |
| ------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| name                                 | string                      | 该监听器的唯一名称。如果没有提供名称，Envoy将为监听器分配一个内部UUID。如果监听器要通过LDS动态更新或删除，必须提供唯一的名字。 |
| address                              | core.v3.Address, REQUIRED   | 听众应该收听的地址。一般来说，这个地址必须是唯一的，不过这要由操作系统的绑定规则决定。例如，多个监听器可以在Linux上监听端口0，因为实际的端口将由操作系统分配。 |
| stat_prefix                          | string                      | 可选的前缀，用于监听器的统计信息。如果为空，统计信息将以 `listener.<address as string>` 为根。如果非空，统计信息将以`listener.<stat_prefix>`为根。 |
| filter_chains                        | FilterChain[], REQUIRED     | 考虑用于这个监听器的过滤链列表。最符合 FilterChainMatch 条件的 FilterChain 将用于连接。 |
| use_original_dst                     | google.protobuf.BoolValue   | 如果使用iptables重定向连接，代理接收连接的端口可能与原始目标地址不同。当这个标志被设置为 "true" 时，监听器会将重定向的连接交给与原始目标地址相关的监听器。如果没有与原始目标地址相关联的监听器，连接将由接收它的监听器处理。默认为false。 |
| default_filter_chain                 | FilterChain                 | 如果没有一个过滤器链相匹配，则为默认过滤器链。如果没有提供默认的过滤链，连接将被关闭。在这个字段中，过滤链的匹配被忽略。 |
| per_connection_buffer_limit_bytes    | UInt32Value                 | 对监听器新建连接的读写缓冲区大小的软限制。如果未指定，则使用实现定义的默认值（1MiB）。 |
| metadata                             | core.v3.Metadata            | 监听器元数据                                                 |
| drain_type                           | DrainType                   | 在监听器范围内执行的逐出类型。                               |
| listener_filters                     | ListenerFilter[]            | Listener 过滤器有机会操作和增加连接元数据，例如在连接过滤器链匹配中使用。这些过滤器会在任何过滤器链之前运行。顺序很重要，因为过滤器是在监听器接受套接字后，在创建连接前，按顺序处理的。当监听器套接字地址中的协议是UDP时，可以指定UDP监听器过滤器。UDP监听器目前只支持一个过滤器。 |
| listener_filters_timeout             | google.protobuf.Duration    | 等待所有监听器过滤器完成操作的超时时间。如果达到超时，接受的套接字将被关闭，而不创建连接，除非`continue_on_listener_filters_timeout`被设置为true。指定0来禁用超时。如果不指定，将使用默认的15s超时。 |
| continue_on_listener_filters_timeout | bool                        | 当监听器过滤器超时时，是否应该创建连接。默认为false。        |
| transparent                          | google.protobuf.BoolValue   | 监听器是否应该被设置为透明套接字。<br /><br />当这个标志设置为 "true "时，连接可以使用iptables TPROXY目标重定向到监听器，在这种情况下，接受的连接会保留原来的源地址和目的地址以及端口。这个标志应该与 original_dst 监听器过滤器结合使用，以标记连接的本地地址为 "restored"。这可以用来把每个重定向的连接移交给与该连接的目的地址相关的另一个监听器。没有使用TPROXY的套接字的直接连接无法与使用TPROXY重定向的连接区分开来，因此会被当作重定向的连接。当这个标志被设置为false时，监听器的套接字会被明确地重置为非透明的。设置这个标志需要Envoy以CAP_NET_ADMIN能力运行。当这个标志没有设置时（默认），套接字不会被修改，也就是说，透明选项既不会被设置也不会被重置。 |
| freebind                             | google.protobuf.BoolValue   | 监听器是否应设置 IP_FREEBIND 套接字选项。 当此标志设置为true时，可以将监听器绑定到未在运行Envoy的系统上配置的IP地址。 当此标志设置为false时，套接字上的选项 IP_FREEBIND 被禁用。 如果未设置此标志（默认值），则不会修改套接字，即既未启用也未禁用该选项。 |
| socket_options                       | core.v3.SocketOption        | Envoy源代码或预编译二进制文件中可能不存在的其他套接字选项。  |
| tcp_fast_open_queue_length           | UInt32Value                 | 监听器是否应接受 TCP Fast Open（TFO）连接。 当此标志设置为大于0的值时，将在套接字上启用选项TCP_FASTOPEN，其队列长度为指定大小（请参阅RFC7413中的详细信息）。 当此标志设置为0时，套接字上禁用选项TCP_FASTOPEN。 如果未设置此标志（默认值），则不会修改套接字，即既未启用也未禁用该选项。 |
| traffic_direction                    | core.v3.TrafficDirection    | 指定流量相对于本地 envoy 的预期方向。对于使用原始目的地过滤器的 listener 来说，这个属性在Windows下是必需的，见原始目的地 |
| udp_listener_config                  | UdpListenerConfig           | 如果协议中的监听器套接字地址的协议是UDP，这个字段指定UDP监听器的具体配置。 |
| api_listener                         | ApiListener                 | 用于表示一个API监听器，在非代理客户端中使用。暴露给非代理应用程序的API类型取决于API监听器的类型。当这个字段被设置时，除了名字之外，其他字段都不应该被设置。 |
| connection_balance_config            | ConnectionBalanceConfig     | 监听器的连接平衡器配置，目前只适用于TCP监听器。如果没有指定配置，Envoy将不会尝试在工作线程之间平衡活动连接。<br/><br/>在这种情况下，监听器X通过在X中设置use_original_dst，在Y1和Y2中设置bind_to_port为false，将所有连接重定向到监听器Y1和Y2，建议禁用监听器X中的平衡配置，以避免平衡的成本，并启用Y1和Y2中的平衡配置，在工作者之间平衡连接。 |
| reuse_port                           | bool                        | 弃用。请使用 enable_reuse_port                               |
| enable_reuse_port                    | google.protobuf.BoolValue   | 当这个标志被设置为 "true" 时，监听器会设置SO_REUSEPORT套接字选项，为每个工作线程创建一个套接字。这使得在连接数较多的情况下，入站连接在工作线程之间大致均匀分布。当这个标志被设置为false时，所有工作线程共享一个套接字。这个字段的默认值是true。 |
| access_log                           | accesslog.v3.AccessLog[]    | 该监听器发出的访问日志的配置。                               |
| tcp_backlog_size                     | google.protobuf.UInt32Value | tcp 监听器的待处理连接队列可以增长的最大长度。如果没有提供net.core.somaxconn的值，在Linux上将会使用，否则将使用128。 |
| bind_to_port                         | google.protobuf.BoolValue   | 监听器是否应该与端口绑定。不绑定的监听器只能接收从其他设置use_original_dst为true的监听器重定向的连接。默认为true。 |



