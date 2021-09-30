---
title: "xDS中的通用消息定义"
linkTitle: "通用消息"
weight: 1402
date: 2021-09-30
description: >
  xDS通用消息定义
---



## 通用消息定义

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/discovery/v3/discovery.proto

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



DiscoveryRequest 用同样的API为给定 envoy 节点请求一组相同类型的版本化资源。

| Field            | Type                | Description                                                  |
| :--------------- | :------------------ | :----------------------------------------------------------- |
| `version_info`   | `string`            | 请求消息中提供的 version_info 使用最近成功处理的响应接收的version_info，或者如果是第一个请求则设置为空。预期在收到响应之后不会发送新请求，直到客户端实例准备好对新配置进行ACK/NACK。 通过分别返回应用的新API配置版本或先前的API配置版本来进行ACK/NACK。 每个type_url（见下文）都有一个与之关联的独立版本。 |
| `node`           | `core.Node`         | 发起请求的节点                                               |
| resource_names   | string[]            | 要订阅的资源列表，例如集群名称列表或路由配置名称列表。 如果为空，则返回API的所有资源。 LDS/CDS期望空resource_names，因为这是Envoy实例的全局发现。 然后，LDS和CDS响应将意味着需要通过EDS / RDS获取许多资源，这些资源将在resource_names中明确枚举。 |
| `type_url`       | `string`            | 正在请求的资源的类型, e.g. “type.googleapis.com/envoy.api.v2.ClusterLoadAssignment”。对于单独的xDS API（如CDS，LDS等）发出的请求中是隐含的，但是对于ADS需要明确设定。 |
| `response_nonce` | `string`            | 对应于DiscoveryResponse的nonce，进行ACK / NACK。 请参阅上面有关version_info和DiscoveryResponse nonce注释的讨论。 如果没有可用的nonce，这可能是空的，例如，在启动时。 |
| `error_detail`   | `google.rpc.Status` | 当前一个DiscoveryResponse无法更新配置时，将填充此选项。 error_details中的message字段提供与失败相关的客户端内部异常。 它仅供手动调试时使用，不保证在客户端版本中提供的字符串是稳定的。 |

### DiscoveryResponse[ ](http://localhost:1313/docs/introduction/other/discovery-message.html#discoveryresponse)



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

