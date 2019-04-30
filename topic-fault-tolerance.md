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

# Fault tolerance
{: #fault-tolerance}

Fault tolerance is a system's ability to continue functioning in the event of partial failure. Creating a resilient system places requirements on all of the services in it. The dynamic nature of cloud environments demands that services be written to expect and respond gracefully to the unexpected, like receiving bad data, being unable to reach a required backing service, or dealing with conflicts due to concurrent updates in a distributed system. 

Fault tolerance solutions usually focus on timeouts, fallbacks, bulkheads, and circuit breakers.

In some environments, fault tolerance mechanisms might be provided by infrastructure components like Istio. Regardless of whether the infrastructure helps out, a service must assume that the remote call can fail and should be prepared with appropriate fallback actions.

## Timeouts

The first line of defense against partial failure is using timeouts. Timeouts ensure that your application receives an error when a backing service is non-responsive, allowing it to handle the condition with appropriate fallback behavior. This does not necessarily mean that the requested operation failed. Timeouts change how long a client making a request waits for a response; they have no impact on the processing behavior of the target service.

Many language libraries use a default timeout for requests, and Istio does, too. By default, the sidecar proxy fails the request if it does not receive a response in 15 seconds. This value can be changed by setting a timeout policy for the route in the VirtualService definition, which looks something like the following, using a service that returns stock quotes as an example:

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

With this configuration, requests made to the stock quote service through the sidecar proxy or ingress gateway wait for 30 seconds before failing the request, rather than the default value of 15 seconds.

The interesting aspect of tuning timeouts this way is that all requests using the route have the timeout applied. This is a base layer of safety provided by Istio: even if an application framework or library doesn't impose a timeout, it won't wait forever, because Istio does. Application-level timeouts still apply, however. Given the above example, the infrastructure-level timeout for the stock-quote service was extended to 30 seconds. If the application library sets a timeout for 5 seconds, the application's request still fail with a timeout.

## Fallbacks

An application should define what happens when a request to a backing service fails. There are a few options, but the goal is to degrade gracefully when these services don't respond in a timely manner. When a remote service fails, you might retry the request, try a different request, or return cached data instead.

Retrying the request is the easiest fallback mechanism at first glance. What isn't so apparent is that retrying requests can contribute to cascading system failures ("retry storms", a variation of the [thundering herd problem](https://en.wikipedia.org/wiki/Thundering_herd_problem)). Application-level code is not aware enough of system or network health, and exponential backoff algorithms are hard to get right.

Istio can perform retries much more effectively. It is already directly involved in request routing and it provides a consistent, language-agnostic implementation for retry policies. For example, we could define a policy like the following for our stock quote service:

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

With this simple configuration, requests made to the stock quote service through an Istio sidecar proxy or ingress gateway will be retried up to 3 times, with a 5 second timeout for each attempt. [Additional route matching rules](https://istio.io/docs/reference/config/istio.networking.v1alpha3/#HTTPMatchRequest) could further restrict this retry policy to `GET` requests, for example.

There is a nuance here that is easy to miss: you are not specifying the retry interval. The sidecar determines the interval between retries, and deliberately introduces "jitter" between attempts to avoid bombarding overloaded services.

## Bulkheads

In shipping, a bulkhead is a partition that prevents a leak in one compartment from sinking the whole ship. The pattern is applied in cloud environments to achieve a similar purpose, and is done in a few different ways.

For multi-threaded languages like Java, internal bulkheads can be used internally to constrain or control how resources are used for communication to remote resources, using either queue or a semaphore mechanisms:

- To use a queue, the service associates a specific number of threads with a particular queue. Any requests made once the queue is full get a fast failure. In Java, this could be a `ThreadPoolExecutor` associated with a `BlockingQueue`, as an example.
- The semaphore mechanism works by having a set number of permits. A outbound request requires a permit. After a request has completed successfully the permit is released for another request to use.

You can also define inter-service bulkheads using an Istio DestinationRule to constrain the connection pool for an upstream service, using something like the following example:

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

This configuration limits the maximum number of concurrent connections being made to the each instance of the stock quote service to 10; Services that can't make a connection within 30 seconds will get a `503 -- Service Unavailable` response. This kind of bulkhead can be used to prevent a compute-heavy service from receiving more requests than it can manage, for example.

## Circuit breakers

Circuit breakers are used to optimize the behavior of outbound requests when failures occur. Instead of repeatedly making requests to a non-responsive service, a circuit breaker observes the number of failures that have occurred within a given time period. If the error rate exceeds a threshold, the circuit breaker opens the circuit and fails the request, and all subsequent requests until the circuit is closed again.

Circuit breakers are also defined using an Istio DestinationRule:

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

This configuration puts constraints on requests that other services are making to the stock quote service. The specified `outlierDetection` traffic policy is applied to each individual instance. To phrase the preceding configuration as a sentence, "Eject any stock-quote-service instance that fails 3 times in 5 seconds for at least 5 minutes; further, all instances can be ejected". The latter `maxEjectionPercent` setting is related to load-balancing. Istio maintains a load-balancing pool, and ejects failing instances from that pool. By default, it ejects a maximum of 10% of all available instance from the load balancing pool.

For those familiar with other mechanisms of circuit breaking, Istio does not have a half-open state. It instead applies some simple math: an instance remains ejected from the pool for `baseInjectionTime * <number of times it has been ejected>`. This allows for instances to recover from transient failures, while keeping consistently failing instances out of the pool.

