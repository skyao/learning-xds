---
title: "[译]Envoy中的Network (L3/L4) filters"
linkTitle: "[译]Network (L3/L4) filters"
weight: 425
date: 2021-10-08
description: >
  翻译Envoy中的Network (L3/L4) filters介绍内容
---

https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/network_filters

> As discussed in the [listener](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listeners#arch-overview-listeners) section, network level (L3/L4) filters form the core of Envoy connection handling. The filter API allows for different sets of filters to be mixed and matched and attached to a given listener. There are three different types of network filters:

正如在 listener 部分所讨论的，网络级（L3/L4）过滤器构成了Envoy连接处理的核心。过滤器API允许混合和匹配不同的过滤器集合，并附加到给定的监听器。有三种不同类型的网络过滤器：

> - **Read**: Read filters are invoked when Envoy receives data from a downstream connection.
> - **Write**: Write filters are invoked when Envoy is about to send data to a downstream connection.
> - **Read/Write**: Read/Write filters are invoked both when Envoy receives data from a downstream connection and when it is about to send data to a downstream connection.

- **读**：当Envoy收到来自下游连接的数据时，会调用读过滤器。

- **写**：写过滤器在Envoy收到下游连接的数据时被调用。当Envoy要向下游连接发送数据时，会调用写过滤器。

- **读/写**：读/写过滤器在Envoy从下游连接接收数据和即将向下游连接发送数据时都会被调用。

> The API for network level filters is relatively simple since ultimately the filters operate on raw bytes and a small number of connection events (e.g., TLS handshake complete, connection disconnected locally or remotely, etc.). Filters in the chain can stop and subsequently continue iteration to further filters. This allows for more complex scenarios such as calling a [rate limiting service](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_features/global_rate_limiting#arch-overview-global-rate-limit), etc. Network level filters can also share state (static and dynamic) among themselves within the context of a single downstream connection. Refer to [data sharing between filters](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/advanced/data_sharing_between_filters#arch-overview-data-sharing-between-filters) for more details. Envoy already includes several network level filters that are documented in this architecture overview as well as the [configuration reference](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/network_filters#config-network-filters).

网络级过滤器的API相对简单，因为最终过滤器操作的是原始字节和少量的连接事件（例如，TLS握手完成，连接在本地或远程断开，等等）。链上的过滤器可以停止，随后继续迭代到更多的过滤器。这允许更复杂的场景，如调用速率限制服务等。网络级过滤器也可以在单个下游连接的范围内相互共享状态（静态和动态）。更多细节请参考过滤器之间的数据共享。Envoy已经包含了几个网络级过滤器，在这个架构概述以及配置参考中都有记录。
