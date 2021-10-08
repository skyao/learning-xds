---
title: "[译]Envoy中的Listener Filter"
linkTitle: "[译]Listener Filter"
weight: 423
date: 2021-10-08
description: >
  翻译Envoy中的Listener Filter介绍内容
---

https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listener_filters

> As discussed in the [listener](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listeners#arch-overview-listeners) section, listener filters may be used to manipulate connection metadata. The main purpose of listener filters is to make adding further system integration functions easier by not requiring changes to Envoy core functionality, and also make interaction between multiple such features more explicit.

如 监听器 一节所述, 监听器过滤器可以用于操纵连接元数据。 监听器过滤器的主要目的是更方便地添加系统集成功能，而无需更改 Envoy 核心功能，并使多个此类功能之间的交互更加明确。

> The API for listener filters is relatively simple since ultimately these filters operate on newly accepted sockets. Filters in the chain can stop and subsequently continue iteration to further filters. This allows for more complex scenarios such as calling a [rate limiting service](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/other_features/global_rate_limiting#arch-overview-global-rate-limit), etc. Envoy already includes several listener filters that are documented in this architecture overview as well as the [configuration reference](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/listener_filters#config-listener-filters).

监听器过滤器的 API 相对简单，因为最终这些过滤器是在新接收的套接字上操作的。可停止链中的过滤器并继续执行后续的过滤器。这允许去运作更复杂的业务场景，例如调用 限速服务 等。 Envoy 包含多个监听器过滤器，这些过滤器在架构概述以及 配置参考 中都有记录。

## 参考文档

- https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listener_filters envoy官方文档中对Listener的说明
- https://cloudnative.to/envoy/intro/arch_overview/listeners/listener_filters.html 上文的中文翻译

