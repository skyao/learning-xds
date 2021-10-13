---
title: "Envoy内建的Original Source Listener Filter"
linkTitle: "Original Source"
weight: 427
date: 2021-10-13
description: >
  Envoy内建的Original Source Listener Filter
---

原始来源(original source)监听器过滤器在Envoy的上游复制连接的下游远程地址(remote address)。例如，如果一个下游连接以IP地址 `10.1.2.3` 连接到Envoy，那么Envoy将以源IP `10.1.2.3` 连接到上游。

这个过滤器应该被配置为 `envoy.filters.listener.original_src`。

Original source 过滤器的配置范例是:

```yaml
listener_filters:
  - name: "envoy.filters.listener.original_src"
```

### 与代理协议的交互

如果该连接的源地址没有被转换或代理，那么Envoy可以简单地使用现有的连接信息来构建正确的下游远程地址(remote address)。然而，如果不是这样，可以使用代理协议过滤器(Proxy Protocol filter)来提取下游的远程地址。

### IP版本支持

该过滤器同时支持IPv4和IPv6作为地址。注意，上游连接必须支持所使用的版本。

### 额外设置

使用的下游远程地址可能是全球可路由的。默认情况下，从上游主机返回该地址的数据包不会通过Envoy路由。必须对网络进行配置，以强制将任何由Envoy复制的IP流量通过Envoy主机进行路由。

如果Envoy和上游主机在同一台主机上--例如在 sidecar 部署中--那么可以使用 iptables 和路由规则来确保行为正确。该过滤器有一个无符号的整数配置，mark。将其设置为X会使Envoy对来自该监听器的所有上游数据包进行X值标记。注意，如果mark设置为0，Envoy不会对上游数据包进行标记。

我们可以使用下面这组命令来确保所有标有X（在本例中假定为123）的ipv4和ipv6流量正确路由。注意，这个例子假设eth0是默认的出站接口。

下面的例子将Envoy配置为对8888端口的所有连接使用原始来源。它使用代理协议来确定下游的远程地址。所有上游数据包都被标记为123。

```yaml
listeners:
- address:
    socket_address:
      address: 0.0.0.0
      port_value: 8888
  listener_filters:
    - name: envoy.filters.listener.proxy_protocol
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.filters.listener.proxy_protocol.v3.ProxyProtocol
    - name: envoy.filters.listener.original_src
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.filters.listener.original_src.v3.OriginalSrc
        mark: 123
```

## 参考文档

- https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/original_src_filter
- https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/listener/original_src/v3/original_src.proto#envoy-v3-api-msg-extensions-filters-listener-original-src-v3-originalsrc

