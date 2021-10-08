---
title: "[译]Envoy中的Listener"
linkTitle: "[译]Listener"
weight: 402
date: 2021-09-28
description: >
  翻译Envoy中的Listener介绍内容
---

https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listeners

**Listeners**

> The Envoy configuration supports any number of listeners within a single process. Generally we recommend running a single Envoy per machine regardless of the number of configured listeners. This allows for easier operation and a single source of statistics. Envoy supports both TCP and UDP listeners.

Envoy 支持单个进程中配置任意数量的监听器。通常情况下，部署 Envoy 的数量与监听器数量无关，建议每台机器部署一个 Envoy 即可。这使得操作会更加简便，而且每台机器只有一个监控数据来源。Envoy 同时支持 TCP 和 UDP 监听器。

**TCP**

> Each listener is independently configured with some number filter chains, where an individual chain is selected based on its match criteria. An individual filter chain is composed of one or more network level (L3/L4) filters. When a new connection is received on a listener, the appropriate filter chain is selected, and the configured connection local filter stack is instantiated and begins processing subsequent events. The generic listener architecture is used to perform the vast majority of different proxy tasks that Envoy is used for (e.g., rate limiting, TLS client authentication, HTTP connection management, MongoDB sniffing, raw TCP proxy, etc.).

每个监听器都独立配置了多个 过滤器链，其中根据其 匹配条件 选择某个过滤器链。 一个独立的过滤器链由一个或多个网络层(L3/L4) 过滤器 组成。 当监听器上接收到新连接时，会选择适当的过滤器链，接着实例化配置的本地筛选器堆栈和处理后续事件。 通用监听器系统架构通常用于处理各不相同的代理任务，例如 Envoy 用于（ 限速、TLS 客户端身份验证、HTTP 连接管理、MongoDB 嗅探、原始 TCP 代理 等）。

> Listeners are optionally also configured with some number of listener filters. These filters are processed before the network level filters, and have the opportunity to manipulate the connection metadata, usually to influence how the connection is processed by later filters or clusters.

监听器还可以选择配置一些 监听过滤器。 这些过滤器在网络层过滤器之前处理，并且有机会去操作连接元数据，这样通常是为了影响后续过滤器或集群如何处理连接。

> Listeners can also be fetched dynamically via the listener discovery service (LDS).

还可以通过 监听器发现服务 (LDS) 动态获取监听器。

**UDP**

> Envoy also supports UDP listeners and specifically UDP listener filters. UDP listener filters are instantiated once per worker and are global to that worker. Each listener filter processes each UDP datagram that is received by the worker listening on the port. In practice, UDP listeners are configured with the SO_REUSEPORT kernel option which will cause the kernel to consistently hash each UDP 4-tuple to the same worker. This allows a UDP listener filter to be “session” oriented if it so desires. A built-in example of this functionality is the UDP proxy listener filter.

Envoy 还支持 UDP 监听器和 UDP 监听过滤器 。 每个工作线程都会实例化一次 UDP 监听过滤器，并且 UDP 监听过滤器对于工作线程来讲是全局可见的。 每个工作线程监听端口接收到的 UDP 数据报文由监听过滤器处理。 实际上，UDP 监听器配置有一个 `SO_REUSEPORT` 内核选项，这将导致内核会将属于同一个 UDP 四个元组的 UDP 报文分配给同一个工作线程。 这就允许 UDP 监听过滤器在需要时可以打开会话保持功能。 这个功能的一个内置示例就是 UDP 代理 监听过滤器。

## 参考文档

- https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listeners envoy官方文档中对Listener的说明
- https://cloudnative.to/envoy/intro/arch_overview/listeners/listeners.html 上文的中文翻译
