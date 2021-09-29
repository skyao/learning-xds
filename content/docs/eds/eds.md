---
title: "EDS概述"
linkTitle: "EDS概述"
weight: 1001
date: 2021-09-28
description: >
  介绍Envoy的XDS API中的EDS
---

EDS API定义在 `api/envoy/api/v2/eds.proto`:

```protobuf
service EndpointDiscoveryService {
  rpc StreamEndpoints(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc FetchEndpoints(DiscoveryRequest) returns (DiscoveryResponse) {
  }
}
```