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

# Cloud platform concepts
{: #platform}

Learn about the core technologies and concepts that developers interact with when you build cloud-native applications, starting with Containers, Kubernetes, Helm, and Istio.
{:shortdesc}

## Containers
{: #containers}

Containers are a standard mechanism for packaging an application with all of its dependencies into a single, self-contained unit. Containers solve a portability issue. The container artifact, image, ensures that everything that an application needs to run is in the right place. Container engines focus on running containers as isolated processes in an efficient, safe, and secure way.

Container images are usually built from a list of instructions that are defined in a `Dockerfile`. Container images are almost always built from other container images (as essentially a continuation of instructions from known previous state). You can use the following snippet to make your own Open Liberty image, for example:

```yaml
FROM open-liberty:kernel
COPY server.xml /config/
```
{: codeblock}

After an image is built, it can be run. Container execution engines, like Docker or [containerd](https://containerd.io/){: external}, take that image definition and run the defined entrypoint as a resource-isolated process directly on the host operating system. The overhead of virtual machines is eliminatedd.

Container images are stored in *registries*. The best known is the public Docker Hub registry. You push images into and pull images from access-controlled container registries, like {{site.data.keyword.registryshort_notm}}, that are more closely associated with your infrastructure and CI/CD pipelines.

## Kubernetes
{: #kubernetes}

IBM's cloud platforms use Kubernetes for container orchestration. Therefore, in addition to knowing container basics, it is important for developers to be familiar with Kubernetes fundamentals, including basic commands and deployment artifacts. The following table includes some important Kubernetes concepts:

| Concept | Description |
|---------|-------------|
| Pod | A localized group of containers that are deployed together as a single unit. Pods are relatively immutable, requiring the original Pod to be replaced to modify various attributes of the Pod. A typical application has one container with core business logic and more Pods that provide platform capabilities at the granular level. |
| Deployment | A repeatable template for a stateless Pod, adding a dimension of scale to the Pod concept. Additionally, the templated definition can be updated and the underlying Pod instances replaced. A Kubernetes Deployment configuration is monitored by a Kubernetes Deployment Controller to ensure that the declared number of Pods for a Deployment are maintained. A Deployment is displayed as `kind: Deployment` in `.yaml` files. |
| Service | A well-known name that represents a set of relatively unstable Pod IP addresses. A Service can exist on the cluster private network only, or be exposed externally, by using a cloud provider-specific load balancer. A Service is displayed as `kind: Service` in `.yaml` files. |
| Ingress | The ability to share a single network address with multiple services by way of virtual hosting or context-based routing. An Ingress can also perform network connection management activities like TLS termination. An Ingress is displayed as `kind: Ingress` in `.yaml` files. |
| Secret | An object that stores sensitive information for Pod runtime use and separates the deployment-specific information from the container image or orchestration. A secret can be exposed to a Pod at run time through either environment variables or virtual file system mounts. Without secrets, sensitive data is stored in either the container image or the orchestration, both of which create more opportunities for accidental exposure or unintended access. |
| ConfigMap | Plays a similar role to Secrets in that it separates deployment-specific information from the container orchestration. However, a ConfigMap is a general-purpose configuration structure. It is used to bind information, such as command-line arguments, environment variables, and other configuration artifacts, to your Pod's containers and system components at run time. | 
{: caption="Table 1. Kubernetes concepts" caption-side="bottom"}

All resources are defined within the Kubernetes resource model, which can be configured either through the RESTful API or through configuration files that are submitted through the `kubectl` command line.

For more information, see [Kubernetes basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/){: external}, [Kubernetes Object Model](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)![External link icon](../icons/launch-glyph.svg "External link icon"), and [`kubectl` command line](https://kubernetes.io/docs/reference/kubectl/overview/){: external}. 

## Helm
{: #helm}

Helm is a package manager that provides an easy way to find, share, and use software built for Kubernetes. Helm also addresses a common user need: deploying the same application to multiple environments. Helm uses *charts*, which are collections of templates that produce valid Kubernetes objects (YAML) at installation time. These charts are built from a template language that includes support for variables, range operations, and other things that take the labor out of maintaining Kubernetes deployment metadata.

For more information, see [Helm](https://helm.sh/){: external}.

## Istio Service Mesh
{: #istio}

Istio is an open source platform for managing and securing microservices. It works with orchestrators like Kubernetes, providing a way to manage and control communication between services.

Istio operates by using a sidecar model. A sidecar (an Envoy proxy) is a separate process that sits alongside your application. The sidecar manages all communication to and from your service, and applies a common level of capability to all services independent of the programming language or framework the service with which they're built. In effect, Istio provides a mechanism to centrally configure routing and security policies, while having those policies applied by way of sidecars in a decentralized way.

Use capabilities that are provided by Istio instead of the similar capabilities that are provided by individual programming languages or frameworks. Load balancing and other routing policies are more consistently defined, managed, and enforced by the infrastructure, as an example.

In some cases, as with distributed tracing, Istio, and application-level libraries are complementary. You can improve operations by using both together. With distributed tracing, Istio can ensure only that trace headers are present; application libraries provide the important context about the relationships between requests. Your understanding of the system as a whole improves when both Istio and supporting libraries or framework libraries are used together.

At the highest level, Istio extends the Kubernetes platform, providing more management concepts, visibility, and security. The features of Istio can be broken down into the following four categories:

* Traffic management: Control the traffic between your microservices to perform traffic splitting, failure recovery, and canary releases.
* Security: Provide strong identity-based authentication, authorization, and encryption between microservices.
* Observability: Collect metrics and logs for better visibility into the applications that are running in your cluster.
* Policies: Enforce access controls, rate limits, and quotas to protect your applications.

See [What is Istio?](https://istio.io/docs/concepts/what-is-istio/){: external} for more information.



