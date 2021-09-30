---
title: "xDS概述"
linkTitle: "xDS概述"
weight: 1101
date: 2021-09-28
description: >
  介绍Envoy的XDS API
---

在 Envoy 中，xDS 被称为数据平面API，是控制平面（如Istio）和数据平面之间的通讯协议。

### xDS的含义

xDS 是指 "X Discovery Service"，这里的 "X" 代指多种服务发现协议，包括：

| 简写 |                全称                |        描述        |
| :--: | :--------------------------------: | :----------------: |
| LDS  |     Listener Discovery Service     |   监听器发现服务   |
| RDS  |      Route Discovery Service       |    路由发现服务    |
| CDS  |     Cluster Discovery Service      |    集群发现服务    |
| EDS  |     Endpoint Discovery Service     |  集群成员发现服务  |
| ADS  |    Aggregated Discovery Service    |    聚合发现服务    |
| HDS  |      Health Discovery Service      |   健康度发现服务   |
| SDS  |      Secret Discovery Service      |    密钥发现服务    |
|  MS  |           Metric Service           |      指标服务      |
| RLS  |         Rate Limit Service         |    限流发现服务    |
| LRS  |       Load Reporting service       |    负载报告服务    |
| RTDS |     Runtime Discovery Service      |   运行时发现服务   |
| CSDS |  Client Status Discovery Service   | 客户端状态发现服务 |
| ECDS | Extension Config Discovery Service |  扩展配置发现服务  |
| xDS  |        X Discovery Service         | 以上诸多API的统称  |

> 备注：
>
> 1. SDS 在 xDS v1版本中指的是 Service Discovery Service，后来 SDS 改名 Endpoint Discovery Service/EDS。再后来增加了 Security Discovery Service.
> 2. 后来新增了一些协议，名字不再是 Discovery Service，但也归入xDS的名下

### xDS版本

目前 xDS 主要有三个版本：

- v1: 
- v2: 将于 2020 年底停止使用
- v3: 目前正在支持的版本



## xds通用模式

LDS/EDS/CDS/EDS 这四个xDS API的定义非常类似，模式都是一样的。

LDS：

```protobuf
service ListenerDiscoveryService {
  rpc StreamListeners(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc FetchListeners(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```

RDS：

```protobuf
service RouteDiscoveryService {
  rpc StreamRoutes(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc DeltaRoutes(stream DeltaDiscoveryRequest) returns (stream DeltaDiscoveryResponse) {
  }

  rpc FetchRoutes(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```

CDS：

```protobuf
service ClusterDiscoveryService {
  rpc StreamClusters(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc DeltaClusters(stream DeltaDiscoveryRequest) returns (stream DeltaDiscoveryResponse) {
  }

  rpc FetchClusters(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```

EDS：

```protobuf
service EndpointDiscoveryService {
  rpc StreamEndpoints(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc FetchEndpoints(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```

模式都是通用的：

- 都有一个单次调用的 `Fetch***` 方法和一个gRPC双向流的  Stream***` 
- 而且四个xDS API的这两个方法的参数都是一样的：DiscoveryRequest / DiscoveryResponse
- RDS和CDS 还有用于增量更新的 `Delta***`，参数也相同：DeltaDiscoveryRequest / DeltaDiscoveryResponse

## ADS

ADS 的定义在 `api/envoy/service/discovery/v2/ads.proto`。

注意：ADS 是只能用于 gRPC 的API

```protobuf
service AggregatedDiscoveryService {
  rpc StreamAggregatedResources(stream envoy.api.v2.DiscoveryRequest)
      returns (stream envoy.api.v2.DiscoveryResponse) {
  }

  rpc DeltaAggregatedResources(stream envoy.api.v2.DeltaDiscoveryRequest)
      returns (stream envoy.api.v2.DeltaDiscoveryResponse) {
  }
}
```

模式其实和LDS/EDS/CDS/EDS 这四个xDS API的定义是一样的。



