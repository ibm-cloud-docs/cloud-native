---

copyright:
  years: 2019
lastupdated: "2019-09-09"

---

{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Logging
{: #logging}

Log messages are strings with contextual information that pertains to the state and activity of a microservice when the log entry is created. Logs are required to diagnose how and why services fail. They play a supporting role to metrics in monitoring application health.
{:shortdesc}

Make sure that you write log entries directly to standard output and error streams. This makes log entries viewable by using command-line tools, and allows log forwarding services that are configured at the infrastructure level to manage log collection and data management.

Handling log files requires more thought if a containerized application can't be configured to write logs to standard out or standard err.

* One option is to use a volume for log data, whether a simple bind mount for local development and test, or a proper Persistent Volume as part of a Kubernetes deployment. A [dedicated sidecar or logging agent](https://kubernetes.io/docs/concepts/cluster-administration/logging/#sidecar-container-with-a-logging-agent){: external} can read from a shared volume to forward logs to a centralized aggregator. Log rotation must be configured explicitly to control the amount of log data stored on volumes.
* Another option is using application libraries or agents to forward logs directly to aggregators. This option can carry some configuration complexity across deployment environments

## JSON logging
{: #json-logging}

As your application evolves over time, the nature of what you log can change. By using a JSON log format, you gain the following benefits:

* Logs are indexable, which makes searching an aggregated body of logs much easier.
* Logs are resilient to change, as parsing isn't reliant on the position of elements in a string.

While JSON formatted logs are easier for machines to parse, they are harder for humans to read. Consider by using environment variables to toggle the log format between plain text for local development and debugging, and JSON-formatted logs for longer term storage and aggregation.

Command-line JSON parsers, like JSON Query tool (jq), can be used to create human-readable views of JSON-formatted logs. In the following example, the logs are piped through grep to ensure that the message field is there before jq parses the line:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | \
  grep message | \
  jq .message -r
```
{: pre}

## Viewing logs by using `kubectl`
{: #view-logs-kube}

Logs sent to standard output and error streams can be viewed in Kubernetes through the console or by way of `kubectl` commands that are of the form: `kubectl logs <podname>`.

If you use a custom namespace, such as stock-trader, include that in the command, `kubectl logs -n stock-trader <podname>`.

If there are multiple containers for each Pod, as there are with istio sidecars, you also need to specify the container. In the following example, the stock-trader namespace is used to view logs from the `portfolio` container in the `portfolio-54b4d579f7-4zvzk` Pod:

```bash
kubectl logs -n stock-trader portfolio-54b4d579f7-4zvzk -c portfolio
```
{: pre}

For logs in JSON format, you can use `jq` to extract a message field, for example:

```bash
kubectl logs trader-54b4d579f7-4zvzk -n stock-trader -c trader | grep message | jq .message -r
```
{: pre}

Log entries are piped through `grep` to ensure that `jq` parses lines with a message field.
{: note}
