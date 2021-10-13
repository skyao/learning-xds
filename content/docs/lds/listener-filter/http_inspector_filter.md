---
title: "Envoy内建的HTTP Inspector"
linkTitle: "HTTP Inspector"
weight: 425
date: 2021-10-13
description: >
  Envoy内建的HTTP Inspector
---

HTTP Inspector监听器过滤器允许检测应用协议是否为HTTP，如果是HTTP，则进一步检测HTTP协议（HTTP/1.x或HTTP/2）。这可以用来通过FilterChainMatch 的 application_protocols 来选择 FilterChain。

这个过滤器应该被配置为 `envoy.filters.listener.http_inspector`。

HTTP Inspector 过滤器的配置范例是:

```yaml
listener_filters:
  - name: "envoy.filters.listener.http_inspector"
```

## 参考文档

- https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/http_inspector
- https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/listener/http_inspector/v3/http_inspector.proto#envoy-v3-api-msg-extensions-filters-listener-http-inspector-v3-httpinspector

