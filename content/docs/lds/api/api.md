---
title: "LDS API的概述"
linkTitle: "LDS API概述"
weight: 421
date: 2021-09-28
description: >
  介绍LDS API的服务定义和参数说明
---

### LDS服务定义

LDS的服务定义如下：

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/listener/v3/lds.proto

```protobuf
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

### 请求参数

对于 DeltaDiscoveryRequest 和 DiscoveryRequest，请求参数中的 type_url 字段的取值是 "type.googleapis.com/envoy.config.listener.v3.Listener" (或者隐含)。

### 应答参数

对于 DiscoveryResponse 和 DeltaDiscoveryResponse，返回的资源类型被抽象为 `google.protobuf.Any` ，对于LDS返回的实际是 Listener 这个消息体，定义在 `api/envoy/config/listener/v3/listener.proto` 文件中: 

```protobuf
package envoy.config.listener.v3;

message Listener {

  string name = 1;

  core.v3.Address address = 2 [(validate.rules).message = {required: true}];


  repeated FilterChain filter_chains = 3;

  repeated ListenerFilter listener_filters = 9;

  ......
}
```

详细的字段定义请见后续的详细解说。
