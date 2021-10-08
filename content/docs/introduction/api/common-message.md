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
| `resource_names` | `string[]`          | 要订阅的资源列表，例如集群名称列表或路由配置名称列表。 如果为空，则返回API的所有资源。 LDS/CDS可以设置空resource_names，这将导致返回Envoy实例的所有资源。 然后，LDS和CDS响应将暗示需要通过EDS/RDS获取许多资源，这些资源将被明确列举在resource_names中。 |
| `type_url`       | `string`            | 正在请求的资源的类型, e.g. “type.googleapis.com/envoy.api.v2.ClusterLoadAssignment”。对于单独的xDS API（如CDS，LDS等）发出的请求中是隐含的，但是对于ADS需要明确设定。 |
| `response_nonce` | `string`            | 对应于DiscoveryResponse的nonce，进行 ACK/NACK。 请参阅上面有关version_info和DiscoveryResponse nonce注释的讨论。 只有当1）这是一个非持久流的xDS，如HTTP，或2）客户端尚未接受这个xDS流中的更新时，它才可能是空的（与delta不同，它只对新的明确ACKs进行填充）。 |
| `error_detail`   | `google.rpc.Status` | 当前一个DiscoveryResponse无法更新配置时，将填充此选项。 error_details中的message字段提供与失败相关的客户端内部异常。 它仅供手动调试时使用，不保证在客户端版本中提供的字符串是稳定的。 |

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

DiscoveryResponse 提供一组相同类型的版本化资源以响应 DiscoveryRequest。

| Field           | Type                          | Description                                                  |
| :-------------- | :---------------------------- | :----------------------------------------------------------- |
| `version_info`  | `string`                      | 响应数据的版本。                                             |
| `resources`     | `google.protobuf.Any[]`       | 响应资源。这些资源是有类型的，而且取决于被调用的API。        |
| `canary`        | `bool`                        | Canary用于支持两个Envoy命令行标志。<br/><br/>* `--terminat-on-canary-transition-failure`。设置后，Envoy能够在检测到配置卡在canary时终止。考虑下面这个例子的更新顺序：<br/>     - 管理服务器成功应用了一个canary配置。<br/>     - 管理服务器回滚到一个生产配置。<br/>     - Envoy拒绝新的生产配置。 由于没有合理的方法来继续接收配置更新，Envoy将终止并从一个干净的地方应用生产配置。<br/>  * `--dry-run-canary`. 如果设置了这个选项，就不会应用金丝雀响应，只通过干运行来验证。 |
| `type_url`      | `string`                      | 资源类型URL。 在通过ADS复用时识别xDS API。<br /><br />必须与`resources`中的type_url一致（如果非空）。 |
| `nonce`         | `string`                      | 对于基于gRPC的订阅，nonce提供了一种在后续DiscoveryRequest中明确ACK特定DiscoveryResponse的方法。 客户端可能已经在此DiscoveryResponse之前在流上向管理服务器发送了其他消息，这些消息在响应发送时未被处理。 nonce允许管理服务器忽略先前版本的任何后续DiscoveryRequest，直到带有nonce的DiscoveryRequest。nonce是可选的，对于不是基于stream的xDS实现来说不是必须的。 |
| `control_plane` | `config.core.v3.ControlPlane` | 发送应答的控制平面实例。                                     |



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

DeltaDiscoveryRequest 和 DeltaDiscoveryResponse 用于 Delta xDS 的新gRPC端点。

使用Delta xDS，DeltaDiscoveryResponses不需要包含跟踪资源的完整快照。 相反，DeltaDiscoveryResponses是xDS客户端状态的差异。

在Delta XDS中，每个资源都有版本，允许以资源粒度跟踪状态。

xDS Delta 会话始终位于 gRPC 双向流的上下文中。 允许 xDS 服务器跟踪连接到它的 xDS 客户端的状态。

在 Delta xDS 中，nonce字段是必需的，用于将 DeltaDiscoveryResponse 与 DeltaDiscoveryRequest 配对进行 ACK或NACK。 可选地，响应消息级别 system_version_info 仅用于调试目的。

DeltaDiscoveryRequest扮演两个独立的角色。任何DeltaDiscoveryRequest都可以是以下两者中的一个或两个：[1] 通知服务器客户端对哪些资源感兴趣（使用 resource_names_subscribe 和 resource_names_unsubscribe），或者 [2] (N) ACK先前来自服务器的资源更新（使用 response_nonce，出现 error_detail 则变成 NACK）。此外，重新连接的gRPC流的第一个消息（对于给定的type_url）有第三个作用：使用 `initial_resource_versions` 字段，告知服务器客户端已经拥有的资源（及其版本）。

与世界现状一样，当多种资源类型被复用时（ADS），所有的请求/确认/更新在逻辑上被type_url隔离：集群ACK存在于与之前的路由NACK完全不同的世界。 特别是，在每个gRPC流的 "开始" 时发送的initial_resource_versions实际上包含了每个type_url的消息，每个都有自己的initial_resource_versions。

| Field                        | Type                  | Description                                                  |
| :--------------------------- | :-------------------- | :----------------------------------------------------------- |
| `node`                       | `core.Node`           | 响应数据的版本。                                             |
| `type_url`                   | `string`              | 正在请求的资源的类型, e.g. “type.googleapis.com/envoy.api.v2.ClusterLoadAssignment”。如果资源只通过 *xds_resource_subscribe* 和 *xds_resources_unsubscribe* 来引用，则不需要设置。 |
| `resource_names_subscribe`   | `string[]`            | 要添加到跟踪资源列表的资源名称列表。                         |
| `resource_names_unsubscribe` | `string[]`            | 要从跟踪资源列表中删除的资源名称列表。                       |
| `initial_resource_versions`  | `map<string, string>` | 通知服务器xDS客户端所知道的资源版本，以使客户端即使在gRPC流重新连接的情况下也能继续同一个逻辑xDS会话。以下情况可以不用设置：[1]在会话的第一个流中，因为客户端还没有任何资源 [2]在流的第一个消息之后的任何消息中（对于一个给定的type_url），因为服务器已经正确地跟踪了客户端的状态。 (在ADS中，重新连接的流中每个 type_url 的第一条消息会填充这个地图）。<br /><br />该map的键是xDS客户端已知的xDS资源的名称。map的值是不透明的资源版本。 |
| `response_nonce`             | `string`              | 当 DeltaDiscoveryRequest 是响应前一个 DeltaDiscoveryResponse 的 ACK 或NACK 消息时，response_nonce 必须是 DeltaDiscoveryResponse 中的nonce。 <br /><br />否则必须省略 response_nonce。 |
| `error_detail`               | `google.rpc.Status`   | 当前一个DiscoveryResponse无法更新配置时，将填充此选项。 error_details中的message字段提供与失败相关的客户端内部异常。 |

resource_names_subscribe 和 resource_names_unsubscribe 字段的额外说明：

> DeltaDiscoveryRequests 允许客户端在流的上下文中向被跟踪的资源集合添加或删除单个资源。
>
> resource_names_subscribe 列表中的所有资源名称都将添加到跟踪资源集合中，而 resource_names_unsubscribe 列表中的所有资源名称将从该组跟踪资源中删除。
>
> 与xDS不同，空的 resource_names_subscribe 或 resource_names_unsubscribe 列表仅表示不会向资源列表添加或删除任何资源。
>
> xDS 服务器必须为所有跟踪的资源发送更新，但也可以发送客户端尚未订阅的资源的更新。 此行为类似 xDS。
>
> 注意：服务器必须响应所有在 resource_names_subscribe 中列出的资源，即使它认为客户端拥有它们的最新版本。原因是：客户可能已经放弃了它们，但在它有机会发送 unsubscribe 消息之前又恢复了兴趣。参见DeltaSubscriptionStateTest.RemoveThenAdd。
>
> 这两个字段可以在任何 DeltaDiscoveryRequest 中设置，包括 ACKs 和 initial_resource_versions。

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



| Field                 | Type                          | Description                                                  |
| :-------------------- | :---------------------------- | :----------------------------------------------------------- |
| `system_version_info` | `string`                      | 响应数据的版本（用于debug）。                                |
| `resources`           | `Resource[]`                  | 响应资源。这些资源是有类型的，他们的类型必须和 DeltaDiscoveryRequest 中的 type url 匹配。 |
| `type_url`            | `string`                      | 资源类型URL。 在通过ADS复用时识别xDS API。<br /><br />必须与`resources`中的type_url一致（如果非空）。 |
| `removed_resources`   | `string[]`                    | 被删除的资源的资源名称，这些资源在xDS客户端也应该被删除。<br /><br />如果要删除的资源不存在则可以忽略。 |
| `nonce`               | `string`                      | nonce为 DeltaDiscoveryRequests 提供唯一一种在ACN或者NACK时引用到 DeltaDiscoveryResponse 的方法。 nonce是必需的。 |
| `control_plane`       | `config.core.v3.ControlPlane` | 发送应答的控制平面实例。                                     |



## 引用到的消息定义

### Resource

Delta 中用到的 Resource 消息：

```protobuf
message Resource {
  option (udpa.annotations.versioning).previous_message_type = "envoy.api.v2.Resource";

  message CacheControl {
    bool do_not_cache = 1;
  }

  string name = 3;

  repeated string aliases = 4;

  string version = 1;

  google.protobuf.Any resource = 2;

  google.protobuf.Duration ttl = 6;

  CacheControl cache_control = 7;
}
```



| Field           | Type                       | Description                                                  |
| :-------------- | :------------------------- | :----------------------------------------------------------- |
| `name`          | `string`                   | 资源的名称，以区别于其他同类型的资源。                       |
| `aliases`       | `string[]`                 | 别名是该资源可以使用的其他名称的列表。                       |
| `version`       | `string[]`                 | 资源层面的版本。它允许xDS跟踪单个资源的状态。                |
| `resource`      | `google.protobuf.Any`      | 被跟踪的资源。                                               |
| `ttl`           | `google.protobuf.Duration` | 资源的生存时间值。对于每个资源，都会启动一个定时器。每次收到带有新的TTL的资源时，该计时器就会被重置。如果收到的资源没有设置TTL，则该资源的定时器被删除。计时器过期后，该资源的配置将被删除。<br /><br />可以通过发送一个不改变资源版本的响应来刷新或改变TTL。在这种情况下，资源字段不需要被填充，这允许轻量级的 "心跳" 更新，以保持一个有TTL的资源的活力。<br /><br />TTL功能是为了支持那些在管理服务器故障时应该被删除的配置。例如，该功能可用于故障注入测试，在Envoy与管理服务器失去联系的情况下，故障注入应该被终止。 |
| `cache_control` | `CacheControl`             | 该资源的缓存控制属性。                                       |



### config.core.v3.Node

https://github.com/envoyproxy/envoy/blob/main/api/envoy/config/core/v3/base.proto

Node 用于识别特定的Envoy实例。节点标识符被提交给管理服务器，管理服务器可以使用这个标识符来区分它服务的每个Envoy的配置。

```protobuf
message Node {
  option (udpa.annotations.versioning).previous_message_type = "envoy.api.v2.core.Node";

  reserved 5;

  reserved "build_version";

  string id = 1;

  string cluster = 2;

  google.protobuf.Struct metadata = 3;

  map<string, xds.core.v3.ContextParams> dynamic_parameters = 12;

  Locality locality = 4;

  string user_agent_name = 6;

  oneof user_agent_version_type {
    string user_agent_version = 7;

    BuildVersion user_agent_build_version = 8;
  }

  repeated Extension extensions = 9;

  repeated string client_features = 10;

  repeated Address listening_addresses = 11
      [deprecated = true, (envoy.annotations.deprecated_at_minor_version) = "3.0"];
}
```



| Field                 | Type                                     | Description                                                  |
| :-------------------- | :--------------------------------------- | :----------------------------------------------------------- |
| `id`                  | `string`                                 | Envoy节点的不透明的节点标识符。这也提供了本地服务节点的名称。 |
| `cluster`             | `string`                                 | 定义了运行Envoy的本地服务集群名称。                          |
| `metadata`            | `google.protobuf.Struct`                 | 扩展节点标识符的不透明元数据。Envoy将把这个直接传递给管理服务器。 |
| `dynamic_parameters`  | `map<string, xds.core.v3.ContextParams>` | 从xDS资源 type URL 到动态上下文参数的映射。这些在运行时可能会有变化（与本消息中的其他字段不同）。例如，xDS客户端可能有一个分片标识符，在xDS客户端的生命周期内会发生变化。在Envoy中，这将通过更新 Server::Instance 的 LocalInfo 上下文提供者上的动态上下文来实现。在未来的发现请求中，分片ID动态参数会出现在这个字段中。 |
| `locality`            | `Locality`                               | 指定Envoy实例的运行区域。                                    |
| `user_agent_name`     | `string`                                 | 自由格式的字符串，用于识别请求配置的实体。<br/><br />例如，"envoy "或 "grpc"。 |
| `extensions`          | `extensions[]`                           | 节点支持的扩展及其版本的列表。                               |
| `client_features`     | `string`                                 | 客户端功能支持列表。这些是在Envoy API库中描述的众所周知的功能，适用于某一API的主要版本。客户端功能使用反向DNS命名方案，例如`com.acme.feature`。<br /><br />参见 xDS 客户端可能支持的功能列表。 |
| `listening_addresses` | `string`                                 | 节点上已知的监听端口，作为管理服务器的通用提示，用于过滤 :ref:`listeners <config_listeners>` 返回。例如，如果有一个监听器绑定到80端口，列表中可以选择包含SocketAddress `(0.0.0.0,80)`。这个字段是可选的，只是一个提示。 |

### config.core.v3.ControlPlane

识别Envoy所连接的特定控制平面实例。

```protobuf
message ControlPlane {
  option (udpa.annotations.versioning).previous_message_type = "envoy.api.v2.core.ControlPlane";

  string identifier = 1;
}
```

|              | Type     | Description                                                  |
| :----------- | :------- | :----------------------------------------------------------- |
| `identifier` | `string` | 一个不透明的控制平面标识符，唯一标识控制平面的一个实例。这可以用来识别Envoy连接到哪个控制平面实例。 |

## type_url取值

以下是几个常见的 type_url: 

| 资源名                   | type_url的值                                                 |
| ------------------------ | ------------------------------------------------------------ |
| Listener                 | "type.googleapis.com/envoy.config.listener.v3.Listener"      |
| Cluster                  | "type.googleapis.com/envoy.config.cluster.v3.Cluster"        |
| ClusterLoadAssignment    | "type.googleapis.com/envoy.config.endpoint.v3.ClusterLoadAssignment" |
| Secret                   | "type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.Secret" |
| RouteConfiguration       | "type.googleapis.com/envoy.config.route.v3.RouteConfiguration" |
| VirtualHost              | "type.googleapis.com/envoy.config.route.v3.VirtualHost"      |
| ScopedRouteConfiguration | "type.googleapis.com/envoy.config.route.v3.ScopedRouteConfiguration" |
| Runtime                  | "type.googleapis.com/envoy.service.runtime.v3.Runtime"       |

type_url 取值的规律是 `"type.googleapis.com" 前缀 + 资源名称`，其在envoy中的实现代码在头文件 [source/common/config/resource_name.h](https://github.com/envoyproxy/envoy/blob/main/source/common/config/resource_name.h) 中:

```c++
/**
 * Get type url from api type.
 */
template <typename Current> std::string getTypeUrl() {
  return "type.googleapis.com/" + getResourceName<Current>(); 
}
```

如 Listener 的定义在 proto 文件 `https://github.com/envoyproxy/envoy/blob/main/api/envoy/config/listener/v3/listener.proto` 中,

```protobuf
syntax = "proto3";

package envoy.config.listener.v3;

message Listener {
......
}
```

因此 Listener/LDS 的 type_url 就是 `"type.googleapis.com/envoy.config.listener.v3.Listener"` 。
