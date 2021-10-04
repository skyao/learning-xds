---
title: "xDS协议的API版本指南"
linkTitle: "[译]API版本指南"
weight: 1610
date: 2021-10-04
description: >
  xDS协议的API版本指南
---



> 内存翻译自 https://github.com/envoyproxy/envoy/blob/main/api/API_VERSIONING.md



# API版本指南

Envoy项目和 [xDS工作组](https://github.com/cncf/xds) 认真对待API的稳定性和版本问题。提供稳定的API是确保API采用和生态系统成功的必要步骤。下面我们阐述了旨在提供这种稳定性的API版本管理准则。

# API语义版本化

Envoy API由一系列包组成，例如：`envoy.admin.v2alpha`，`envoy.service.trace.v2`。每个包都是独立的版本，采用基于https://cloud.google.com/apis/design/versioning 的protobuf语义版本化方案。

包的主版本（major version）在其名称（和目录结构）中得到体现。例如，tracing API包的第二版被命名为 `envoy.service.trace.v2`，其组成的protos位于`api/envoy/service/trace/v2`中。每个protobuf都必须直接存在于版本包的命名空间中，我们不允许子包，如`envoy.service.trace.v2.somethingelse`。

小版本和补丁版本将在未来实施，这项工作在 https://github.com/envoyproxy/envoy/issues/8416 中进行跟踪。

在日常讨论和GitHub标签中，我们指的是`v2`、`v3`、`vN`、`...`等API。这有一个特定的技术含义。Envoy API中的任何特定信息，例如`envoy.config.bootstrap.v3.Bootstrap`，都会转而引用Envoy API中的一些包。这些包可能在`vN`，`v(N-1)`，等等。从技术上讲，Envoy API是一个版本包命名空间的DAG。当我们谈论`vN xDS API`时，我们实际上指的是根配置资源的`N`（例如bootstrap，xDS资源，如`Cluster`）。v3 API的引导配置是`envoy.config.bootstrap.v3.Bootstrap`，尽管它可能会转而引用`envoy.service.trace.v2`。

# 向后兼容

一般来说，在包的主API版本内，我们不允许任何破坏性的改变。指导原则是，无论是通讯格式还是protobuf编译器生成的语言绑定，都不应该在变化中出现不能向后兼容的变化。具体来说：

* 字段不应该被重新编号或改变其类型。这是标准的proto开发程序。
* 不能重命名proto的字段或包的命名空间。这在本质上是危险的，因为：
  * 字段重命名会破坏协议的兼容性。这比标准的proto开发程序更严格，因为它不会破坏二进制通讯格式。然而，它**会**破坏 YAML/JSON 加载到 protos 以及文本protos。由于我们认为 YAML/JSON 是第一类输入，我们不能改变字段名。
  * 对于服务定义，gRPC端点URL是由包的命名空间推断出来的，所以这将破坏客户/服务器的通信。
  * 对于嵌入 "Any" 对象的消息，type URL，即包命名空间的一部分，可以被Envoy或其他API消费代码使用。目前，这适用于嵌入 "DiscoveryResponse" 对象的顶级资源，例如 `Cluster`, `Listener` 等。
  * 消耗的代码将被破坏，需要修改源代码以配合API的变化。
* 其他一些变化被认为是对Envoy API的破坏，在protobuf的兼容性方面通常被认为是安全的:
  * 将一个单例字段升级为重复字段，例如`uint32 foo = 1;`升级为`repeated uint32 foo = 1`。
    这改变了JSON格式的表示，因此被认为是一个破坏性的改变。
  * 用 "oneof" 来包装一个现有的字段。这对 protobuf 或 JSON/YAML 格式没有影响，但对Go等语言中的各种消费存根有干扰，造成不必要的搅动。
  * 增加 [protoc-gen-validate](https://github.com/envoyproxy/protoc-gen-validate) 注释的严格性。如果这些更严格的条件是对已经在结构上或文档中隐含的行为进行建模，则可以允许有例外。

上述策略也有例外：

* 在引入新的API字段或消息后的14天内所做的更改，前提是该新字段或消息未被包含在 Envoy 发布中。
* 标记为 `vNalpha` 的API版本。在alpha主版本中，允许任意的破坏性改变。
* 任何带有`[#not-implemented-hide:...`注释的字段、消息或枚举。
* 任何带有 `(udpa.annotations.file_status).work_in_progress`,
  `(xds.annotations.v3.file_status).work_in_progress`
  `(xds.annotations.v3.message_status).work_in_progress` 的 proto, 或者
  `(xds.annotations.v3.field_status).work_in_progress` 选项注解。

请注意，对包装类型的默认值的改变，例如`google.protobuf.UInt32Value`，不受上述政策的约束。任何需要在Envoy API或主要版本内实现的稳定性的管理服务器都应该为这些字段设置明确的值。

# API生命周期

一个新的主版本是xDS API生态系统中的重大事件，不可避免地需要客户端（Envoy、gRPC）和大量的控制平面的支持，从简单的内部定制管理服务器到供应商运行的xDS即服务（xDS-as-a-service）产品。[xDS API牧羊人](https://github.com/orgs/envoyproxy/teams/api-shepherds) 将在以下限制条件下决定增加一个新的主要版本：

* 在现有支持的主版本中，xDS APIs存在足够的技术债务，以证明xDS客户端/服务器实现的成本负担。
* 自上一个主版本被削减以来，至少已经过去了一年。
* 与Envoy社区（通过Envoy社区会议、Slack上的`#xds`频道）以及gRPC OSS社区（通过与语言维护者联系）进行协商。这不是一个否决的过程；API牧羊人保留了在权衡这些投入和上述前两个考虑因素后推进新的主要API版本的权利。

在新的主版本发布后，API的生命周期遵循废弃时钟。Envoy在任何时候都会支持任何API包的最多三个主要版本：

* 当前稳定的主版本，如v3。
* 上一个稳定主版本，如v2。这是为了确保我们为所支持的主要版本提供至少1年的时间。通过同时支持两个稳定的主要版本，这使得控制平面和Envoy的推出也更容易协调。在新的当前稳定的主要版本推出后，之前的这个稳定的主要版本将被支持整整1年，之后它将从Envoy实现中移除。
* 可以选择下一个实验性的alpha主版本，例如v4alpha。这是下一个稳定大版本的候选发布版本。只有当当前的稳定大版本需要在下一个周期进行突破性的改变时，才会生成这个版本，例如，废弃或字段重命名。这个发布候选版本是通过 [protoxform](https://github.com/envoyproxy/envoy/tree/main/tools/protoxform) 工具从当前稳定的主要版本中机械地生成的，使用了诸如 `deprecated = true` 等注释。这不是一个可供人类编辑的工件。

一个例子是，在2020年12月底，如果v4主版本是合理的，我们可能会冻结`envoy.config.bootstrap.v4alpha`，然后这个包将成为当前稳定的主要版本`envoy.config.bootstrap.v4`。`envoy.config.bootstrap.v3`包将成为之前的稳定主版本，并且对`envoy.config.bootstrap.v2`的支持将从Envoy实现中放弃。需要注意的是，如果被引用的软件包没有发生变化，那么一些过渡性引用的软件包，例如`envoy.config.filter.network.foo.v2`在这个版本中可能仍然是2版本。如果此时没有主版本的理由，削减v4的决定可能会在2021年或以后的某个时间点发生，然而v2支持仍将在2020年底被移除。

这个API生命周期和时钟的含义是，Envoy API中任何被废弃的功能将至少保留1-2年的实现支持。

我们目前正在制定一个策略，引入次要版本（https://github.com/envoyproxy/envoy/issues/8416）。这将使xDS API的次要版本在每次废弃和字段引入/修改时都会发生变化。这将为控制平面提供一个机会，使其有条件支持客户端和主要/次要API版本。目前正在讨论，但没有最终确定的是，在一个主要版本中支持了一年之后，Envoy客户端将不再支持过时的功能。请将关于这个问题的任何想法发布到https://github.com/envoyproxy/envoy/issues/8416。

# 新的 API 特性

Envoy的API可以被 [安全扩展](https://cloud.google.com/apis/design/compatibility)，增加新的包、消息、枚举、字段和枚举值，同时保持[向后兼容](#backwards-compatibility)。对一个给定的包的API的添加通常应该只对*当前稳定的主版本*进行。这个政策的基本原理是：

* 该功能对使用当前稳定主版本的Envoy用户来说是立即可用的。如果该功能被放在 "vNalpha "中，情况就不是这样了。
* `vNalpha`可以从`vN`中机械地生成，而不需要开发者在两个位置都维护新功能。
* 我们鼓励Envoy用户从以前的版本转移到当前的稳定大版本，以使用新功能。

# 什么时候可以对软件包的上一个稳定的主要版本进行API修改？

作为一个务实的让步，我们允许在一个主要的API版本增加后的一个季度内对上一个稳定的主要版本进行API功能的增加。对上一个稳定的主版本的任何修改都必须以一致的方式反映在当前稳定的主要版本中。

# 如何进行跨主要版本的突破性修改

我们在一个主版本中保持[向后兼容](#backwards-compatibility)，但允许跨主版本的破坏性改变。这使得API的废弃、清理、重构和重组成为可能。Envoy API有一个风格化的工作流程来实现这一点。有两种规定的方法，取决于变化是机械的还是手动的。

## 机械破坏性变更

字段废弃、重命名等是机械性的改变，由 [protoxform ](https://github.com/envoyproxy/envoy/tree/main/tools/protoxform)工具支持。这些是由[注释](STYLE.md#api-annotations)指导的。

## 手动破坏性变更

手动更改不同于机械更改，如字段废弃，因为一般来说，它需要在Envoy中手动实现新的代码和测试。例如，如果一个开发者想在路由配置中把 `HeaderMatcher`  和 `StringMatcher` 统一起来，这可能是这类变化的一个候选者。需要采取以下步骤:

1. 新版本的功能，例如 `NewHeaderMatcher` 消息应该和引用字段一起被添加到路由配置原语的当前稳定主版本中。
2. 应该改变Envoy的实现，以便从(1)中添加的字段中使用配置。
   应该编写翻译代码（和测试），以便从现有的字段和消息映射到（1）。
3. 旧的消息/enum/字段/enum值应该被注释为废弃的。
4. 在下一个主版本中，`protoxform`将自动删除废弃的版本。

这种先做后破的方法确保了API主要版本的发布是可预测的、机械的，并且大部分的Envoy代码和测试变化都是由功能开发者而不是API所有者拥有的。除了上述过程之外，不会有重大的 "vN "举措来解决技术债务。

# 客户端功能

不是所有的客户端都会支持某个主API版本中的所有字段和特性。一般来说，最好是使用Protobuf语义来支持，例如：

* 忽略一个字段的内容就足以表明该支持在一个客户端中是缺失的。
* 如果需要对一系列客户端的支持，同时设置废弃的和新的方法来表达一个字段（在这里不涉及巨大的开销或操作）。

这种方法并不总是有效，例如：

* 一个路由匹配器的连接条件不应该仅仅因为客户端缺少实现匹配的能力而被忽略；这可能会导致路由策略被绕过。
* 客户端可能期望服务器以某种格式或编码提供响应，例如不透明的扩展配置的JSON编码的`Struct`-in-`Any`表示。

为了这个目的，我们有[客户端功能](https://www.envoyproxy.io/docs/envoy/latest/api/client_features)。

# One Definition Rule (ODR)

为了避免维护包的两个以上的稳定的主要版本，并应对钻石依赖，我们对包如何被过境引用增加了一个限制；包在其横向依赖集中最多可以有另一个包的一个版本。这意味着一些软件包在发布周期中会有一个重大的版本升级，只是为了让它们赶上其依赖关系的当前稳定版本。

通过对软件包之间的相互引用有严格的规定，可以避免这种复杂性和折腾。包的组织和 `BUILD` 的可见性约束应该被用来限制，以保持任何给定包的依赖树的浅层深度。

# 尽量减少折腾的影响

除了稳定性之外，API版本策略还有一个明确的目标，即尽量减少开发者对Envoy社区、其他API客户端（如gRPC）、管理服务器供应商和更广泛的API工具生态系统的开销。为了减少技术债务和支持API的发展，在主版本之间有一定数量的API折腾是可取的，但过多的API折腾会造成升级的成本和障碍。

我们认为弃用是*强制性的变化*。任何弃用都会在下一个稳定的API版本中被移除。

其他机械性的破坏性变化被认为是*谨慎性的*。这些变化包括字段重命名等，主要反映在protobuf注释中。`protoxform'工具可以决定通过推迟应用自由裁量的变化来尽量减少API的折腾，直到一个主版本周期，各自的信息正在经历一个强制性的变化。

Envoy的API结构有助于最大限度地减少版本间的流失。开发者应该设计和拆分包，使高流失率的程序，如HTTP连接管理器，被隔离在包中，并有一个浅的参考层次。
