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

# What is cloud native?
{: #overview}

Cloud computing environments are dynamic, with on-demand allocation and release of resources from a virtualized, shared pool. These elastic environments enable more flexible scaling options when compared to the up-front resource allocation that is typically used in traditional on-premises data centers.
{:shortdesc}

According to the [Cloud Native Computing Foundation](https://cncf.io/about/charter){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon"), cloud-native systems have the following attributes:

- Applications or processes are run in software containers as isolated units.
- Processes are managed by central orchestration processes to improve resource utilization and reduce maintenance costs.
- Applications or services (microservices) are loosely coupled with explicitly described dependencies.

These attributes describe a highly dynamic system that is composed of independent processes working together to provide business value: a distributed system.

Distributed computing is a concept with roots stretching back decades. [Fallacies of Distributed Computing](http://www.rgoarchitects.com/Files/fallacies.pdf){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") captures the following assumptions made by architects and designers of distributed systems that prove wrong in the long run. 

* The network is reliable.
* The network is secure.
* The network is homogeneous.
* Latency is zero.
* Bandwidth is infinite.
* Topology doesn't change.
* There is one administrator.
* Transport cost is zero.

Cloud technologies like Kubernetes and Istio aim to address these concerns in the infrastructure itself.

## Twelve factors
{: #twelve-factors}

The [twelve-factor application](http://12factor.net){: new_window} ![External link icon](../icons/launch-glyph.svg "External link icon") methodology was drafted by developers at Heroku. The characteristics mentioned in the twelve factors are not specific to a cloud provider, platform, or language. The factors represent a set of guidelines or best practices for portable, resilient applications that thrive in cloud environments (specifically Software as a Service applications). The twelve factors are provided in the following list:

1. There is a one-to-one association between a versioned codebase, for example, a git repository, and a deployed service. The same codebase is used for many deployments.
2. Services explicitly declare all dependencies, and do not rely on the presence of system-level tools or libraries.
3. Configuration that varies between deployment environments is stored in the environment, specifically in environment variables.
4. All backing services are treated as attached resources, which are managed (attached and detached) by the execution environment.
5. The delivery pipeline has strictly separate stages: build, release, run.
6. Applications are deployed as one or more stateless processes. Specifically, transient processes are stateless and share nothing. Persisted data is stored in an appropriate backing service.
7. Self-contained services make themselves available to other services by listening on a specified port.
8. Concurrency is achieved by scaling individual processes (horizontal scaling).
9. Processes are disposable: fast startup and graceful shutdown behaviors lead to a more robust and resilient system.
10. All environments, from local development to production, are as similar as possible.
11. Applications produce logs as event streams, for example, writing to `stdout` and `stderr`, and trust the execution environment to aggregate streams.
12. If one-off admin tasks are needed, they are kept in source control and packaged alongside the application to ensure they are ran with the same environment as the application.

You don't have to strictly follow these factors to achieve a quality microservice environment; however, keeping them in mind enables you to build and maintain portable applications or services in continuous delivery environments.

## Microservices
{: #microservices}

A *microservice* is a set of small, independent architectural components, each with a single purpose, that communicate over a common lightweight API. Each microservice in the following simple example is a twelve factor application that uses replaceable backing services to store data and pass messages.

![A microservices application](images/microservice.png "A microservices application") A microservices application

Microservices are independent. Agility is one of the benefits of microservice architectures, but it only exists when services are capable of being completely re-written without disturbing other services. That isn't likely to happen often, but it explains the requirement. Clear API boundaries give the team working on a service the most flexibility to evolve the implementation. This characteristic is what enables polyglot programming and persistence.

Microservices are resilient. Application stability depends on individual microservices being robust to failure. This is a big difference from traditional architectures, where the supporting infrastructure handles failures for you. Each service needs to apply isolation patterns, such as circuit breakers and bulkheads, to contain failures and define appropriate fallback behaviors to protect upstream services.

Microservices are stateless, transient processes. This is not the same as saying microservices can't have state! It means state should be stored in external backing cloud services, like Redis, rather than in-memory. Fast startup and graceful shutdown behavior further allow microservices to work well in automated environments that create and destroy instances in response to load or to maintain system health.

### The meaning of "small"
{: #small-microsvc}

The use of the word "small", as applied to a microservice, essentially means that it is focused in purpose: it should do one thing and do that one thing well. Many descriptions make parallels between the roles of individual microservices and chained commands on the Unix command line:

```
ls | grep 'service' | sort -r
```
{:pre}

These Unix commands each perform distinctly different tasks, and you can chain them together regardless of programming language or quantity of code.

## Polyglot applications: choosing the right tool for the job
{: #polyglot-apps}

Polyglot is a frequently cited benefit of microservice-based architectures. On one hand, the ability to choose the appropriate language or data store for the function a service is providing can be very powerful, and can bring a lot of efficiency. On the other hand, the use of obscure technologies can complicate long-term maintenance and inhibit the movement of developers between teams. 

Create a balance between the two by creating a list of supported technologies to choose from at the outset, with a defined policy for extending the list with new technologies over time. Ensure that non-functional or regulatory requirements like maintainability, auditability, and data security can be satisfied, while preserving agility and supporting innovation through experimentation.

## REST and JSON
{: #rest-json}

Polyglot applications are only possible with language-agnostic protocols. REST architecture patterns define guidelines for creating uniform interfaces that separate the on-the-wire data representation from the implementation of the service.

JSON has emerged in microservices architectures as the wire format of choice for text-based data, displacing XML with its comparative simplicity and conciseness. As a comparison, the following example is a basic record that contains data about an employee in JSON:

  ```json
{
  "name": "Marley Cassin",
  "birthDate": "1971-06-14",
  "gender": "Male",
  "address": {
    "office": "501-B101",
    "street": "3858 Kuvalis Pass",
    "city": "East Craig",
    "state": "NC",
    "zip": 64519
  }
}
```
{: codeblock}

And the following example is the same employee record in XML:

```xml
<person>
  <name>Marley Cassin</name>
  <birthDate>1971-06-14</birthdate>
  <gender>Male</gender>
  <address>
    <office>501-B101</office>
    <street>3858 Kuvalis Pass</street>
    <city>East Craig</city>
    <state>NC</state>
    <zip>64519</zip>
  </address>
</person>
```
{: codeblock}

JSON uses attribute-value pairs to represent data objects in a concise syntax that preserves information about a few basic types, such as numbers, strings, arrays, and objects. Both JSON and XML clearly represent the nested address object in the previous examples, but you need the associated XML schema to understand what the type is for the zip element. In JSON, the syntax makes it clear that the value of zip is a number and not a string.
