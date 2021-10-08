---
title: "[译]Envoy中的Listener Filter Chain"
linkTitle: "[译]Listener Filter Chain"
weight: 424
date: 2021-10-08
description: >
  翻译Envoy中的Listener Filter Chain介绍内容
---

https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/network_filter_chain

> As discussed in the [listener](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/listeners/listeners#arch-overview-listeners) section, network level (L3/L4) filters form the core of Envoy connection handling.

正如在 Listener 部分所讨论的，网络级（L3/L4）过滤器构成了Envoy连接处理的核心。

> The network filters are chained in a ordered list known as [filter chain](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener_components.proto#envoy-v3-api-msg-config-listener-v3-filterchain). Each listener has multiple filter chains and an optional [default filter chain](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener.proto#envoy-v3-api-field-config-listener-v3-listener-default-filter-chain). associated with each filter chain. If the best match filter chain cannot be found, the default filter chain will be chosen to serve the request. If the default filter chain is not supplied, the connection will be closed.

网络过滤器在一个有序的列表中被链起来，称为过滤器链。每个监听器都有多个过滤器链和一个与每个过滤器链相关的可选默认过滤器链。如果找不到最佳匹配的过滤链，将选择默认的过滤链来处理请求。如果没有提供默认的过滤链，连接将被关闭。

## Filter chain only update



> [Filter chains](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener_components.proto#envoy-v3-api-msg-config-listener-v3-filterchain) can be updated independently. Upon listener config update, if the listener manager determines that the listener update is a filter chain only update, the listener update will be executed by adding, updating and removing filter chains. The connections owned by these destroying filter chains will be drained as described in listener drain.

过滤链可以独立更新。在 Listener 配置更新时，如果 listener 管理器确定 listener 更新只是更新过滤链，listener 更新将通过添加、更新和删除过滤链来执行。这些破坏的过滤链所拥有的连接将被逐出，如 listener drain 中所描述的那样。

> If the new [filter chain](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener_components.proto#envoy-v3-api-msg-config-listener-v3-filterchain) and the old [filter chain](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/listener/v3/listener_components.proto#envoy-v3-api-msg-config-listener-v3-filterchain) is protobuf message equivalent, the corresponding filter chain runtime info survives. The connections owned by the survived filter chains remain open.

如果新的过滤链和旧的过滤链是 protobuf 消息等价的，那么相应的过滤链运行时信息就会存活。存活的过滤链所拥有的连接保持打开。

> Not all the listener config updates can be executed by filter chain update. For example, if the listener metadata is updated within the new listener config, the new metadata must be picked up by the new filter chains. In this case, the entire listener is drained and updated.

并非所有的 listener 配置更新都可以通过过滤链更新来执行。例如，如果 listener 元数据在新的 listener 配置中被更新，新的元数据必须被新的过滤链所接收。在这种情况下，整个 listener 会被逐出并更新。

