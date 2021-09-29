---
title: "XDS API中的CDS概述"
linkTitle: "CDS概述"
weight: 801
date: 2021-09-28
description: >
  介绍Envoy的XDS API中的CDS
---

CDS API定义在 `api/envoy/api/v2/cds.proto`:

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