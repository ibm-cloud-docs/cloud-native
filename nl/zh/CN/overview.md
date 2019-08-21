---

copyright:
  years: 2019
lastupdated: "2019-07-16"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# 什么是云本机？
{: #overview}

云计算环境是动态的，支持根据需要对虚拟化共享池中的资源进行分配和释放。与传统内部部署数据中心内通常使用的提前资源分配相比，这种弹性环境支持更灵活的缩放选项。
{:shortdesc}

根据 [Cloud Native Computing Foundation ](https://github.com/cncf/foundation/blob/master/charter.md){: new_window}![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 的说法，云本机系统具有以下特性：

- 应用程序或进程在软件容器中作为隔离单元运行。
- 进程由中央编排进程进行管理，以提高资源使用率，降低维护成本。
- 应用程序或服务（微服务）通过显式描述的依赖关系松散耦合。

这些特性描述了由多个独立进程组成的高度动态的系统，这些进程一起运行以提供业务价值，这是一种分布式系统。

分布式计算这一概念可追溯到几十年前。[Fallacies of Distributed Computing](http://www.rgoarchitects.com/Files/fallacies.pdf){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 中描述了分布式系统架构设计师和设计人员所做的以下假设，最终这些假设被证明是错误的。 

* 网络是可靠的。
* 网络是安全的。
* 网络是同种类型的。
* 等待时间为零。
* 带宽无限。
* 拓扑不会更改。
* 存在一个管理员。
* 传输成本为零。

Kubernetes 和 Istio 等云技术旨在解决基础架构本身的这些问题。

## 十二要素
{: #twelve-factors}

[十二要素应用程序](https://12factor.net){: new_window} ![外部链接图标](../icons/launch-glyph.svg "外部链接图标") 方法是 Heroku 的开发者起草的。十二要素中提到的特征并不特定于云提供者、平台或语言。这些要素代表了针对云环境中蓬勃发展的可移植弹性应用程序（特别是“软件即服务”应用程序）的一组准则或最佳实践。下面列出了这十二要素：

1. 已版本化的代码库（例如 Git 存储库）和已部署的服务之间存在一对一关联。同一代码库用于多个部署。
2. 服务显式声明所有依赖关系，并且不依赖于系统级别的工具或库的存在。
3. 在不同部署环境之间变化的配置存储在环境中，特别是存储在环境变量中。
4. 所有后备服务均被视为连接的资源，由执行环境进行管理（连接和拆离）。
5. 交付管道具有严格分离的阶段：构建、发布和运行。
6. 应用程序部署为一个或多个无状态进程。具体来说，是部署为瞬态进程，这种进程无状态且不共享任何内容。持久存储的数据存储在相应的后备服务中。
7. 自包含服务通过侦听指定的端口，使自身可供其他服务使用。
8. 通过缩放各个进程（水平缩放）来实现并行性。
9. 进程是一次性的：快速启动和正常关闭行为使得系统更加稳健，更富有弹性。
10. 从本地开发一直到生产在内的所有环境尽可能相似。
11. 应用程序将日志生成为事件流，例如写入 `stdout` 和 `stderr`，并信任执行环境来聚集流。
12. 如果需要一次性管理任务，那么这些任务会保留在源代码控制中，并与应用程序一起打包，以确保其与应用程序在相同的环境中运行。

要实现高质量的微服务环境，您无需严格遵循这些要素。但是，遵循这些要素，您可以在持续交付环境中构建和维护可移植应用程序或服务。

## 应用程序
{: #apps-intro}

应用程序由代码、数据、服务和工具链构成。例如，{{site.data.keyword.cloud_notm}} 移动应用程序包含设备代码，以及后端逻辑、数据存储器、分析和安全服务，并设置为支持持续交付。

![复用](images/garage_reuse2.png "通过 Developer Experience，您可以复用代码而避免重复劳动")

您可以使用任何 {{site.data.keyword.cloud_notm}} 开发者门户网站或 {{site.data.keyword.dev_cli_notm}} 来创建和管理应用程序。

您可以直接创建简单的空白应用程序，也可以使用入门模板工具包来创建更复杂的应用程序。如果您选择不借助入门模板工具包来创建空白应用程序，那么可以在 [{{site.data.keyword.cloud_notm}} 仪表板 ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](https://{DomainName}){: new_window} 中执行此操作，而不访问门户网站。

您可以使用代码模式来快速创建应用程序并将其部署到 {{site.data.keyword.cloud_notm}}。从 [IBM Developer Web 站点 ![外部链接图标](../icons/launch-glyph.svg "外部链接图标")](https://developer.ibm.com/patterns/){:new_window} 中，选择一个代码模式。您可以在 GitHub 中查看代码，也可以在 {{site.data.keyword.cloud_notm}} 中创建并构建应用程序，在其中您可以使用 DevOps 工具链来自动部署应用程序。


## 微服务
{: #microservices}

*微服务*是一组小型独立体系结构组件，它们通过公共轻量级 API 进行通信。以下简单示例中的每个微服务都是一个十二要素应用程序，都使用可替换的后备服务来存储数据和传递消息：

![微服务应用程序](images/microservice.png "微服务应用程序")

微服务是独立的。敏捷性是微服务体系结构的其中一个优点，但仅当服务能够被重写而不妨碍其他服务时，才能体现出此优点。 

通过明确的 API 边界，基于服务工作的团队能以最高灵活性逐渐发展实现。正是此特征支持多语言编程和持久存储。

微型服务具有弹性。应用程序稳定性取决于各个微服务抗故障的稳健性。这与传统体系结构有很大不同，传统体系结构通过支持基础架构来处理故障。每个服务都需要应用隔离模式（例如，断路器和舱壁），才能控制故障并定义相应的回退行为，从而保护上游服务。

微服务是无状态的，存储在外部后备云服务（如 Redis）中。快速启动和正常关闭行为可进一步支持微服务维护系统运行状况，或在根据负载来创建和除去实例的自动化环境中高效工作。

### “小型”的含义
{: #small-microsvc}

“小型”这个词用于形容微服务时，表示微服务专注于其用途。许多描述会将各个微服务的角色与 Unix 命令行上的链接命令进行类比：

```
ls | grep 'service' | sort -r
```
{:pre}

这些 UNIX 命令中，每个命令执行明显不同的任务，可以将这些命令链接在一起，而不考虑编程语言或代码数量。

## 多语言应用程序：选择适用于作业的工具
{: #polyglot-apps}

多语言是基于微服务的体系结构中经常提到的一个优点。通过创建可供选择的受支持技术的列表，并利用定义的策略随时间变化不断用新技术扩展此列表，可达到平衡。确保可以满足可维护性、可审计性和数据安全性等非功能性或法规需求，同时保留敏捷性并支持通过试验进行创新。

... :FIXME：解包？在此使用语言表？:...

## REST 和 JSON
{: #rest-json}

多语言应用程序只能使用与语言无关的协议。REST 体系结构模式定义了用于创建统一接口的准则，使线上数据表示法独立于服务的实现。

在微服务体系结构中，JSON 是基于文本的数据的首选有线格式，以相对更简单、更简洁的形式取代了 XML。作为比较，以下示例是包含员工相关数据的 JSON 格式基本记录：

```json
{
  "name": "Marley Cassin",
  "serial": 228264,
  "title": "Senior Software Engineer",
  "address": {
    "office": "501-B101",
    "street": "3858 Kuvalis Pass",
    "city": "East Craig",
    "state": "NC",
    "zip": "64519-8934"
  }
}
```
{: codeblock}

以下示例是 XML 格式的同一员工记录：

```xml
<person>
  <name>Marley Cassin</name>
  <serial>228264</serial>
  <title>Senior Software Engineer</title>
  <address>
    <office>501-B101</office>
    <street>3858 Kuvalis Pass</street>
    <city>East Craig</city>
    <state>NC</state>
    <zip>64519-8934</zip>
  </address>
</person>
```
{: codeblock}

JSON 使用属性值对以简洁的语法来表示数据对象，该语法保留了一些有关基本类型（例如，数字、字符串、数组和对象）的信息。在以上示例中，JSON 和 XML 都明确表示了嵌套地址对象，但您需要关联的 XML 模式来确定 `serial` 元素的类型。对于 JSON，语法明确表明了 `serial` 的值是数字，而不是字符串。

