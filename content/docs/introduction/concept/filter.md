---
title: "envoy的filter机制"
linkTitle: "filter"
weight: 1203
date: 2021-09-28
description: >
  envoy的filter机制：提供强大的可扩展能力
---

{{% pageinfo color="primary" %}}
Envoy通过Filter机制提供了极为强大的可扩展能力。
{{% /pageinfo %}}

### **Filter**：强大源于可扩展

Filter，通俗的讲，就是插件。通过Filter机制，Envoy提供了极为强大的可扩展能力。在Envoy中，很多核心功能都使用Filter来实现。比如对于Http流量和服务的治理就是依赖HttpConnectionManager（Network Filter，负责协议解析）以及Router（负责流量分发）两个插件来实现。利用Filter机制，Envoy理论上可以实现任意协议的支持以及协议之间的转换，可以对请求流量进行全方位的修改和定制。强大的Filter机制带来的不仅仅是强大的可扩展性，同时还有优秀的可维护性。Filter机制让Envoy的使用者可以在不侵入社区源码的基础上对Envoy做各个方面的增强。

Filter本身并没有专门的xDS服务来发现配置。Filter所有配置都是嵌入在LDS、RDS以及CDS（Cluster Network Filter）中的。





参考文档：

- [Envoy-入门介绍与xDS协议](https://zhuanlan.zhihu.com/p/108846492)
