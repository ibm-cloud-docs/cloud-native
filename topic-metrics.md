---

copyright:
  years: 2019
lastupdated: "2019-03-26"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Metrics
{: #metrics}

Metrics are simple numerical measurements captured as key-value pairs. Some metrics are incrementing counters; others perform aggregations, like the sum of all values collected within the last minute, or the average elapsed time in the last minute. Some metrics are just simple gauges that return whatever the last observed value was. Capturing and processing metrics can help you identify and respond to potential issues before they escalate and cause more serious problems.
{:shortdesc}

There are three general factors when talking about metrics in a distributed system: producers, aggregators, and processors. There are some fairly common combinations of these factors, such as using Prometheus as the aggregator with Grafana processing collected metrics for display in graphical dashboards, or using StatsD with graphite.

![The three factors in distributed system metrics](images/metrics-systems.png "The three factors in distributed system metrics"){: caption="Figure 1. The three factors in distributed system metrics" caption-side="bottom"}

The producer is, of course, the application itself. In some cases, the application is directly involved in producing metrics. In other cases, agents or other infrastructure either passively observe or actively instrument the application to produce metrics on its behalf. What happens next depends on the aggregator.

Metrics are transferred from the producer to the aggregator by way of either "push" or "pull" mechanisms. Some aggregators, like StatsD, expect the application (or an agent on the application's behalf) to connect to the aggregator to transmit data. This requires connection information for the aggregator to be distributed to all of the application processes that should be measured. Other aggregators, like Prometheus, periodically connect to a known endpoint to gather (or scrape) metrics data. This requires the producer to define and provide an endpoint that can be scraped, and for the aggregator to be told where the endpoints are. When used with Kubernetes, Prometheus can discover endpoints based on service annotations.

Finally, the Processor is something that makes use of all that aggregated data. As mentioned earlier, services like Grafana process aggregated metrics for visualization using dashboards. Grafana also supports alerts by rules that are stored and evaluated independently of the dashboard.

## Automatic discovery of Prometheus endpoints in Kubernetes
{: #prometheus-kubernetes}

The pull-based model has created its own ecosystem. Other aggregators, like Sysdig, can also scrape metrics data from Prometheus endpoints. This might mean that some systems use Prometheus metrics without using the Prometheus server.

In Kubernetes environments, annotations are used for Prometheus endpoint discovery. For example, a service that provides a `/metrics` endpoint over HTTP on port 8080 adds the following annotations to the service definition:

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

Any Prometheus-compatible aggregator can then discover these endpoints using the Kubernetes API by filtering on the annotation.

## Application vs platform metrics
{: #app-platform-metrics}

Collecting metrics requires thought. Which data points should you gather? When considering a cloud-native application, you can broadly divide the metrics into two categories, application and platform:

* Application metrics focus on application domain entities. For example, how many logins completed in the last five minutes? How many users are currently connected? How many trades were performed in last second? Gathering application-specific metrics requires custom code to collect and publish the information. Most languages have metrics libraries to simplify adding custom metrics.
* Platform metrics, in contrast, are hosting domain entities. For example, how long does it take to invoke a service? How long does this database query take? They are aligned with how traffic and work is flowing through the system, and can often be measured without any changes to the application. Some application frameworks also provide built-in support for measuring these concerns, such as query execution times.

These categories aren't sufficient on their own to really decide what you need to measure. Remember that in a distributed system, there are lots of things producing metrics. There are a few well known methods that attempt to distill the vast pool of metrics down to the essential few that should be watched:

* The [four golden signals](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") are the essential metrics identified by the Google Site Reliability Engineering (SRE) team for service-level observability when monitoring a distributed system. In their words, "If you can only measure four metrics of your user-facing system, focus on these four." The four metrics are latency, traffic, errors, and saturation.
* The [Utilization Saturation and Errors (USE) method](http://www.brendangregg.com/usemethod.html){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") is designed as an emergency checklist to analyze the performance of a system. The USE method can be summarized in a sentence, "For every resource, check utilization, saturation, and errors." A resource, in this case, physical or logical resources with hard limits, like CPUs or disks. For each finite resource, you measure utilization, saturation, and errors. 
* The [RED method](https://thenewstack.io/monitoring-microservices-red-method/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") is a mnemonic is derived from the four golden signals that defines three key metrics you should measure for every microservice in your architecture. This method is very request-centric, especially when compared to the USE method, which aligns well with the design and architecture of many cloud native applications. The metrics are rate, errors, and duration. 

There are some common elements across these methods, all of them track the error rate, for example. But there are notable differences, too. The USE method focuses on infrastructure metrics (utilization/saturation). Cloud-native environments are designed to make better use of physical (or virtual) hardware. These infrastructure measurements look at whether or not your system is properly handling load. The RED method focuses entirely on request metrics (latency/duration, traffic/rate), which can indicate problems with the infrastructure (network problems) or with your applications (misconfigurations, deadlocks, repeated service failures, and so on), especially when combined with observed error rates. The golden signals span both, bringing infrastructure and request metrics together into a holistic view.

## Defining metrics
{: #defining-metrics}

Monitoring metrics gathered from a single service might give you an idea of its resource utilisation, but if there are multiple instances of the service (due to horizontal scaling or multi-region deployments), you need to distinguish between them to isolate issues within the mass of similar data coming in. This ultimately comes down to naming. In some cases, the metrics system you're using might impose a structure on your metrics. Prometheus, for example, recommends something like `namespace_subsystem_name`, others recommend `namespace.subsystem.targetNoun.actioned`.

For example, if you wanted to track the number of "trades" that a stock trading application performed, you might capture it in a property called `stock.trades`. To distinguish between instances, you might prefix the property with the instance ID: `<instanceid>.stock.trades`. This supports collection of individual instance values as well as aggregate data by using `*.stock.trades`. But, what happens when you deploy to multiple data centers and you want to analyze metrics that way? You can update your name to be `<datacenter>.<instanceid>.stock.trades`, but that would break any reporting that uses the previous wildcard of `*.stock.trades`. It would need to use `*.*.stock.trades`, instead. 

Without care, use of hierarchically named properties alone can lead to fragile wildcard patterns tied to the arbitrary structure of your naming strategy that don't help you observe the information you need to ensure your application is functioning well.

Metrics systems that support dimensional data allow you to associate additional identifying labels, or tags, with metrics data. You can then filter, group, or analyze collected metrics by using these additional dimensions without relying on wildcards and naming conventions. Typical labels include endpoint or service name, data center, response code, hosting environment (prod/staging/dev/test), or runtime identifiers (Java version, app server information).

Using the same stock trading example, the `stock.trades` metric is associated with several labels: `{ instanceid=..., datacenter=... }`, which allows the aggregated value to be filtered or grouped by `instanceid` or `datacenter` without relying on wildcards. There is a balance between the named metric (`stock.trades`) and the associated labels (compare this to the hierarchical example of `<datacenter>.<instanceid>.stock.trades`): each metric should capture meaningful data, with labels allowing for disambiguation where appropriate.

Also, be cautious when defining labels. In Prometheus, for example, every unique combination of key-value label pairs is treated as a separate time series. A best practice to ensure good query behavior and bounded data collection is to use labels with a finite number of allowed values. For example, if you use a metric that counts the number of errors, you can use the return code as a label, where values are within a reasonable set (401, 404, 409, 500, ... ), but you would not want a label for the failed URL, as that is an unbounded set (any request URL that failed for any reason, including being invalid).

For more information on best practices for naming metrics and labels (labels), see [Metric and Label Naming](https://prometheus.io/docs/practices/naming/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

## Additional considerations
{: #metrics-considerations}

When collecting your metrics, remember that a failure path is often wildly different than a success path. For example, an error response on a HTTP resource might take much longer than a successful response if the failure involved timeouts and stack trace collection. Count and treat error paths separately from successful requests.

A distributed system has natural variations in certain measurements. Occasional errors are normal, as requests might be directed to processes in the middle of starting up or shutting down. Filter the raw data to catch when this natural variation begins to exceed a valid range. For example, split metrics into buckets. Categorize request duration into categories like 'smallest/quickest', 'medium/normal', and 'longest/largest', as observed within a sliding time window. If request durations are consistently landing in the "longest/largest" butcket, you can identify a problem. Histogram or summary metrics are usually used for this kind of data. For more information, see [Histograms and Summaries](https://prometheus.io/docs/practices/histograms/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

As an application developer, ensure that your applications or services are emitting metrics with names and labels that follow organization-wide conventions to support monitoring efforts that are focused on end-to-end paths central to your business. For more information, see [Monitoring distributed systems](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").
