---
title: "xDS API中的RDS概述"
linkTitle: "RDS概述"
weight: 601
date: 2021-09-28
description: >
  xDS API中的RDS
---


RDS API定义在 `api/envoy/api/v2/rds.proto`:

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