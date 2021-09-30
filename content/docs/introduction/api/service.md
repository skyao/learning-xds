---
title: "xDS的服务定义"
linkTitle: "服务定义"
weight: 1402
date: 2021-09-30
description: >
  xDS服务定义方式
---



## 通用模式

### LDS/RDS/CDS/EDS

LDS/RDS/CDS/EDS 这四个xDS API的定义非常类似，模式都是一致的。

LDS：

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

RDS：

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/route/v3/rds.proto

```protobuf
service RouteDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.config.route.v3.RouteConfiguration";

  rpc StreamRoutes(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc DeltaRoutes(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc FetchRoutes(discovery.v3.DiscoveryRequest) returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:routes";
    option (google.api.http).body = "*";
  }
}
```

CDS:

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/cluster/v3/cds.proto

```protobuf
service ClusterDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.config.cluster.v3.Cluster";

  rpc StreamClusters(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc DeltaClusters(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc FetchClusters(discovery.v3.DiscoveryRequest) returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:clusters";
    option (google.api.http).body = "*";
  }
}
```

EDS:

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/endpoint/v3/eds.proto

```protobuf
service EndpointDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.config.endpoint.v3.ClusterLoadAssignment";

  rpc StreamEndpoints(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc DeltaEndpoints(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc FetchEndpoints(discovery.v3.DiscoveryRequest) returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:endpoints";
    option (google.api.http).body = "*";
  }
}
```

LDS/RDS/CDS/EDS 这四个xDS API的定义方式是非常类似的：

- 都有一个单次调用的 `Fetch***()` 方法和一个gRPC双向流的 `Stream***()`方法，以及一个用于实现增量xDS的 `Delta***()`方法
- 而且四个xDS API的这3个方法的参数和应答都是一样的：DiscoveryRequest / DiscoveryResponse / DeltaDiscoveryRequest / DeltaDiscoveryResponse

### ADS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/discovery/v3/ads.proto

```properties
service AggregatedDiscoveryService {
  // This is a gRPC-only API.
  rpc StreamAggregatedResources(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc DeltaAggregatedResources(stream DeltaDiscoveryRequest)
      returns (stream DeltaDiscoveryResponse) {
  }
}
```

ADS的定义类似LDS/RDS/CDS/EDS，只是缺少单次调用的 `Fetch***()` 方法。

### LEDS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/endpoint/v3/leds.proto

```protobuf
service LocalityEndpointDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.config.endpoint.v3.LbEndpoint";

  // State-of-the-World (DiscoveryRequest) and REST are not supported.

  // The resource_names_subscribe resource_names_unsubscribe fields in DeltaDiscoveryRequest
  // specify a list of glob collections to subscribe to updates for.
  rpc DeltaLocalityEndpoints(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }
}
```

LEDS的定义也类似LDS/RDS/CDS/EDS，但只有一个 `Delta***()` 方法。

### SRDS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/route/v3/srds.proto

```protobuf
service ScopedRoutesDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.config.route.v3.ScopedRouteConfiguration";

  rpc StreamScopedRoutes(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc DeltaScopedRoutes(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc FetchScopedRoutes(discovery.v3.DiscoveryRequest) returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:scoped-routes";
    option (google.api.http).body = "*";
  }
}
```

SRDS的定义和LDS/RDS/CDS/EDS完全一致，三个方法都有。

### RTDS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/runtime/v3/rtds.proto

```protobuf
service RuntimeDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.service.runtime.v3.Runtime";

  rpc StreamRuntime(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc DeltaRuntime(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc FetchRuntime(discovery.v3.DiscoveryRequest) returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:runtime";
    option (google.api.http).body = "*";
  }
}
```

RTDS的定义和LDS/RDS/CDS/EDS完全一致，三个方法都有。

### SDS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/secret/v3/sds.proto

```protobuf
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

SDS的定义和LDS/RDS/CDS/EDS完全一致，三个方法都有。

### ECDS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/extension/v3/config_discovery.proto

```protobuf
service ExtensionConfigDiscoveryService {
  option (envoy.annotations.resource).type = "envoy.config.core.v3.TypedExtensionConfig";

  rpc StreamExtensionConfigs(stream discovery.v3.DiscoveryRequest)
      returns (stream discovery.v3.DiscoveryResponse) {
  }

  rpc DeltaExtensionConfigs(stream discovery.v3.DeltaDiscoveryRequest)
      returns (stream discovery.v3.DeltaDiscoveryResponse) {
  }

  rpc FetchExtensionConfigs(discovery.v3.DiscoveryRequest)
      returns (discovery.v3.DiscoveryResponse) {
    option (google.api.http).post = "/v3/discovery:extension_configs";
    option (google.api.http).body = "*";
  }
}
```

ECDS的定义和LDS/RDS/CDS/EDS完全一致，三个方法都有。

## 和通用模式类似

### HDS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/health/v3/hds.proto

```protobuf
service HealthDiscoveryService {
  rpc StreamHealthCheck(stream HealthCheckRequestOrEndpointHealthResponse)
      returns (stream HealthCheckSpecifier) {
  }

  rpc FetchHealthCheck(HealthCheckRequestOrEndpointHealthResponse) returns (HealthCheckSpecifier) {
    option (google.api.http).post = "/v3/discovery:health_check";
    option (google.api.http).body = "*";
  }
}
```

HDS的定义和LDS/RDS/CDS/EDS类似，定义有 `Fetch***()` 方法和 `Stream***()`方法。由于是健康检查，不存在增量，因此不需要定义 `Delta***()`方法。

另外就是Request/Response的消息体不再采用通用的消息体，而是HDS自己的定义，这也是因为健康检查的信息和LDS/RDS/CDS/EDS差异较大的原因。

### LRS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/load_stats/v3/lrs.proto

```protobuf
service LoadReportingService {
  rpc StreamLoadStats(stream LoadStatsRequest) returns (stream LoadStatsResponse) {
  }
}
```

LRS 只定义了 `Stream***()`方法，Request/Response的消息体也不采用通用的消息体，而是自己定义。

### MS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/metrics/v3/metrics_service.proto

```protobuf
service MetricsService {
  rpc StreamMetrics(stream StreamMetricsMessage) returns (StreamMetricsResponse) {
  }
}
```

MS 只定义了 `Stream***()`方法，Request/Response的消息体也不采用通用的消息体，而是自己定义。

### CSDS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/status/v3/csds.proto

```
service ClientStatusDiscoveryService {
  rpc StreamClientStatus(stream ClientStatusRequest) returns (stream ClientStatusResponse) {
  }

  rpc FetchClientStatus(ClientStatusRequest) returns (ClientStatusResponse) {
    option (google.api.http).post = "/v3/discovery:client_status";
    option (google.api.http).body = "*";
  }
}
```

CSDS 只定义了 `Stream***()`方法和 `Fetch***()` ，Request/Response的消息体也不采用通用的消息体，而是自己定义。

### TAP

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/tap/v3/tap.proto

```protobuf
service TapSinkService {
  rpc StreamTaps(stream StreamTapsRequest) returns (StreamTapsResponse) {
  }
}
```

TAP 只定义了 `Stream***()`方法，Request/Response的消息体也不采用通用的消息体，而是自己定义。

### TraceService

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/trace/v3/trace_service.proto

```protobuf
service TraceService {
  rpc StreamTraces(stream StreamTracesMessage) returns (StreamTracesResponse) {
  }
}
```

TraceService 只定义了 `Stream***()`方法，Request/Response的消息体也不采用通用的消息体，而是自己定义。

### EventReportingService

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/event_reporting/v3/event_reporting_service.proto

```protobuf
service EventReportingService {
  rpc StreamEvents(stream StreamEventsRequest) returns (stream StreamEventsResponse) {
  }
}
```

EventReportingService 只定义了 `Stream***()`方法，Request/Response的消息体也不采用通用的消息体，而是自己定义。

### ALS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/accesslog/v3/als.proto

```protobuf
service AccessLogService {
  rpc StreamAccessLogs(stream StreamAccessLogsMessage) returns (StreamAccessLogsResponse) {
  }
}
```

AccessLogService 只定义了 `Stream***()`方法，Request/Response的消息体也不采用通用的消息体，而是自己定义。

## 不采用通用模式

### RLS

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/ratelimit/v3/rls.proto

```protobuf
service RateLimitService {
  rpc ShouldRateLimit(RateLimitRequest) returns (RateLimitResponse) {
  }
}
```

### ExternalProcessor

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/ext_proc/v3alpha/external_processor.proto

```protobuf
service ExternalProcessor {
  rpc Process(stream ProcessingRequest) returns (stream ProcessingResponse) {
  }
}
```

### Authorization

https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/auth/v3/external_auth.proto

```protobuf
service Authorization {
  rpc Check(CheckRequest) returns (CheckResponse) {
  }
}
```

