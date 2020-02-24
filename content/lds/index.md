---
date: 2018-11-07T14:50:00+08:00
title: LDS
weight: 400
description : "介绍Envoy的XDS API中的LDS"
---

## LDS介绍

LDS是Listener Discovery Service的首字母缩写。

监听器发现服务（LDS）是一个可选的 API，Envoy 将调用它来动态获取监听器。Envoy 将协调 API 响应，并根据需要添加、修改或删除已知的监听器。

监听器更新的语义如下：

- 每个监听器必须有一个独特的名字。如果没有提供名称，Envoy 将创建一个 UUID。要动态更新的监听器，管理服务必须提供监听器的唯一名称。
- 当一个监听器被添加，在参与连接处理之前，会先进入“预热”阶段。例如，如果监听器引用 RDS 配置，那么在监听器迁移到 “active” 之前，将会解析并提取该配置。
- 监听器一旦创建，实际上就会保持不变。因此，更新监听器时，会创建一个全新的监听器（使用相同的侦听套接字）。新增加的监听者都会通过上面所描述的相同“预热”过程。
- 当更新或删除监听器时，旧的监听器将被置于 “draining（逐出）” 状态，就像整个服务重新启动时一样。监听器移除之后，该监听器所拥有的连接，经过一段时间优雅地关闭（如果可能的话）剩余的连接。逐出时间通过 `--drain-time-s` 选项设置。

> **注意**： 任何在 Envoy 配置中静态定义的监听器都不能通过 LDS API 进行修改或删除。

## LDS定义

https://github.com/envoyproxy/envoy/blob/master/api/envoy/api/v2/lds.proto



```protobuf
service ListenerDiscoveryService {
  rpc StreamListeners(stream DiscoveryRequest) returns (stream DiscoveryResponse) {
  }

  rpc FetchListeners(DiscoveryRequest) returns (DiscoveryResponse) {
    option (google.api.http) = {
      post: "/v2/discovery:listeners"
      body: "*"
    };
  }
}
```

