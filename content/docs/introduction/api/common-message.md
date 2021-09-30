---
title: "xDS中的通用消息定义"
linkTitle: "通用消息定义"
weight: 1402
date: 2021-09-30
description: >
  xDS通用消息定义
---



## 通用消息定义

https://github.com/envoyproxy/envoy/blob/45ec050f91407147ed53a999434b09ef77590177/api/envoy/service/discovery/v3/discovery.proto

### DiscoveryRequest消息

```protobuf
message DiscoveryRequest {
  option (udpa.annotations.versioning).previous_message_type = "envoy.api.v2.DiscoveryRequest";

  string version_info = 1;

  config.core.v3.Node node = 2;

  repeated string resource_names = 3;

  string type_url = 4;

  string response_nonce = 5;

  google.rpc.Status error_detail = 6;
}
```

### DiscoveryResponse消息

```protobuf
message DiscoveryResponse {
  option (udpa.annotations.versioning).previous_message_type = "envoy.api.v2.DiscoveryResponse";

  string version_info = 1;

  repeated google.protobuf.Any resources = 2;

  bool canary = 3;

  string type_url = 4;

  string nonce = 5;

  config.core.v3.ControlPlane control_plane = 6;
}
```

### DeltaDiscoveryRequest消息

```protobuf
message DeltaDiscoveryRequest {
  option (udpa.annotations.versioning).previous_message_type = "envoy.api.v2.DeltaDiscoveryRequest";

  config.core.v3.Node node = 1;

  string type_url = 2;

  repeated string resource_names_subscribe = 3;

  repeated string resource_names_unsubscribe = 4;

  map<string, string> initial_resource_versions = 5;

  string response_nonce = 6;

  google.rpc.Status error_detail = 7;
}
```

### DeltaDiscoveryResponse消息

```protobuf
message DeltaDiscoveryResponse {
  option (udpa.annotations.versioning).previous_message_type =
      "envoy.api.v2.DeltaDiscoveryResponse";

  string system_version_info = 1;

  repeated Resource resources = 2;

  string type_url = 4;

  repeated string removed_resources = 6;

  string nonce = 5;

  config.core.v3.ControlPlane control_plane = 7;
}
```

