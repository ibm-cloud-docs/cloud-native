---

copyright:
  years: 2019, 2020
lastupdated: "2020-03-23"

---

{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Metrics
{: #metrics}

Metrics are simple numerical measurements that are captured as key-value pairs. Some metrics are incrementing counters and others perform aggregations. For instance, the sum of all values collected within the last minute or the average elapsed time in the last minute. Some metrics are simple gauges that return whatever the last observed value was. Capturing and processing metrics can help you identify and respond to potential issues before they escalate and cause more serious problems.
{:shortdesc}

There are three general factors in distributed system metrics: producers, aggregators, and processors. There are common combinations of factors, like using Prometheus as the aggregator with Grafana processing collected metrics for display in graphical dashboards. Another is StatsD with graphite.

![The three factors in distributed system metrics](images/metrics-systems.png "The three factors in distributed system metrics"){: caption="Figure 1. The three factors in distributed system metrics" caption-side="bottom"}

The producer is the application. In some cases, the application is directly involved in producing metrics. In other cases, agents or other infrastructure either passively observe or actively instrument the application to produce metrics on its behalf. What happens next depends on the aggregator.

Metrics are transferred from the producer to the aggregator by way of either "push" or "pull" mechanisms. Some aggregators, like StatsD, expect the application to connect to the aggregator to transmit data. The connection information for the aggregator must be distributed to all of the application processes that are measured. Other aggregators, like Prometheus, periodically connect to a known endpoint to gather metrics data. This requires the producer to define and provide an endpoint that can be scraped, and for the aggregator to be told where the endpoints are. When used with Kubernetes, Prometheus can discover endpoints based on service annotations.

Finally, the processor uses all of the aggregated data. Services like Grafana process aggregated metrics for visualization by using dashboards. Grafana also supports alerts by rules that are stored and evaluated independently of the dashboard.

## Automatic discovery of Prometheus endpoints in Kubernetes
{: #prometheus-kubernetes}

The Prometheus pull-based model has fostered its own ecosystem. Other aggregators, like Sysdig, can also scrape metrics data from Prometheus endpoints, allowing systems to use Prometheus metrics without using the Prometheus server.

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

Any Prometheus-compatible aggregator can then discover these endpoints by using the Kubernetes API by filtering on the annotation.

## Application vs platform metrics
{: #app-platform-metrics}

Which data points need to be gathered? With a cloud-native application, you can broadly divide the metrics into two categories, application and platform:

* Application metrics focus on application domain entities. For example, how many logins completed in the last five minutes? How many users are currently connected? How many trades were performed in last second? Gathering application-specific metrics requires custom code to collect and publish the information. Most languages have metrics libraries to simplify adding custom metrics.
* Platform metrics, in contrast, are hosting domain entities. For example, how long does it take to invoke a service? How long does this database query take? They are aligned with how traffic and work are flowing through the system, and can often be measured without any changes to the application. Some application frameworks also provide built-in support for measuring these concerns, such as query execution times.

These categories aren't sufficient on their own to really decide what you need to measure. In a distributed system, there are lots of things producing metrics. There are a few known methods that attempt to distill the vast pool of metrics down to the essential few that must be watched:

* The [four golden signals](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals){: external} are the essential metrics for monitoring a system. They are identified by the Google Site Reliability Engineering (SRE) team for service-level observability when monitoring. In their words, "If you can only measure four metrics of your user-facing system, focus on these four." The four metrics are latency, traffic, errors, and saturation.
* The [Utilization Saturation and Errors (USE) method](http://www.brendangregg.com/usemethod.html){: external} is designed as an emergency checklist to analyze the performance of a system. The USE method can be summarized in a sentence, "For every resource, check utilization, saturation, and errors." A resource, in this case, physical or logical resources with hard limits, like CPUs or disks. For each finite resource, you measure utilization, saturation, and errors. 
* The [RED method](https://thenewstack.io/monitoring-microservices-red-method/){: external} is a mnemonic is derived from the four golden signals that defines three key metrics you measure for every microservice in your architecture. This method is request-centric, especially when compared to the USE method, which aligns well with the design and architecture of many cloud-native applications. The metrics are rate, errors, and duration. 

The USE method focuses on infrastructure metrics. Cloud-native environments are designed to make better use of physical or virtual hardware. These infrastructure measurements look at whether your system is properly handling load. The RED method focuses entirely on request metrics that indicate problems with infrastructure and applications. The golden signals span both, bringing infrastructure and request metrics together into a holistic view.

## Defining metrics
{: #defining-metrics}

Monitoring metrics from a single service might give you an idea of its resource utilization. But if there are multiple instances of the service, you need to distinguish them to isolate issues with similar incoming data. This ultimately comes down to naming. In some cases, the metrics system you're using might impose a structure on your metrics. Prometheus recommends something like `namespace_subsystem_name`, others recommend `namespace.subsystem.targetNoun.actioned`.

For example, if you wanted to track the number of "trades" that a stock trading application performed, you might capture it in a property called `stock.trades`. To distinguish between instances, you might prefix the property with the instance ID: `<instanceid>.stock.trades`. This supports collection of individual instance values and aggregates data by using `*.stock.trades`. But, what happens when you deploy to multiple data centers and you want to analyze metrics that way? You can update your name to be `<datacenter>.<instanceid>.stock.trades`, but that would break any reporting that uses the previous wildcard of `*.stock.trades`. It would need to use `*.*.stock.trades`, instead. 

Without care, use of only hierarchically named properties can lead to fragile wildcard patterns tied to the arbitrary structure of your naming strategy. Fragile wildcard patterns don't help you observe the information that you need to ensure that your application is functioning well.

With metrics systems that support dimensional data, you can associate identifying labels, or tags. Typical labels include endpoint or service name, data center, response code, hosting environment (prod/staging/dev/test), or runtime identifiers (Java version, app server information).

Using the same stock trading example, the `stock.trades` metric is associated with several labels, like `{ instanceid=..., datacenter=... }`. This allows the aggregated value to be filtered or grouped by `instanceid` or `datacenter` without relying on wildcards. There is a balance between the named metric, `stock.trades`, and the associated labels. Each metric captures meaningful data, with labels for disambiguation.

Define labels with caution. In Prometheus, every unique combination of key:value pairs is treated as a separate time series. A best practice to ensure good query behavior and bounded data collection is to use labels with a finite number of allowed values. For example, if you count the number of errors encountered by an HTTP endpoint, the HTTP return code (401, 404, 409, 500, ... ) is a good label. The failed requst URL, on the other hand, should not be used as a label, as there is no limit to the number of permutations for invalid URLs.

For more information on best practices for naming metrics and labels, see [Metric and Label Naming](https://prometheus.io/docs/practices/naming/){: external}.

## More considerations
{: #metrics-considerations}

A failure path is often wildly different than a success path. For example, an error response on an HTTP resource might take much longer than a successful response if the failure involved timeouts and stack trace collection. Count and treat error paths separately from successful requests.

A distributed system has natural variations in certain measurements. Occasional errors are normal, as requests might be directed to processes in the middle of starting up or shutting down. Filter the raw data to catch when this natural variation exceeds a valid range. For example, split metrics into buckets. Categorize request duration into categories like 'smallest/quickest', 'medium/normal', and 'longest/largest', as observed within a sliding time window. If request durations are consistently landing in the "longest/largest" bucket, you can identify a problem. Histogram or summary metrics are usually used for this kind of data. For more information, see [Histograms and Summaries](https://prometheus.io/docs/practices/histograms/){: external}.

Ensure that your applications and services emit metrics with names and labels that follow organization-wide conventions that support your business's monitoring efforts. For more information, see [Monitoring distributed systems](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/){: external}.
