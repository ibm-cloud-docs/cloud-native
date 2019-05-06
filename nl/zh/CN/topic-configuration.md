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

# 配置
{: #configuration}

云本机应用程序必须可移植。您应该能够使用相同的固定工件来部署到多个环境（从本地硬件部署到基于云的测试和生产环境），而无需更改代码或执行其他未测试的代码路径。
{:shortdesc}

[十二要素方法](https://12factor.net/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中的三个要素与此实践直接相关：

* 第一个要素建议正在运行的服务与已版本化的代码库之间建立一对一的关联。这意味着通过可原样部署到多个环境的已版本化代码库，创建固定部署工件，如 Docker 映像。
* 第三个要素建议将特定于应用程序的配置（应该是固定工件的一部分）与特定于环境的配置（应在部署时提供给服务）相分离。
* 第十个要素建议尽可能使所有环境保持相似。特定于环境的代码路径很难进行测试，并且在部署到不同环境时会增加发生故障的风险。后备服务也有同样的问题。如果使用内存内数据库进行开发和测试，那么在测试、编译打包或生产环境中可能会发生意外故障，因为这些环境中使用的数据库具有不同的行为。

## 配置源
{: #config-inject}

特定于应用程序的配置应该是固定工件的一部分。例如，在 WebSphere Liberty 上运行的应用程序定义了已安装功能部件的列表，这些功能部件用于控制在运行时中处于活动状态的二进制文件和服务。此配置特定于应用程序，并且应该包含在 Docker 映像中。Docker 映像还会定义正在侦听或已公开的端口，因为在启动容器时，执行环境会处理端口映射。

部署环境向容器提供特定于环境的配置，如用于与其他服务进行通信的主机和端口、数据库用户或资源利用率约束。服务配置和凭证的管理可能有很大差异：

* Kubernetes 将配置值（字符串化的 JSON 或平面属性）存储在 ConfigMap 或 Secret 中。这些值可以作为环境变量传递到容器化应用程序，也可以作为虚拟文件系统安装。服务使用的机制在 Kubernetes YAML 或 Helm chart 中的部署元数据中指定。
* 本地开发环境通常是使用简单的键/值环境变量的简化变体。
* Cloud Foundry 将配置属性和服务绑定详细信息存储在字符串化的 JSON 对象中，这些对象作为环境变量（例如，`VCAP_APPLICATION` 和 `VCAP_SERVICES`）传递到应用程序。
* 在任何环境中，还可以选择使用后备服务（如 etcd、HashiCorp Vault、Netflix Archaius 或 Spring Cloud Config）来存储和检索特定于环境的配置属性。

在大多数情况下，应用程序会在启动时处理特定于环境的配置。例如，在进程启动后无法更改环境变量的值。但是，Kubernetes 和后备配置服务为应用程序提供了动态响应配置更新的机制。这是可选功能。对于无状态的瞬态进程，重新启动服务通常足以满足要求。
{: note}

许多语言和框架都提供了标准库，用于支持应用程序使用特定于应用程序的配置和特定于环境的配置，以便您可以专注于应用程序的核心逻辑，并对这些基础功能进行抽象化处理。

### 使用服务凭证
{: #portable-credentials}

服务配置和凭证（服务绑定）的管理因不同的平台而有所不同。Cloud Foundry 是将服务绑定详细信息存储在字符串化的 JSON 对象中，该对象会作为环境变量 `VCAP_SERVICES` 传递到应用程序。Kubernetes 是将服务绑定作为字符串化的 JSON 或者作为平面 `ConfigMap` 或 `Secret` 属性来存储，可以将其作为环境变量传递到容器化应用程序，或作为临时卷安装。在本地开发（具有其自己的配置）的情况下，本地测试通常是在云中运行的任何内容的简化版本。在不具备特定于环境的代码路径的情况下，以可移植方式应对这些变化因素可能很困难。

在 Cloud Foundry 和 Kubernetes 环境中，可以使用[服务代理程序](https://cloud.ibm.com/apidocs/ibm-cloud-osb-api){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 来管理如何绑定到后备服务以及如何将关联的凭证注入到应用程序的环境中。这可能会影响应用程序可移植性，因为在不同环境中凭证可能不会以相同方式提供给应用程序。

{{site.data.keyword.IBM}} 具有多个开放式源代码库，这些库使用 `mappings.json` 文件来映射应用程序用于将凭证信息检索到可能源的有序列表的密钥。它支持三种搜索模式类型：

* `cloudfoundry`：在 Cloud Foundry 服务 (VCAP_SERVICES) 环境变量中搜索值。
* `env`：搜索映射到环境变量的值。
* `file`：在 JSON 文件中搜索值。

在以下示例 `mappings.json` 文件中，`cloudant-password` 是应用程序代码用于查找密码凭证的密钥。特定于语言的库以特定顺序在整个 `searchPatterns` 数组中进行迭代，直到找到匹配项。

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

库将在以下位置搜索 Cloudant 密码：

* Cloud Foundry `VCAP_SERVICES` 环境变量中的 `['cloudant'][0].credentials.password` JSON 路径。
* 名为 `cloudant_password` 的不区分大小写的环境变量。
* **`localdev-config.json`** 文件中的 **cloudant_password** JSON 字段，该文件保存在特定于语言的资源位置。

有关更多信息，请参阅：

* [{{site.data.keyword.Bluemix}} Environment for Go](https://github.com/ibm-developer/ibm-cloud-env-golang){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [{{site.data.keyword.Bluemix_notm}} Environment for Node](https://github.com/ibm-developer/ibm-cloud-env){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")
* [{{site.data.keyword.Bluemix_notm}} Service Bindings For Spring Boot](https://github.com/ibm-developer/ibm-cloud-spring-bind){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")

## Kubernetes 中的配置值
{: #config-kubernetes}

Kubernetes 提供了多种不同的方法来定义环境变量并分配其值。

### 字面值
{: #config-literal}

定义环境变量最简单的方法是直接将其包含在服务的 `deployment.yaml` 文件中。在以下[基本 Kubernetes 示例](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中，各环境中的值一致时，直接指定非常有效：

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

### Helm 变量
{: #config-helm}

如前所述，Helm 使用模板来创建 chart，以便可以稍后替换值。通过在 `mychart/templates/pod.yaml` 文件模板中使用以下示例，可以实现与前一个示例中相同的结果，但此示例对于在不同环境中使用具有更大灵活性：

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

`mychart/values.yaml` 文件中的以下示例：

```yaml
greeting: "Hello from the environment"
farewell: "Such a sweet sorrow"
```
{: codeblock}

Helm 呈现该模板时，会生成以下输出：

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

  这两个示例之间有细微差别。在第一个示例和样本 `values.yaml` 文件中，引号是人工添加的。YAML 中的字符串不需要引号。当 Helm 呈现模板时，将省去引号。
  {: note}

### ConfigMap
{: #kubernetes-configmap}

ConfigMap 是一个独特的 Kubernetes 工件，用于将数据定义为一组键/值对。上面示例中所示环境变量的 ConfigMap 可能类似于以下示例：

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

然后，初始 Pod 定义将更改为使用 ConfigMap 中的值，如下所示：

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

现在，ConfigMap 是 Pod 中的一个完全独立的工件。它可以具有完全不同的生命周期。您可以更新或更改 ConfigMap 中的值，而不必因此重新部署 Pod。您还可以直接在命令行中更新和处理 ConfigMap，这在开发/测试/调试周期中会很有用。

与 Helm 一起使用时，可以在 ConfigMap 声明中使用变量。这些变量会像部署 chart 时一样正常进行解析。

有关更多信息，请参阅 [Kubernetes ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

### 凭证和私钥
{: #kubernetes-secrets}

配置通常通过环境变量或 ConfigMap 提供给在 Kubernetes 中运行的容器。在任一情况下，都可以相当快地发现配置值。这就是为什么 Kubernetes 使用 Secret 来存储敏感信息的原因。

Secret 是包含 Base64 编码值的独立对象：

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

然后，可以将 Secret 用作安装在一个或多个 Pod 容器上的卷中的文件：

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

以这种方式安装为卷时，Secret 的数据映射中的每个密钥都会成为指定的 `mountPath` 下的一个文件名：在本例中为 `/etc/foo/username` 和 `/etc/foo/password`。

Secret 还可用于定义环境变量：

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

Kubernetes 会为您执行 Base64 解码。在 Pod 中运行的容器在检索到该环境变量时会识别 Base64 解码值。

与 ConfigMap 一样，Secret 可以在命令行中进行创建和处理，这在处理 SSL 证书时非常有用。

有关更多信息，请参阅 [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")。

<!-- SSL EXAMPLE -->
