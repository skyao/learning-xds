---
title: "Envoy内建的TLS Inspector Listener Filter"
linkTitle: "TLS Inspector"
weight: 429
date: 2021-10-13
description: >
  Envoy内建的TLS Inspector Listener Filter
---

TLS检查器监听器过滤器允许检测传输似乎是TLS还是明文(plaintext)，如果是TLS，它检测来自客户端的服务器名称指示([Server Name Indication](https://en.wikipedia.org/wiki/Server_Name_Indication))和/或应用层协议协商([Application-Layer Protocol Negotiation](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation))。这可以用来通过 FilterChainMatch 的 `server_names` 和/或`application_protocols` 来选择FilterChain。

这个过滤器应该被配置为 `envoy.filters.listener.tls_inspector`。

TLS Inspector 过滤器的配置范例是:

```yaml
listener_filters:
- name: "envoy.filters.listener.tls_inspector"
```

或者通过指定 typed_config 的 type_url:

```yaml
listener_filters:
- name: "tls_inspector"
  typed_config:
    "@type": type.googleapis.com/envoy.extensions.filters.listener.tls_inspector.v3.TlsInspector
```

## SNI相关配置

https://www.envoyproxy.io/docs/envoy/latest/faq/configuration/sni#faq-how-to-setup-sni

### 如何为Listener配置SNI？

```yaml
listener_filters:
- name: "envoy.filters.listener.tls_inspector"
```

> 备注：原文大段配置，但和 tls_inspector 相关的应该就是这些

### 如何为Cluster配置SNI？

对于集群，可以在 UpstreamTlsContext 中设置一个固定的SNI。要从下游的HTTP头（如`host`或`:authority`）导出SNI，打开 auto_sni 以覆盖UpstreamTlsContext 中的固定SNI。也可以使用可选的 `override_auto_sni_header` 字段提供 `host` 或 `:authority` 以外的自定义头信息。如果上游将在SAN中呈现带有主机名的证书，也要打开 auto_san_validation。它仍然需要 UpstreamTlsContext 中的验证上下文中的信任CA作为信任锚。

## 背景知识

### SNI

https://en.wikipedia.org/wiki/Server_Name_Indication

服务器名称指示（Server Name Indication / SNI）是对传输层安全（TLS）计算机网络协议的扩展，通过SNI，客户端在握手过程开始时表明它试图连接到哪个主机名。这允许服务器在同一IP地址和TCP端口号上呈现多个可能的证书之一，因此允许多个安全（HTTPS）网站（或任何其他通过TLS的服务）由同一IP地址提供，而不要求所有这些网站使用同一证书。它在概念上等同于HTTP/1.1基于名字的虚拟主机，但用于HTTPS。这也允许代理在TLS/SSL握手期间将客户端流量转发到正确的服务器。所需的主机名在原始SNI扩展中没有加密，所以窃听者可以看到正在请求哪个网站。

### ALPN

应用层协议协商（Application-Layer Protocol Negotiation / ALPN）是一个传输层安全（Transport Layer Security / TLS）的扩展，它允许应用层以避免额外往返的方式协商应该在安全连接上执行的协议，而这是独立于应用层协议的。它被用来建立HTTP/2连接而不需要额外的往返（客户端和服务器可以通过以前分配给HTTPS的HTTP/1.1端口进行通信，并升级到使用HTTP/2或继续使用HTTP/1.1而不关闭初始连接）。

## 参考文档

- https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/tls_inspector
- https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/listener/tls_inspector/v3/tls_inspector.proto#extension-envoy-filters-listener-tls-inspector

