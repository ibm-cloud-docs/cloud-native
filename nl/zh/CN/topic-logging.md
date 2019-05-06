---

copyright:
  years: 2019
lastupdated: "2019-02-10"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 日志记录
{: #logging}

日志消息是包含上下文信息的字符串，这些信息与创建日志条目时微服务的状态和活动相关。需要日志来诊断服务失败的方式和原因。在监视应用程序运行状况时，日志还可对度量值起到支持作用。
{:shortdesc}

确保将日志条目直接写入标准输出和错误流。这将使日志条目可通过命令行工具进行查看，并允许使用在基础架构级别配置的日志转发服务（如 Logstash、Filebeat 或 Fluentd）来管理日志收集和数据管理。

如果容器化应用程序无法配置为将日志写入标准输出或标准错误，那么处理日志文件需要考虑更多方面。

* 一个选项是将卷用于日志数据，不管是使用针对本地开发和测试的简单绑定安装，还是作为 Kubernetes 部署一部分的相应持久卷。[专用侧柜或日志记录代理程序](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 可以从共享卷中读取日志，以将日志转发到集中聚集器。必须显式配置日志循环，以控制存储在卷上的日志数据量。
* 另一个选项是使用应用程序库或代理程序将日志直接转发到聚集器。此选项可能会给各个部署环境带来一些配置复杂性。

## JSON 日志记录
{: #json-logging}

随着应用程序逐渐发展，记录的内容性质可能会变化。通过使用 JSON 日志格式，可获得以下优点：

* 日志是可索引的，这使搜索聚集的大量日志变得容易得多。
* 日志可弹性适应更改，因为解析并不依赖于元素在字符串中的位置。

虽然 JSON 格式的日志能使机器更容易解析，但加大了人类阅读的难度。请考虑使用环境变量来切换使用以下日志格式：纯文本（用于本地开发和调试）和 JSON 格式的日志（用于更长时间的存储和聚集）。

JSON Query 工具 (jq) 等命令行 JSON 解析器可用于创建 JSON 格式日志的人类可阅读的视图。在以下示例中，日志通过 grep 进行管道传输，以确保在 jq 解析行之前消息字段已经存在：

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## 使用 `kubectl` 查看日志
{: #view-logs-kube}

发送到标准输出和错误流的日志可以通过控制台或通过 `kubectl` 命令（格式为：`kubectl logs <podname>`）在 Kubernetes 中进行查看。

如果使用定制名称空间（例如，stock-trader），请将其包含在命令中，例如 `kubectl logs -n stock-trader <podname>`。

如果每个 Pod 有多个容器（正如在使用 Istio 侧柜时那样），那么还需要指定容器。在以下示例中，stock-trader 名称空间用于查看来自 `portfolio-54b4d579f7-4zvzk` Pod 中 `portfolio` 容器的日志：

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

对于 JSON 格式的日志，可以使用 `jq` 来抽取消息字段，例如：

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

日志条目通过 `grep` 进行管道传输，以确保 `jq` 仅对包含消息字段的行进行解析。
{: note}
