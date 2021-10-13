---
title: "LDS版本演进"
linkTitle: "LDS版本演进"
weight: 481
date: 2021-09-28
description: >
  介绍LDS在各个版本中的演进
---

## LDS定义

### LDS v1

TODO：LDS v1的定义在哪里？

### LDS v2

https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/lds.proto

```protobuf
// Envoy实例在启动时发起一个RPC，以发现监听器列表。
// 更新是通过LDS服务器的流式传输的，包括所有监听器的完整更新。
// 监听器不再存在时，现有的连接将被允许剔除。
service ListenerDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.api.v2.Listener";

  rpc DeltaListeners(stream DeltaDiscoveryRequest) returns (stream DeltaDiscoveryResponse) {
  }

  rpc StreamListeners(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc FetchListeners(DiscoveryRequest) returns (DiscoveryResponse) {
    option (google.api.http).post = "/v2/discovery:listeners";
    option (google.api.http).body = "*";
  }
}
```

### LDS v3

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/listener/v3/lds.proto

```protobuf
// Envoy实例在启动时发起一个RPC，以发现监听器列表。
// 更新是通过LDS服务器的流式传输的，包括所有监听器的完整更新。
// 监听器不再存在时，现有的连接将被允许剔除。
service ListenerDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.config.listener.v3.Listener";

  rpc DeltaListeners(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc StreamListeners(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc FetchListeners(discovery.v3.DiscoveryRequest) returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:listeners";
    option (google.api.http).body = "*";
  }
}
```

和 LDS v2 API相比：

1. option type 从 `envoy.api.v2.Listener` 修改为 `envoy.config.listener.v3.Listener`
2. 各种message定义都增加了 `discovery.v3` 的前缀

## 相关消息体的定义

DeltaDiscoveryRequest

