---

copyright:
  years: 2019
lastupdated: "2019-02-11"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 容错
{: #fault-tolerance}

容错是指系统在发生局部故障时继续运行的能力。创建弹性系统会对系统中的所有服务施加需求。云环境的动态性质要求将服务编写为预期并正常响应意外情况，如收到错误数据，无法访问必需的后备服务，或处理因分布式系统中的并发更新而导致的冲突。 

容错解决方案通常专注于超时、回退、舱壁和断路器。

在某些环境中，容错机制可能由基础架构组件（如 Istio）提供。无论基础架构是否有所帮助，服务都必须假定远程调用可能失败，并且应该准备好相应的回退操作。

## 超时

预防局部故障的第一道防线是使用超时。超时可确保后备服务无响应时，应用程序可收到错误，从而允许应用程序使用相应的回退行为来处理该状况。这并不一定意味着所请求的操作失败。超时会更改发出请求的客户机等待响应的时间；超时不会影响目标服务的处理行为。

许多语言库对请求使用缺省超时，Istio 同样如此。缺省情况下，侧柜代理在 15 秒内未收到响应时，即会使请求失败。通过在 VirtualService 定义中为路径设置超时策略，可以更改此值，这类似于以下内容，其中以返回股票报价的服务为例：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    timeout: 30s
```
{: codeblock}

使用此配置时，通过侧柜代理或 Ingress 网关向股票报价服务发出的请求会等待 30 秒（而不是缺省值 15 秒），在此之后会使请求失败。

以此方式调整超时的比较有意思的一点是，使用该路径的所有请求都会应用该超时。这是 Istio 提供的基本安全层：即使应用程序框架或库未施加超时，也不会永远等待，因为 Istio 会使其超时。但是，应用程序级别的超时仍适用。在上面的示例中，股票报价服务的基础架构级别超时延长为 30 秒。如果应用程序库将超时设置为 5 秒，那么应用程序的请求仍会因超时而失败。

## 回退

应用程序应该定义对后备服务的请求失败时执行的操作。对此有若干选项，但目标是在这些服务无法及时响应时进行正常降级。远程服务失败时，您可能会重试请求，尝试其他请求，或改为返回高速缓存的数据。

乍一看，重试请求似乎是最容易的回退机制。但不太明显的问题是重试请求可能会造成级联系统故障（“重试风暴”，这是[惊群问题](https://en.wikipedia.org/wiki/Thundering_herd_problem)的一种变体）。应用程序级别的代码对系统或网络运行状况的了解不足，并且指数退避算法很难调整恰当。

Istio 执行重试的方式要有效得多。Istio 已直接参与请求路由，并为重试策略提供了一致且与语言无关的实现。例如，我们可以为股票报价服务定义类似于以下内容的策略：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-quote-service
spec:
  hosts:
  - 'stock-quote-service'
  http:
  - route:
    - destination:
        host: stock-quote-service
    retries:
      attempts: 3
      perTryTimeout: 5s
```
{: codeblock}

使用此简单配置时，通过 Istio 侧柜代理或 Ingress 网关对股票报价服务发出的请求将最多重试 3 次，每次尝试的超时为 5 秒。[其他路由匹配规则](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPMatchRequest)可进一步限制此重试策略，例如限制为 `GET` 请求。

此处有一个很容易忽略的细微差别：您未指定重试时间间隔。侧柜会确定重试时间间隔，并故意在两次尝试之间引入“抖动”，以避免连续对超负荷的服务进行访问。

## 舱壁

在航运中，舱壁是一种隔板墙，可防止因一个舱室发生漏水而导致整条船沉没。云环境中应用此模式来实现类似的目的，这可通过多种不同的方式来执行。

对于 Java 等多线程语言，内部舱壁可以在内部使用，通过队列或信号量机制来限制或控制资源如何用于与远程资源通信。

- 要使用队列，服务会将特定数量的线程与特定队列相关联。队列已满后发出的任何请求都会迅速收到失败。例如，在 Java 中，这可能是与 `BlockingQueue` 关联的 `ThreadPoolExecutor`。
- 信号量机制通过设置的许可数起作用。出站请求需要许可。一个请求成功完成后，会发布许可供另一个请求使用。

您还可以使用 Istio DestinationRule 来定义服务间舱壁，以限制用于上游服务的连接池，例如使用类似以下示例的内容：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 10
        connectTimeout: 30s
```
{: codeblock}

此配置将与股票报价服务的每个实例建立的最大并发连接数限制为 10 个；在 30 秒内无法建立连接的服务将收到 `503 - 服务不可用`响应。例如，这种类型的舱壁可用于防止计算量大的服务收到的请求数多于它可以管理的请求数。

## 断路器

断路器用于在发生失败时优化出站请求的行为。断路器会观察给定时间段内发生的失败数，而不是反复向无响应的服务发出请求。如果错误率超过阈值，那么断路器会使电路开路并使请求失败，同时会使所有后续请求失败，直到电路重新闭合为止。

断路器还可使用 Istio DestinationRule 进行定义：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock-quote-service
spec:
  host: stock-quote-service
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 3
      interval: 5s
      baseEjectionTime: 5m
      maxEjectionPercent: 100
```
{: codeblock}

此配置针对其他服务对股票报价服务发出的请求进行限制。指定的 `outlierDetection` 流量策略会应用于每个实例。要用一句话来描述上述配置，那就是“在至少 5 分钟内，弹出在 5 秒内失败 3 次的任何股票报价服务实例；超过此时间后，可以弹出所有实例”。后面的 `maxEjectionPercent` 设置与负载均衡相关。Istio 维护一个负载均衡池，并从该池中弹出失败的实例。缺省情况下，Istio 从负载均衡池中最多弹出所有可用实例的 10%。

如果您熟悉其他断路机制，那么会知道 Istio 没有半开状态。相反，Istio 应用的是一些简单的数学算法：一个实例保持从池中弹出的状态，持续时间为 `baseInjectionTime * <number of times it has been ejected>`。这允许实例从瞬态失败中恢复，同时让一直失败的实例保持在池之外。

