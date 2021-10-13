---
title: "Envoy内建的Proxy Protocol Listener Filter"
linkTitle: "Proxy Protocol"
weight: 428
date: 2021-10-13
description: >
  Envoy内建的Proxy Protocol Listener Filter
---

这个Listener Filter过滤器增加了对 [HAProxy Proxy Protocol](https://www.haproxy.org/download/1.9/doc/proxy-protocol.txt) 的支持。

在这种模式下，下游连接被假定来自一个代理，它将原始坐标（IP，端口）放入一个连接字符串。然后Envoy提取这些数据并将其作为远程地址。

在 Proxy Protocol v2 中，存在着扩展（TLV）标签的概念，这些标签是可选的。如果TLV的类型被添加到过滤器的配置中，那么TLV将作为动态元数据，以用户指定的 key 发送出去。

这个实现同时支持版本1和版本2，它在每个连接的基础上自动确定这两个版本的存在。注意：如果过滤器被启用，代理协议必须存在于连接上（版本1或版本2），标准不允许解析以确定它是否存在。

如果有协议错误或不支持的地址族（如AF_UNIX），连接将被关闭并抛出一个错误。

这个过滤器应该被配置为 `envoy.filters.listener.proxy_protocol`。

Proxy Protocol 过滤器的配置范例是:

```yaml
listener_filters:
  - name: "envoy.filters.listener.proxy_protocol"
```

## 参考文档

- https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/proxy_protocol
- https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/listener/proxy_protocol/v3/proxy_protocol.proto#extension-envoy-filters-listener-proxy-protocol

