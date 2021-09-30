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





