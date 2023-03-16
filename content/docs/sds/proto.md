---
title: "SDS的proto定义"
linkTitle: "proto"
weight: 20
date: 2021-09-28
description: >
  SDS的proto定义
---



## proto 定义

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/secret/v3/sds.proto

```protobuf
syntax = "proto3";

package envoy.service.secret.v3;
......

service SecretDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.extensions.transport_sockets.tls.v3.Secret";

  rpc DeltaSecrets(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc StreamSecrets(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc FetchSecrets(discovery.v3.DiscoveryRequest) returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:secrets";
    option (google.api.http).body = "*";
  }
}
```

定义了三个方法：

1. FetchSecrets(): 获取 secret，单次调用单次返回的版本 
2. StreamSecrets()： 获取 secret，单次调用stream多次返回的版本
3. DeltaSecrets()：获取 secret，单次调用stream多次返回增量数据的版本

## FetchSecrets()

