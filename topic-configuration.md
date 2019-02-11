---

copyright:
  years: 2019
lastupdated: "2019-02-08"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Configuration
{: #configuration}

Cloud-native systems are highly automated by design. Part of the automation includes the abstraction of environment-specific logic from the core application. Applications might be developed on local hardware or vetted in a staging environment before they are deployed to production. The applications need to be environment-agnostic, as these different environments have unique properties and protocols for accessing information about the underlying infrastructure. Effective abstraction and automation are critical to ensuring cloud-native applications and microservices function properly regardless of where they run.
{:shortdesc}

The key principle of configuration is that cloud-native applications must be portable. You should be able to use the same fixed artifact to deploy to multiple environments, for example, local hardware and cloud-based test and production environments, without changing code or exercising otherwise untested code paths. Three factors from the [twelve factor approach](https://12factor.net/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") directly relate to this best practice:

* The first factor recommends a 1-to-1 correlation between a running service and a versioned codebase. This means creating a fixed deployment artifact, like a Docker image, from a versioned codebase that can be deployed, unchanged, to multiple environments.
* The third factor recommends a separation between application-specific configuration, which should be part of the fixed artifact, and environment-specific configuration, which should be provided to, or injected into, the service at run time.
* The tenth factor recommends keeping all environments as similar as possible. Environment-specific code paths are difficult to test, and increase the risk of failures as you deploy to different environments. This also applies to backing services. If you develop and test with an in-memory database, unexpected failures might occur in test, staging, or production environments because they use a database that has different behavior.

## Environment configuration
{: #config-inject}

Application-specific configuration should be part of the fixed artifact. For example, applications that run on WebSphere Liberty define a list of installed features that control the binaries and services that are active in the runtime. This configuration is specific to the application, and should be included in the Docker image. Docker images also define the listening, or exposed, port as the execution environment handles the port mapping when starting the container.

Environment-specific configuration, like the host and port used to communicate with other services, database users, or resource utilization constraints, are provided to the container by the deployment environment. The management of service configuration and credentials can vary significantly:

* Kubernetes stores configuration values (stringified JSON or flat attributes) in either ConfigMaps or Secrets. These can be passed to the containerized application as environment variables or virtual file system mounts. The mechansim used by a service is specified in the deployment metadata, either in Kubernetes YAML or the helm chart).
* Local development environments are often simplified variants that use simple key-value environment variables.
* Cloud Foundry stores configuration attributes and service binding details in stringified JSON objects that are passed to the application as an environment variable, for example, `VCAP_APPLICATION` and `VCAP_SERVICES`.
* Using a backing service, like etcd, hashicorp Vault, Netflix Archaius, or Spring Cloud config, to store and retrieve environment-specific configuration attributes is also an option in any environment.

In most cases, an application processes an environment-specific configuration at start time. The value of environment variables, for example, cannot be changed after a process has started. Kubernetes and backing configuration services, however, provide mechanisms for applications to dynamically respond to configuration updates. This is an optional capability. In the case of stateless, transient processes, restarting the service is often sufficient.
{: note}

Many languages and frameworks provide standard libraries to aid applications in both application-specific and environment-specific configurations so that you can focus on the core logic of your application and abstract these foundational capabilities.

### Working with service credentials
{: #portable-credentials}

The management of service configuration and credentials (service bindings) varies between platforms. Cloud Foundry stores service binding details in a stringified JSON object that is passed to the application as a `VCAP_SERVICES` environment variable. Kubernetes stores service bindings as stringified JSON or flat `ConfigMaps` or `Secrets` attributes in, which can be passed to the containerized application as environment variables or mounted as a temporary volume. In the case of local development, which has its own configuration, local testing is often a simplified version of whatever is running in the cloud. Working across these variations in a portable way without having environment-specific code paths can be challenging.

In Cloud Foundry and Kubernetes environments, you can use [service brokers](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") to manage binding to a backing service and injecting the associated credentials into the application's environment. This can impact application portability as credentials might not be provided to the application the same way in different environments.

{{site.data.keyword.IBM}} has several open source libraries that work with a `mappings.json` file to map the key that the application uses to retrieve credential information to an ordered list of possible sources. It supports three search pattern types:

* `cloudfoundry`: Search for a value in Cloud Foundry's services (VCAP_SERVICES) environment variable.
* `env`: Search for a value that is mapped to an environment variable.
* `file`: Search for a value in a JSON file.

In the following example `mappings.json` file, `cloudant-password` is the key that application code uses to look up the password credential. A language-specific library iterates through the `searchPatterns` array in a specific order until a match is found.

```json
{
   "cloudant_password": {
      "searchPatterns": [
         "cloudfoundry:$['cloudant'][0].credentials.password",
         "env:cloudant_password",
         "file:/server/localdev-config.json:$.cloudant_password"
      ]
   }
}
```
{: codeblock}

The library searches the following places for the cloudant password:

* The `['cloudant'][0].credentials.password` JSON path in the Cloud Foundry `VCAP_SERVICES` environment variable.
* A case-insensitive environment variable named cloudant_password`.
* A **cloudant_password** JSON field in a **`localdev-config.json`** file that is kept in a language-specific resource location.

For more information, see:

* [{{site.data.keyword.Bluemix}} Environment for Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [{{site.data.keyword.Bluemix_notm}} Environment for Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")
* [{{site.data.keyword.Bluemix_notm}} Service Bindings For Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon")

## Configuration values in Kubernetes
{: #config-kubernetes}

Kubernetes provides a few different ways to define environment variables and assign their values.

### Literal values
{: #config-literal}

The most straightforward way to define an environment variable is to include it directly in the `deployment.yaml` file for the service. In the following [basic Kubernetes example](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"), the direct specification works well when the value is consistent across environments:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```
{: codeblock}

### Helm variables
{: #config-helm}

As previously mentioned, Helm uses templates to create charts so that values can be substituted later. You can achieve the same result as in the preceding example, with more flexibility across environments, by using the following example in the `mychart/templates/pod.yaml` file template:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: {{ .Values.greeting }}
    - name: DEMO_FAREWELL
      value: {{ .Values.farewell }}
```
{: codeblock}

And the following example in a `mychart/values.yaml` file:

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

The following output is produced when Helm renders the template:

```bash
$ helm template mychart
---
# Source: mychart/templates/pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: Hello from the environment
    - name: DEMO_FAREWELL
      value: Such a sweet sorrow
```
{:screen}

  There is a minor difference between these two examples. In the first example and the sample `values.yaml` file, a human added quotation marks. Quotation marks aren't required for strings in YAML. When Helm renders the template, the quotation marks are left out.
  {: note}

### ConfigMap
{: #kubernetes-configmap}

A ConfigMap is a unique Kubernetes artifact that defines data as a set of key-value pairs. A ConfigMap for the environment variables shown in the preceding examples can look like the following example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: envar-demo-config
  labels:
    purpose: demonstrate-envars
data:
  DEMO_GREETING: Hello from the environment
  DEMO_FAREWELL: Such a sweet sorrow
```
{: codeblock}

The initial Pod definition is then changed to use values from the ConfigMap as follows:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    envFrom:
    - configMapRef:
        name: envar-demo-config
```
{: codeblock}

The ConfigMap is now a completely separate artifact from the Pod. It can have a completely different lifecycle. You can update or change the values in the ConfigMap without having to redeploy the Pod in this case. You can also update and manipulate a ConfigMap directly from the command line, which can be useful in the dev/test/debug cycle.

When used with Helm, you can use variables in your ConfigMap declaration. Those variables are resolved as usual when the chart is deployed.

For more information, see [Kubernetes ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

### Credentials and Secrets
{: #kubernetes-secrets}

Configuration is generally provided to containers running in Kubernetes by way of either environment variables or ConfigMaps. In either case, config values can be discovered fairly quickly. This is why Kubernetes uses Secrets to store sensitive information.

Secrets are independent objects that contain base64 encoded values:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```
{: codeblock}

Secrets can then be used as files in a volume mounted on one or more of a Pod's containers:

```yaml
containers:
- name: myservice
  image: myimage:latest
  volumeMounts:
  - name: foo
    mountPath: "/etc/foo"
    readOnly: true
volumes:
- name: foo
  secret:
    secretName: mysecret
```
{: codeblock}

When mounted as a volume in this way, each key in the Secret's data map becomes a file name under the specified `mountPath`: `/etc/foo/username` and `/etc/foo/password` in this case.

Secrets can also be used to define environment variables:

```yaml
containers:
- name: mypod
  image: myimage
  env:
  - name: SECRET_USERNAME
      valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
  - name: SECRET_PASSWORD
    valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
```
{: codeblock}

Kubernetes does the base64 decoding for you. The container running in the Pod recognizes the base64 decoded value when retrieving the environment variable.

As with ConfigMaps, Secrets can be created and manipulated from the command line, which comes in handy when dealing with SSL certificates.

For more information, see [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon").

<!-- SSL EXAMPLE -->
