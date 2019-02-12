---

copyright:
  years: 2019
lastupdated: "2019-02-06"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Observability, telemetry, and monitoring
{: #observability-cn}

There is a culture change around monitoring that comes with a shift to cloud native. Although applications in both on-premises and cloud-native environments are expected to be highly available and resilient to failure, the methods used to achieve those goals are very different. As a result, the purpose of monitoring shifts: instead of monitoring to avoid failure, monitor to manage failure. 
{:shortdesc}

In on-premises environments, infrastructure and middleware is provisioned based on planned capacity and high availability patterns, for example, active-active or active-passive. Unexpected failures can be complex in this environment, requiring significant effort for problem determination and recovery. External monitoring is performed by agents that examine resource utilization to avoid known classes of failures. As an example, consider the tuning of heap size, timeouts, and garbage collection policies for Java applications.

A cloud-native application is composed of independent microservices and the required backing services. Even though a cloud-native application as a whole must remain available and continue to function, individual service instances are started and stopped as necessary to preserve system function. 

## Observability
{: #observability}

Monitoring this fluid system requires each service instance to be observable. Each service instance should produce appropriate data to support automated problem detection and alerting, manual debugging when necessary, and analysis of system health (historical trends and analytics).

What kinds of data should a service produce to be observable?

* Health checks (often custom HTTP endpoints) help orchestrators, like Kubernetes or Cloud Foundry, perform automated actions to maintain overall system health and also enable simple deep-dive understanding of the behaviour of the systems and sub-systems.
* Metrics are a numeric representation of data collected at intervals into a time series. Numerical time series data is easy to store and query, which helps when looking for historical trends. Over a longer series of time, numerical data can be compressed into less granular aggregates, for example, daily, weekly, and so on.
* Log entries represent discrete events that have happened over time. Log entries are essential for debugging, as they often include stack traces and other contextual information that can help identify the root cause of observed failures.
* Distributed, request, or end-to-end tracing captures the end-to-end flow of a request through the system. Tracing essentially captures both relationships between services (the services the request touched), and the structure of work flowing through the system (synchronous or asynchronous processing, child-of or follows-from relationships).

## Telemetry
{: #telemetry}

Cloud-native applications should rely on the environment for *telemetry*, which is the automatic collection and transmission of data to centralized locations for subsequent analysis. This is emphasized by one of the twelve factors that states treat logs as event streams, and extends to all data a microservice produces to ensure it can be observed.

Kubernetes has some built-in telemetry capabilities, like Heapster, but it's more likely telemetry is provided by other systems that integrate with the Kubernetes control plane. Two of Istio's components, Mixer and Envoy, act together to transparently [collect telemetry from deployed applications](https://istio.io/docs/concepts/policies-and-telemetry/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

Failures are no longer rare, disruptive occurrences. Breaking a monolithic application into microservices pushes more of the mainline path onto the network, increasing the impact of latency and other network issues. Requests also reach processes that are not ready for work for any number of reasons. Services are automatically restarted if they run out of resources, and fault tolerance strategies allow the system as a whole to keep functioning. 

## Monitoring
{: #monitoring}

The concepts of observability and telemetry help highlight some significant differences in how cloud-native applications in large-scale distributed systems are monitored. Remember that processes in cloud-native environments are transient. Three of the twelve factors (processes, concurrency, and disposability) emphasize this point. Pre-allocated, long running, monolithic processes are replaced, or surrounded, by many more short-lived processes that are started and stopped in response to load for horizontal scaling or if they aren't functioning properly. Telemetry is critical when data must be collected and persisted somewhere else to prevent it from being lost as processes (containers) are created and destroyed. Telemetry is often required for compliance reasons as well. 

For all of the reasons previously described, monitoring changes focus: instead of monitoring the behavior and health of resources (individual processes or individual machines), the state of the system is monitored as a whole. Each individual service produces data that feeds into this aggregated view.

