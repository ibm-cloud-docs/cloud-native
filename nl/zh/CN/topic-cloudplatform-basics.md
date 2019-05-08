---

copyright:
  years: 2019
lastupdated: "2019-02-18"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 云平台概念
{: #platform}

此部分简要概述了开发者在构建云本机应用程序时与之交互的核心技术和概念，这里会依次介绍容器、Kubernetes、Helm 和 Istio。
{:shortdesc}

## 容器
{: #containers}

容器是用于将应用程序及其所有依赖项打包成单个自包含单元的标准机制。容器解决了可移植性问题：容器工件（映像）确保应用程序需要运行的所有内容都位于正确的位置；然后，容器引擎可以专注于以高效、安全的方式将容器作为隔离进程运行。

容器映像通常是通过 `Dockerfile` 中定义的指令列表构建的。容器映像几乎总是基于其他容器映像而构建（本质上是已知先前状态的指令的延续）。例如，您可以使用以下片段来创建自己的 Open Liberty 映像：

```yaml
FROM open-liberty:kernel
COPY server.xml /config/
```
{: codeblock}

构建映像后，即可以运行该映像。容器执行引擎（如 Docker 或 [containerd](https://containerd.io/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")）会采用该映像定义，并将定义的入口点作为资源隔离的进程直接在主机操作系统上执行，从而避免了虚拟机的开销。

容器映像存储在*注册表*中。最为人熟知的是公共 Docker Hub 注册表，但更常见的做法是将映像推送到与基础架构和 CI/CD 管道更密切关联的访问控制的容器注册表（如 {{site.data.keyword.registryshort_notm}}）中，以及从这类注册表中拉取映像。

## Kubernetes
{: #kubernetes}

IBM 的云平台利用 Kubernetes 来进行容器编排。因此，除了了解容器基础知识外，开发者还必须熟悉 Kubernetes 基础知识，包括基本命令和部署工件。下表包含一些重要的 Kubernetes 概念：

|概念|描述|
|---------|-------------|
|Pod|一组本地化容器，一起部署为单个单元。Pod 是相对不可变的，需要通过替换原始 Pod 来修改 Pod 的各种属性。典型应用程序包含一个容器，容器具有核心业务逻辑和可选的附加 Pod，这些 Pod 在 Pod 的详细级别提供平台功能。|
|部署|无状态 Pod 的可重复模板，用于为 Pod 概念添加缩放维度。此外，还可以更新模板化的定义，并替换底层的 Pod 实例。Kubernetes 部署配置由 Kubernetes 部署控制器进行监视，以确保对于给定部署保持声明数量的 Pod。部署在 `.yaml` 文件中显示为 `kind: Deployment`。|
|服务|为人熟知的名称，表示一组相对不稳定的 Pod IP 地址。服务只能存在于集群专用网络上，或者可以在外部公开，通常使用特定于云提供者的负载均衡器。服务在 `.yaml` 文件中显示为 `kind: Service`。|
|Ingress|通过虚拟托管或基于上下文的路由，提供在多个服务中共享单个网络地址的能力。Ingress 还可以执行网络连接管理活动，如 TLS 终止。Ingress 在 `.yaml` 文件中显示为 `kind: Ingress`。|
|私钥|一个对象，用于存储 Pod 运行时使用的敏感信息，并将特定于部署的信息与容器映像或编排相分离。私钥可以通过环境变量或虚拟文件系统安装，在运行时公开给 Pod。不使用私钥时，敏感数据会存储在容器映像或编排中，这两种情况都会带来意外公开或意外访问这些数据的更多机会。|
|ConfigMap|作用与私钥类似，可将特定于部署的信息与容器编排相分离。但是，ConfigMap 是通用配置结构。它用于在运行时将信息（例如，命令行自变量、环境变量和其他配置工件）绑定到 Pod 的容器和系统组件。| 

所有资源都在 Kubernetes 资源模型中进行定义，可以通过 RESTful API 或通过使用 `kubectl` 命令行提交的配置文件进行配置。

有关更多信息，请参阅 [Kubernetes 基础知识](https://kubernetes.io/docs/tutorials/kubernetes-basics/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")、[Kubernetes 对象模型](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 和 [`kubectl` 命令行](https://kubernetes.io/docs/reference/kubectl/overview/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。 

## Helm
{: #helm}

Helm 是一个软件包管理器，可轻松查找、共享和使用为 Kubernetes 构建的软件。Helm 还能满足一个共同的用户需要：将同一应用程序部署到多个环境。Helm 使用 *chart*，这些是在安装时生成有效 Kubernetes 对象 (YAML) 的模板的集合。这些 chart 基于模板语言而构建，支持变量、范围操作以及能省去维护 Kubernetes 部署元数据所需的大量人工的其他内容。

有关更多信息，请参阅 [Helm](https://helm.sh/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

## Istio 服务网
{: #istio}

Istio 是一个开放式源代码平台，用于管理和保护微服务。Istio 与编排器（如 Kubernetes）配合使用，可管理和控制服务之间的通信。

Istio 使用侧柜模型来运行。侧柜（Envoy 代理）是一个单独的进程，与应用程序一起使用。侧柜可管理与服务之间的所有往来通信，并将公共级别的功能应用于所有服务，与用于构建服务的编程语言或框架无关。实际上，Istio 提供了一种机制，以集中方式配置路由和安全策略，而以分散方式通过侧柜来应用这些策略。

在大多数情况下，我们建议使用 Istio 提供的功能，而不是使用不同编程语言或框架提供的类似功能。例如，基础架构能以更一致的方式对负载均衡和其他路由策略进行定义、管理和强制实施。

在某些情况下，与分布式跟踪一样，Istio 和应用程序级别的库是互补的。您可以同时使用这两者来改进操作。对于分布式跟踪，Istio 只能确保存在跟踪头；应用程序库提供了有关请求之间关系的重要上下文。将 Istio 与支持库或框架库一起使用时，可提高您对系统的总体了解。

在最高级别，Istio 扩展了 Kubernetes 平台，提供了额外的管理概念、可视性和安全性。Istio 的功能可以细分为以下四个类别：

* 流量管理：控制微服务之间的流量，以执行流量分割、故障恢复和金丝雀发布。
* 安全性：在微服务之间提供基于身份的强认证、授权和加密。
* 可观察性：收集度量值和日志，以更好地了解集群中运行的应用程序。
* 策略：强制实施访问控制、速率限制和配额，以保护应用程序。

请参阅 [Istio 是什么？](https://istio.io/docs/concepts/what-is-istio/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")，以获取更多信息。



