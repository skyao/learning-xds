---
title: "Envoy内建的HTTP Inspector"
linkTitle: "Original Destination"
weight: 426
date: 2021-10-13
description: >
  Envoy内建的HTTP Inspector
---

原始目的地(Original destination)监听器过滤器读取 `SO_ORIGINAL_DST` 套接字选项，当连接被 iptables REDIRECT target 重定向，或被iptables TPROXY target 结合设置监听器的透明选项重定向。

这个过滤器应该被配置为 `envoy.filters.listener.original_dst`。

HTTP Inspector 过滤器的配置范例是:

```yaml
listener_filters:
  - name: "envoy.filters.listener.original_dst"
```

## 参考文档

- https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listener_filters/original_dst_filter
- https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/listener/original_dst/v3/original_dst.proto#extension-envoy-filters-listener-original-dst

