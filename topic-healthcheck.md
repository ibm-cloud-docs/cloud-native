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

# Health checking
{: #healthcheck}

Health checks provide a simple mechanism within automated systems to examine the health status of an individual instance. The system then responds to these health inspection events by taking an action, such as replacing a failed instance, updating routing tables, or communicating the resulting health states to the user.
{:shortdesc}

Kubernetes defines two integral mechanisms for checking the health of a container:

* A readiness probe is used to indicate whether the process can handle requests (is routable). Kubernetes doesn't route work to a container with a failing readiness probe. A readiness probe should fail if a service hasn't finished initializing, or is otherwise busy, overloaded, or unable to process requests.
* A liveness probe is used to indicate whether the process should be restarted. Kubernetes stops and restarts a container with a failing liveness probe to ensure that Pods in a defunct state are terminated and replaced. A liveness probe should fail if the service is in an irrecoverable state, for example, if an out of memory condition occurs. Simple liveness checks that always return an OK response can identify containers in an inconsistent state, which can happen when process serving requests has crashed but the container is still running.

Readiness and liveness probes are both defined using a similar structure that includes time delay and retry intervals, failure tolerance periods, timeouts as well as the definition of the probe implementation. The probe can be implemented by running a command, checking a TCP endpoint for connectivity, or performing an HTTP invocation. The same probe implementation can often be used for both readiness or liveness purposes, but the time delay and retry intervals need to be adjusted for the particular purpose.

## Understanding and applying probes
{: #kubernetes-probes}

Fundamentally, cloud-native application development is founded on the principal that container processes do in fact fail, but these processes are readily replaced by a new container. This happens both in response to unexpected events like container or machine failure, but also due to operational events like horizontal scaling and new application image rollouts. Readiness checks are important because they ensure that new container instances are ready to receive work prior to routing traffic, and the same checks prevent traffic from being routed to instances that have exited or are being destroyed.

When readiness checks aren't defined, Kubernetes has little insight into whether a container instance is ready to handle traffic, and routes traffic immediately after the container process is started. Without readiness checks, applications are more likely to experience connectivity timeouts and connection rejection responses when work is routed to an instance that is not ready to serve the request. Readiness checks reduce, but don't completely eliminate, client connectivity errors.

Even though changes to instance routing targets are normal within the lifecycle of a container-enabled application, the process states that liveness checks are intended to identify are less frequent and represent an exception rather than the norm. When a process enters a state from which no recovery is possible, the process is effectively inoperative. Some examples of why this might happen include out of memory conditions or a deadlock caused by a programming error. The best way to recover from situations like this is to terminate the container, which will also terminate any processing currently happening in the container as a result. This also creates the possibility of terminate or restart loops in the application, where containers are unable to completely come online before being terminated and replaced.

Readiness and liveness probes impact the system in different ways. This can be thought of in terms of state transition, where the positive state of a readiness check is routable and the negative state is unrouteable. Likewise, the positive state of a liveness check represents a container that is running normally and the negative state is inoperative. When a container starts, the readiness state is initially negative, and it enters a positive state only after the container is healthy. A liveness check starts in a positive state, and it enters a negative state only when the process becomes inoperative.

Configuring a readiness check very aggressively, such as with a low initial delay, has little effect since running the probe too soon doesn't cause the readiness check to change state. On the other hand, an aggressive liveness check, where the probe fires too soon, causes a state transition change, thus causing the system to terminate the container sooner than intended.

## Best practices for configuring probes
{: #probe-recommendation}

When implementing a health probe by using HTTP, consider the following HTTP status codes for readiness, liveness, and health.

| State    |  Readiness            |  Liveness             |
|----------|-----------------------|-----------------------|
|          | Non-OK causes no load | Non-OK causes restart |
| Starting | 503 - Unavailable     | 200 - OK              |
| Up       | 200 - OK              | 200 - OK              |
| Stopping | 503 - Unavailable     | 200 - OK              |
| Down     | 503 - Unavailable     | 503 - Unavailable     |
| Errored  | 500 - Server Error    | 500 - Server Error    |
{: caption="Table 1. HTTP status codes" caption-side="top"}

Health check endpoints must not require authorization or authentication. Since these protections are not put into place on health probe endpoints, restrict any HTTP probe implementations to GET requests that do not modify any data. Never return data that identifies specifics about the environment, like the operating system, implementation language, or software versions, as these can be used to establish an attack vector.

A liveness probe should be very deliberate about what it checks, because a failure results in immediate termination of the process. Avoid ambiguous metrics that only sometimes indicate a failing process, for example, a simple HTTP endpoint that always returns `{"status": "UP"}` with a 200 status code. Most processes that are in inoperative state fail this check, therefore correctly triggering a restart.

Health checks occur at frequent intervals, which can cause additional overhead. Readiness and liveness probes should test only the viability of backing services like databases or other microservices in their result when there isn't an acceptable fallback. For a liveness probe, a backing check should be included only if an unsuccessful result would cause the local container to enter an unrecoverable state. A readiness probe should verify a backing service only when the local container is unable to handle requests if it fails, but the condition is recoverable.

When configuring the initial time delay, a readiness probe should use the lowest likely value and a liveness check should use the largest likely time value. For instance, if an application server tends to start in 30 seconds, a typical readiness delay is 10 seconds. The liveness check uses a 60 second value to ensure that server start always completes before checking for terminatable states.

The *periodSeconds* attribute for routing decisions is typically configured to a single digit value, provided that the probe implementation is relatively lightweight. For instance, an HTTP probe that returns a 200 OK status without substantial server side processing has a minimal processor load and can be readily repeated every 1 - 5 seconds.

## Configuring probes in Kubernetes
{: #probe-config}

Declare liveness and readiness probes alongside your Kubernetes deployment in the container element. Both probes use the same configuration parameters:

| Parameter | Description |
|-----------|-------------|
| *initialDelaySeconds* | The amount of time that the kubelet waits after the container is created prior to the first probe. |
| *periodSeconds* | How frequently the kubelet probes the service. The default is 1. |
| *timeoutSeconds* | How quickly the probe times out. The default and minimum value is 1. |
| *successThreshold* | The number of times the probe must be successful after a failure. The default and minimum value is 1. The value must be 1 for liveness probes. |
| *failureThreshold* | The number of times that Kubernetes will try to restart a Pod before giving up when the Pod starts and the probe fails (see note). The minimum value is 1 and the default value is 3. |

  For a liveness probe, giving up means restarting the Pod. For a readiness probe, giving up means marking the Pod as unready.
  {: note}

To avoid restart cycles, set the `livenessProbe.initialDelaySeconds` parameter to be safely longer than it takes your service to initialize. You can then use a shorter value for the `readinessProbe.initialDelaySeconds` attribute to route requests to the service as soon as it's ready.

An example configuration might look something like the following example (note the path and port values):

```yaml
spec:
  containers:
  - name: ...
    image: ...
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 60
      timeoutSeconds: 5
    livenessProbe:
      httpGet:
        path: /liveness
        port: 8080
      initialDelaySeconds: 130
      timeoutSeconds: 10
      failureThreshold: 10
```
{: codeblock}

For more information, see [Configure Liveness and Readiness Probes in Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").
