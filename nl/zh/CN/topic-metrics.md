---

copyright:
  years: 2019
lastupdated: "2019-07-19"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 度量值
{: #metrics}

度量值是作为键/值对捕获的简单数字度量。一些度量值会使计数器递增，另一些度量值可执行聚集。例如，计算最近一分钟内收集的所有值的总和，或计算最近一分钟的平均耗用时间。有些度量值是简单的标尺，用于返回最后一次观察到的任何值。捕获和处理度量值可帮助您识别并响应潜在问题，以免它们升级并导致更严重的问题。
{:shortdesc}

在分布式系统度量值中有三个常规因素：生产者、聚集器和处理者。这些因素有一些常见的组合，例如将 Prometheus 作为聚集器与 Grafana（用于处理收集的度量值以在图形仪表板中显示）配合使用。再例如，将 StatsD 与 Graphite 配合使用。

![分布式系统度量值中的三个因素](images/metrics-systems.png "分布式系统度量值中的三个因素")

生产者是应用程序。在一些情况下，应用程序直接参与生成度量值。在另一些情况下，代理程序或其他基础架构被动观察或主动检测应用程序，以代表应用程序生成度量值。接下来发生的情况取决于聚集器。

度量值通过“推送”或“拉取”机制从生产者传输到聚集器。一些聚集器（如 StatsD）期望应用程序连接到聚集器以传输数据。必须将聚集器的连接信息分发到度量的所有应用程序进程。另一些聚集器（如 Prometheus）定期连接到已知端点来收集度量值数据。这需要生产者定义并提供可提取的端点，并且要向聚集器通知端点的位置。Prometheus 与 Kubernetes 一起使用时，可以基于服务注释来发现端点。

最后，处理器会使用所有聚集的数据。Grafana 等服务用于处理聚集的度量值，以使用仪表板进行显示。Grafana 还通过独立于仪表板存储和求值的规则来支持警报。

## 在 Kubernetes 中自动发现 Prometheus 端点
{: #prometheus-kubernetes}

基于拉取的模型已创建其自己的生态系统。其他聚集器（如 Sysdig）还可以从 Prometheus 端点中提取度量值数据。这可能意味着某些系统在不使用 Prometheus 服务器的情况下使用 Prometheus 度量值。

在 Kubernetes 环境中，注释用于 Prometheus 端点发现。例如，在端口 8080 上通过 HTTP 提供 `/metrics` 端点的服务会将以下注释添加到服务定义：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ...
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/scheme: http
    prometheus.io/path: /metrics
    prometheus.io/port: "8080"
  ...
```
{: codeblock}

然后，任何与 Prometheus 兼容的聚集器都可以使用 Kubernetes API 通过过滤该注释来发现这些端点。

## 应用程序与平台度量值
{: #app-platform-metrics}

需要收集哪些数据点？对于云本机应用程序，可以将度量值大体划分为应用程序和平台这两个类别：

* 应用程序度量值关注的是应用程序域实体。例如，在最近 5 分钟内完成了多少次登录？当前连接了多少个用户？在最近一秒内执行了多少个交易？收集特定于应用程序的度量值需要定制代码来收集和发布信息。大多数语言都具有度量值库，可简化添加定制度量值的过程。
* 相反，平台度量值托管的是域实体。例如，调用服务需要多长时间？此数据库查询需要多长时间？它们与流量和工作在系统中流动的方式相对应，并且通常可以在不对应用程序进行任何更改的情况下进行度量。某些应用程序框架还提供了用于度量这些关注点的内置支持，例如查询执行时间。

这两个类别本身并不足以让您真正决定需要度量的对象。在分布式系统中，有大量对象会生成度量值。有一些已知方法用于设法对大量度量值进行提炼，以减少到只包含必须关注的少数几个基本度量值：

* [四大黄金信号](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 是用于监视系统的基本度量值。这些基本度量值由 Google Site Reliability Engineering (SRE) 团队确定，用于在监视系统时实现服务级别的可观察性。用他们的话说，就是“如果对于面向用户的系统，只能度量四个度量值，请重点关注这四个度量值。”这四个度量值分别为等待时间、流量、错误和饱和度。
* [利用率、饱和度和错误 (USE) 方法](http://www.brendangregg.com/usemethod.html){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 设计为紧急核对表，用于分析系统的性能。USE 方法可以用一句话来概括：“检查每个资源的利用率、饱和度和错误。”在这种情况下，资源是指具有硬限制的物理或逻辑资源，如 CPU 或磁盘。对于每个有限资源，可度量利用率、饱和度和错误。 
* [RED 方法](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 是一种记忆术，派生自四大黄金信号，定义了对体系结构中每个微服务进行度量的三个关键度量值。此方法以请求为中心，尤其是与 USE 方法相比，此方法能很好地适应许多云本机应用程序的设计和体系结构。此方法的度量值为速率、错误和持续时间。 

USE 方法专注于基础架构度量。云本机环境旨在更好地利用物理或虚拟硬件。通过这些基础架构度量，可检查系统是否正确处理了负载。RED 方法完全专注于用于指示基础架构和应用程序是否存在问题的请求度量值。黄金信号的范围涵盖基础架构度量值和请求度量值，并将这两类度量值整合到一个整体视图中。

## 定义度量值
{: #defining-metrics}

通过监视来自单个服务的度量值，您可了解该服务的资源利用率。但是如果存在服务的多个实例，那么您需要区分这些实例，以便将问题与类似的传入数据相隔离。这最终会归结到命名上。在某些情况下，使用的度量值系统可能会对度量值施加结构限制。Prometheus 建议采用像 `namespace_subsystem_name` 这样的结构，而其他聚集器建议采用 `namespace.subsystem.targetNoun.actioned`。

例如，如果要跟踪股票交易应用程序执行的“交易”数，可以通过名为 `stock.trades` 的属性进行捕获。要区分实例，可以使用实例标识作为属性的前缀：`<instanceid>.stock.trades`。这支持收集各个实例值以及使用 `*.stock.trades` 来聚集数据。但是，部署到多个数据中心并希望以这种方式分析度量值时，会发生什么情况呢？您可以将名称更新为 `<datacenter>.<instanceid>.stock.trades`，但这将中断使用先前通配符 `*.stock.trades` 的任何报告。需要改为使用 `*.*.stock.trades`。 

如果仅使用分层命名的属性，一不小心就会导致与命名策略的任意结构联系在一起的脆弱通配符模式。脆弱通配符模式无助于您观察确保应用程序正常运行所需的信息。

通过支持维度数据的度量值系统，可以关联标识标签或标记。典型标签包括端点或服务名称、数据中心、响应代码、托管环境（生产/编译打包/开发/测试）或运行时标识（Java 版本和应用程序服务器信息）。

仍使用上面的股票交易示例，但这次 `stock.trades` 度量值与多个标签相关联，如 `{ instanceid=..., datacenter=... }`。这允许在不依赖通配符的情况下，按 `instanceid` 或 `datacenter` 对聚集的值进行过滤或分组。需兼顾考虑指定的度量值 `stock.trades` 与关联的标签。每个度量值用于捕获有意义的数据，而标签用于消歧。

请谨慎定义标签。在 Prometheus 中，键:值对的每个唯一组合被视为一个单独的时间序列。确保良好查询行为和有界数据收集的最佳做法是将标签与有限数量的允许值结合使用。 

如果使用的度量值用于统计错误数，那么可以使用返回码作为标签，其中值位于合理的集范围内。不要将标签用于失败的 URL。这是无界集。

有关命名度量值和标签的最佳实践的更多信息，请参阅 [Metric and Label Naming](https://prometheus.io/docs/practices/naming/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

## 更多注意事项
{: #metrics-considerations}

失败路径往往与成功路径有很大的不同。例如，如果失败涉及超时和堆栈跟踪收集，那么对 HTTP 资源的错误响应需要的时间可能要比成功响应长得多。请统计错误路径数，并将错误路径与成功请求分开来处理。

分布式系统在某些度量中具有自然变体。偶尔有错误是正常的，因为请求可能会定向到正在启动或正在关闭的进程。但当这种自然变体超出有效范围时，请过滤原始数据以捕获相关信息。例如，将度量值拆分成存储区。将请求持续时间分类为“最小/最快”、“中等/正常”和“最长/最大”等类别，如在滑动时间窗口中观察到的那样。如果请求持续时间始终位于“最长/最大”存储区中，那么您就可以识别问题。直方图或摘要度量值通常用于此类数据。有关更多信息，请参阅 [Histograms and Summaries](https://prometheus.io/docs/practices/histograms/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

确保应用程序和服务发出的度量值使用的是遵循组织范围内约定的名称和标签，以支持企业的监视工作。有关更多信息，请参阅 [Monitoring distributed systems](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。
