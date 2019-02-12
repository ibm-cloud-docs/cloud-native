---

copyright:
  years: 2019
lastupdated: "2019-02-05"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Health, Readiness, and Liveness probes
{: #healthcheck}

Health checks provide a simple mechanism within automated systems to introspect an individual instance for health status. The system then responds to these health inspection events by taking an action, such as replacing a failed instance, updating routing tables, or communicating the resulting health states to the user through an observable property.

Kubernetes defines two integral mechanisms for checking the health of a container:

- A ***readiness*** probe is used to indicate whether the process can handle requests (is routable).

    Kubernetes doesn't route work to a container with a failing readiness probe. A readiness probe should fail if a service hasn't finished initializing, or is otherwise busy, overloaded, or unable to process requests.

- A ***liveness*** probe is used to indicate whether the process should be restarted.

    Kubernetes stops and restarts a container with a failing liveness probe to ensure that pods in a defunct state are terminated and replaced. A liveness probe should fail if the service is in an irrecoverable state, for example, if an out of memory condition occurs. Simple liveness checks that always return an OK response can identify containers in an inconsistent state, for example, where the process serving requests has crashed but the container is still running.

Readiness and liveness probes are both defined using a similar structure which includes time delay and retry intervals, failure tolerance periods, timeouts as well as the definition of the probe implementation. The probe can be implemented by executing a command, checking a TCP endpoint for connectivity or performing an HTTP invocation. Often the same probe implementation can be used for both readiness or liveness purposes, but the time delay and retry intervals will likely need to be adjusted for the particular purpose.

## Understanding and applying readiness and liveness probes in Kubernetes
{: #kubernetes-probes}

Fundamentally, cloud native application development is founded upon the principal that container processes do in fact fail, but these processes are readily replaced by a new container. This happens both in response to unexpected events like container or machine failure, but also due to operational events like horizontal scaling and new application image rollouts. Due to the fact that containers tend to come and go, readiness checks are important because they assure that new container instances are in fact ready to receive work prior to routing traffic, and the same checks prevent traffic from being routed to instances which have exited or are being destroyed. When readiness checks have not been defined, Kubernetes has little insight into whether a container instance is ready to handle traffic and will begin routing traffic immediately after the container process is started. Without readiness checks, client applications are more likely to experience connectivity timeouts and connection rejection responses when work is routed to an instance which is not ready to serve the request. Readiness checks reduce but will not completely eliminate such client connectivity errors.

While changes to instance routing targets are quite normal within the lifecycle of a container enabled application, the process states which liveness is intended to identify are less frequent and represent an exception rather than the norm. Sometimes a process will enter a state from which no recovery is possible, the process is effectively defunct. Some examples of why this might happen include out of memory conditions or a semaphore which has not been released due to a programming error. The counteraction to such events is termination of the container, an action which is invasive since any processing currently happening in the container is also terminated. This also creates the possibility of terminate/restart loops in the application, where containers are unable to completely come online before being terminated and replaced, only for the replacement container to undergo the same fate.

Readiness and liveness probes therefore impact the system in different ways. This can be thought of in terms of state transition, where the positive state of a readiness check is where it is routable, while the negative state is unrouteable. Likewise, the positive state of a liveness check represents a container which is running normally while the negative state is a defunct process. When a container starts, the readiness state will initially be negative, and only changes to positive once the container is healthy. In opposition, a liveness check starts in a positive state, and only enters a negative state when the process goes defunct. Because of this dichotomy, configuring a readiness check very aggressively, such as with a low initial delay will have little effect since running the probe too soon will not cause the readiness check to change state. On the other hand, an aggressive liveness check, where the probe fires too soon, will cause a state transition change causing the system to terminate the container sooner than intended.

## Configuring readiness and liveness probes in Kubernetes
{: #probe-config}

Declare liveness and readiness probes alongside your Kubernetes deployment in the container element. Both probes use the same configuration parameters:

- The kubelet waits for `initialDelaySeconds` after the container is created prior to the first probe.

- The kubelet probes the service every `periodSeconds` seconds. The default is 1.

- The probe times out after `timeoutSeconds` seconds. The default and minimum value is 1.

- The probe is successful if it succeeds `successThreshold` times after a failure. The default and minimum value is 1. The value must be 1 for liveness probes.

- When a pod starts and the probe fails, Kubernetes tries `failureThreshold` times to restart the pod before giving up. The minimum value is 1 and the default value is 3.

  - For a liveness probe, "giving up" means restarting the pod.

  - For a readiness probe, "giving up" means marking the pod `Unready`.

To avoid restart cycles, set `livenessProbe.initialDelaySeconds` to be safely longer than it takes your service to initialize. You can then use a shorter value for `readinessProbe.initialDelaySeconds` to route requests to the service as soon as it's ready.

An example configuration might look something like this (note the path and port values):

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

For more information, see how to [Configure Liveness and Readiness Probes in Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/).

## Recommendations for readiness and liveness probes
{: #probe-recommendation}

When implementing a health probe using HTTP, consider the following HTTP status codes for readiness, liveness and health.

| State    |  Readiness            |  Liveness             |
|----------|-----------------------|-----------------------|
|          | Non-OK causes no load | Non-OK causes restart |
| Starting | 503 - Unavailable     | 200 - OK              |
| Up       | 200 - OK              | 200 - OK              |
| Stopping | 503 - Unavailable     | 200 - OK              |
| Down     | 503 - Unavailable     | 503 - Unavailable     |
| Errored  | 500 - Server Error    | 500 - Server Error    |

Due to their role in automated environments, health check endpoints must not require authorization or authentication. Since these protections are not put into place on health probe endpoints, any HTTP probe implementations should be restricted to GET requests which do not modify any data. Never return data which identifies specifics about the environment, like the operating system, implementation language or software versions, as these can be used to establish an attack vector.

A liveness probe should be very deliberate about what it checks, as a failure results in immediate termination of the process. Avoid ambiguous metrics that only sometimes indicate a failing process. A simple http endpoint that always returns {"status": "UP"} with status code 200 may be a reasonable choice in many cases, as most processes that have entered a zombie state will fail this check, correctly triggering a restart. However, care must be taken that this does not become the "default" logic and simple tests are ignored.

Health checks occur at fairly frequent intervals, which can cause additional overhead. Readiness and liveness probes should only test the viability of downstream services like databases or other microservices in their result when there isn't an acceptable fallback. For a liveness probe, a downstream check should only be included if an unsuccessful result would cause the local container to enter an unrecoverable state for which the only remedy is a termination/restart. A readiness probe should only verify a downstream service when the local container is unable to handle requests if it fails, but the condition is recoverable.

When configuring the initial time delay (`initialDelaySeconds` attribute), a readiness probe should use the lowest likely value, while a liveness check should use the largest likely time value. For instance, if an application server tends to start in 30 seconds, a typical readiness delay would be 10 seconds, while the liveness check would use a 60 second value in order to assure that server start always completes before checking for terminatable states.

The `periodSeconds` attribute for routing decisions is typically configured to a single digit value, provided that the probe implementation is relatively lightweight. For instance, an http probe which returns a 200 OK status without substantial server side processing has a minimal processor load and can be readily repeated every 1-5 seconds.


## Advanced health checks

While the basic liveness and readiness probes are usefull to understand the health of the individual component, there is an opportunity to expose deeper and more diagnostic information using similar tecniques. 
For example, if a component reports an error of type `500 Server Error` it would be very useful to know if the component being tested had an internal error of it's own or if a dependency was at fault. 
`{ "State" : "500",
    "message" : "Timeout in access to database",
    {"dependecies": [
            "queue system A" : "200",
            "queue system B" : "200",
            "database A" : "404"
    ] }
}`

By executing this healhtcheck, we can now see the root-cause of the failue (at least from the perspective of the component being tested).
Because this healthcheck exposes more information about the topology of the environment, consider protecting it with an authentication/authorization layer. Another option may be to write out the result of the advanced health check to a log, whose access will be better protected than the HTTP endpoint.
Certain runtime frameworks, such as service meshes, may automatically expose this type of information from a central location.
