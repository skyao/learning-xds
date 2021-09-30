---
title: "增量xDS"
linkTitle: "增量xDS"
weight: 1203
date: 2021-09-28
description: >
  增量xDS
---

{{% pageinfo color="primary" %}}
增量xDS
{{% /pageinfo %}}

## 增量xDS

Envoy通过xDS协议与控制面实现配置数据的交换。当控制面检测中配置变化（比如通过Kubernetes Watch到新service或者其他的CRD资源更新），会向Envoy发送一个discoveryResponse来将更新后的配置下发到Envoy。之后，Envoy主线程在接受到数据之后，通过向各个工作线程中追加配置更新事件来完成配置的实际更新和生效。

但是，需要注意的是，控制面下发的discoveryResponse是一个全量的配置。换言之，哪怕是修改了一条路由中的一个小小请求头匹配，所有Listener、Cluster、Router都必须更新，Envoy会用接受到的新配置替换旧配置。使用现有更新方案虽然逻辑简单明了，但是在负载较多、配置量较大时，会造成大量的流量浪费和不必要的计算开销。

尤其是对于Sidecar模式下的Envoy，该问题会更加的明显。网格中服务需要访问其他服务时，其流量首先会被Envoy Sidecar所拦截，之后由Sidecar将请求转发给对应的服务。由于Sidecar并不了解其代理的服务依赖网格中哪些其他服务，所以它会记录服务网格中所有服务的相关信息。但是，事实上一个网格服务往往只会依赖网格中少量的几个服务。

因为上述问题，Envoy社区提出了delta xDS方案，实现增量的xDS配置更新。简单的说，在delta增量更新方案中，当配置发生变化时，只有发生变化的一项配置（配置的最小单位为一个完整的proto message）需要下发和更新。

基于delta增量更新方案，可以实现以下三种新的功能：

- **Lazy loading**：按照具体需要订阅相关资源。全量xDS中，每个Envoy Sidecar都会缓存大量的Cluster配置，但是实际部分Cluster从未被访问过，甚至将数据流量导向此类Cluster的相关路由配置都不存在，此类的Cluster配置只会浪费内存和降低Envoy效率。使用delta增量更新方案，可以在实际的配置被使用时再订阅该资源，从控制面获取相关配置（首次访问性能会受到一定影响）。
- **增量更新**：当部分资源更新时，如某个Cluster配置发生变化，某条Router修改了参数，那么只有对应的Cluster或Router配置会被下发和更新。在负载较多、配置量较大时，该功能可以有效减少网络内因配置更新而引入的数据流量。
- **缓存擦除**（或者说on demand loading**）**：根据Envoy当前负载实际请求动态调整订阅资源类型，对于不再活跃的配置，取消订阅，从Envoy内存中擦除，直到相关配置再次被使用。通过该功能，可以有效限制Envoy配置所占用的数据量，在超大规模应用场景中，可以有效减少Envoy内存开销。



参考文档：

- [Envoy-入门介绍与xDS协议](https://zhuanlan.zhihu.com/p/108846492)
