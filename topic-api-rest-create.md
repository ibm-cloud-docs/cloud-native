---

copyright:
  years: 2019, 2020
lastupdated: "2020-10-02"

---

{:external: target="_blank" .external}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}

# Creating RESTful microservices
{: #rest-api}

Cloud-native applications produce and use APIs, whether in a microservices architecture or not. Some APIs are considered internal, or private, and some are considered external. 
{:shortdesc}

Internal APIs are used only within a firewalled environment for backend services to communicate with each other. External APIs present a unified entry point for consumers, and are often **managed** by tools like {{site.data.keyword.apiconnect_long}}, which can impose rate limits or other usage constraints. An example of this kind of API is the [GitHub Developer API](https://developer.github.com/v3/){: external}. It provides a unified API with consistent use of HTTP verbs, return codes, and pagination behavior without showing internal implementation details. This API can be backed by one large application, or it can be backed by a collection of microservices. That detail isn't shown to the consumer, so GitHub can evolve their internal systems as necessary.

## Best practices for RESTful APIs
{: #bps-apis}

REST APIs use standard HTTP verbs for Create, Retrieve, Update, and Delete (CRUD) operations, with special attention that is paid to whether the operation is idempotent (safe to retry multiple times).

* POST operations can be used to create resources. POST operations can't be invoked repeatedly. For example, if a POST request is used to create resources, and it is invoked multiple times, a new, unique resource is created as a result of each invocation.
* GET operations must be able to be invoked repeatedly and must not cause side effects. They're to be used to retrieve information. GET requests with query parameters are not to be used to change or update information. Use the POST, PUT, or PATCH operations instead.
* PUT operations can be used to update resources. PUT operations usually include a complete copy of the resource to be updated, making it possible to invoke the operation multiple times.
* PATCH operations allow partial update of resources. They can be invoked repeatedly depending on how the delta is specified and then applied to the resource. For example, if a PATCH operation indicates to change a value from A to B, it can be invoked repeatedly. The operation has no effect if it is invoked multiple times and the value is already B.
* DELETE operations cannot be invoked multiple times, as a resource can be deleted only once. However, the return code varies, as the first operation succeeds (`200` or `204`), while subsequent invocations do not find the resource (`404` or `410`).

### Machine-friendly, descriptive results
{: #rest-results}

Given that APIs are invoked by software instead of by humans, take care to communicate information to the caller in the most effective and efficient way possible.

Use relevant and useful HTTP status codes, as described in the following table: 

| HTTP Error Code | Usage Guidance |
|-----------------|----------------|
| `200 (OK)` | Use when everything is fine and there is data to return |
| `204 (NO CONTENT)` | Use when everything is fine but there is no response data |
| `201 (CREATED)` | Use for POST requests that result in the creation of a resource, whether there is a response body or not |
| `409 (CONFLICT)` | Use when concurrent changes conflict |
| `400 (BAD REQUEST)` | Use when parameters are malformed |

For more information, see [Response status codes](https://tools.ietf.org/html/rfc7231#section-6){: external}. 

Consider what data to return in your responses to make communication efficient. For example, when a resource is created with a POST request, the response must include the location of the newly created resource in a Location header. The created resource is often included in the response body as well to eliminate the extra GET request to fetch the created resource. The same applies for PUT and PATCH requests.

### RESTful resource URIs
{: #rest-uris}

There are varying opinions about some aspects of RESTful resource URIs. In general, it is agreed that resources must be nouns, not verbs, and endpoints must be plural. This model results in a clear structure for CRUD operations:

* `POST /accounts`: Create a new account.
* `GET /accounts`: Retrieve a list of accounts.
* `GET /accounts/16`: Retrieve a specific account.
* `PUT /accounts/16`: Update a specific account.
* `PATCH /accounts/16`: Update a specific account.
* `DELETE /accounts/16`: Delete a specific account.

Relationships are modeled by using hierarchical URIs, for example, ` /accounts/16/credentials` for managing credentials associated with an account.

There is no single way to manage operations with a resource that doesn't fit within a typical structure. For these operations, do what works best for the consumer of the API.

### Robustness and RESTful APIs
{: #robust-api}

The [Robustness Principle](https://tools.ietf.org/html/rfc1122#page-12){: external} provides the best guidance: "Be liberal in what you accept, and conservative in what you send". Assume that APIs will evolve over time and be tolerant of data you do not understand.

#### Producing APIs
{: #robust-producer}

When you provide an API to external clients, there are two things you must do when you accept requests and return responses: 

* Accept unknown attributes as part of the request.
    > If a service calls your API with unnecessary attributes, throw those values away. Returning an error in this scenario can cause unnecessary failures, negatively impacting the user.
* Return the attributes required by your consumers
    > Avoid exposing internal service details. Expose attributes that consumers need as part of the API.

#### Consuming APIs
{: #robust-consumer}

When consuming APIs:

* Validate the request against the variables or attributes that you need.
    > Do not validate against variables just because they are provided. If you are not using them as part of your request, do not rely on them being there.

* Accept unknown attributes as part of the response.
    > Do not issue an exception if you receive an unexpected variable. If the response contains the information you need, it doesn't matter what else comes along for the ride.

These guidelines are especially relevant for strongly typed languages like Java, where JSON serialization and deserialization often occur indirectly. For example, the Jackson libraries, or JSON-P/JSON-B. Look for language mechanisms that let you specify more generous behavior like ignoring unknown attributes, or to define or filter which attributes must be serialized.

### Versioning RESTful APIs
{: #version-api}

One of the major benefits of microservices is the ability to allow services to evolve independently. Given that microservices call other services, that independence comes with a giant caveat: you can't cause breaking changes in your API.

If the robustness principle is followed, it can take a long while before a breaking change is required. When that breaking change happens, you can opt to build a different service entirely and retire the original over time.

If you do need to make breaking API changes for an existing service, decide how to manage those changes: Does the service handle all versions of the API, do you maintain independent versions of the service to support each version of the API, or does your service support only the newest version of the API and rely on other adaptive layers to convert to and from the older API?

After you determine how to manage the changes, the much easier problem to solve is how to reflect the version in your API. There are generally three ways to version a REST resource:

* Include the version in the URI path.
* Include the version in the HTTP Accept header and rely on content negotiation.
* Use a custom request header.

#### Including the version in the URI path
{: #version-path}

The easiest way to specify a version is to include it in the path of the URI. This approach has advantages: it's obvious, it's easy to achieve when you build the services in your application, and it's compatible with API-browsing tools like Swagger, command-line tools like `curl`, and so on.

If you're going to include the version in the path of the URI, the version should apply to your application as a whole, for example, `/api/v1/accounts` instead of `/api/accounts/v1`. Hypermedia as the Engine of Application State (HATEOAS) is one way of providing URIs to API consumers so they are not responsible for constructing URIs themselves. GitHub, for example, provides [hypermedia URLs](https://developer.github.com/v3/#hypermedia){: external} in the responses for this reason. HATEOAS becomes difficult, if not impossible, to achieve if different backend services can have independently varying versions in their URIs.

#### Modifying the Accept header to include the version
{: #version-accept}

The Accept header is an obvious place to define a version, but is one of the most difficult to test. The Accept header is also a frequent landing place for feature toggles. Specifying HTTP headers requires more detailed API invocations.

#### Adding a custom request header
{: #version-custom}

You can add a custom request header to indicate the API version. As with the Accept header, you can also use custom headers to route traffic to specific backend instances. With this method, you encounter the same ease-of-use issues that you encounter with the Accept header method, with the additional requirement that consumers need to learn about this header.

For more information, see [Your API versioning is wrong, which is why I decided to do it 3 different wrong ways](https://www.troyhunt.com/your-api-versioning-is-wrong-which-is/){: external}.

## Creating and generating APIs
{: #create-api}

[OpenAPI v3](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.2.md){: external} is the official specification for RESTful services and is governed by the [OpenAPI Initiative](https://www.openapis.org/){: external}, an association of companies under the Linux Foundation.

You can use either of the following ways to create an API:

  * Start with an OpenAPI Definition (top-down): In this approach, you begin by creating an OpenAPI definition in a language-independent format (usually YAML). You then use a code generator to create a skeleton, and build your service implementation from there. This pattern is usually adopted by companies that have a central API design team, and allows for development and test to progress in parallel.
  * Start with code (bottom-up): Your code is the source of your API definition. This approach works well for new applications with an experimental aspect to them, as your API definition evolves as you gain a better understanding of what your service needs to do. This approach also works better in some languages than others, as it relies on tools that generate an OpenAPI definition from your code. Java, for example, has excellent support for generating OpenAPI documents from annotation-based REST frameworks.

In either case, working with an OpenAPI definition can help identify areas where the API is inconsistent or difficult to understand from a consumer point of view. Published or version-controlled OpenAPI definitions can also be used by build tools to help flag breaking changes that would impact consumers.

### Creating an API from an OpenAPI Definition
{: #openapi-first}

You can author your OpenAPI YAML file in whatever tool you choose. However, using a plain text editor can be error prone. Some editors have basic support for YAML, and some might have more extensions to support OpenAPI definitions. For example, you can use Visual Studio Code extensions like [Swagger Viewer](https://marketplace.visualstudio.com/items?itemName=Arjun.swagger-viewer){: external} or [OpenAPI Preview](https://marketplace.visualstudio.com/items?itemName=zoellner.openapi-preview){: external} to validate your OpenAPI definition against a specified spec version and render a web view in the preview pane:

![OpenAPI Preview](images/create-api-image1.png "OpenAPI Preview"){: caption="Figure 1. OpenAPI Preview" caption-side="bottom"} 

You can use many browser-based, live-parsing editors either online or locally. Some examples include:

* The [OpenAPI-GUI project](https://github.com/Mermade/openapi-gui){: external} supports both v2 and v3 of the OpenAPI specification and can migrate an OpenAPI v2 definition to v3 for you.
* [Swagger Editor from SmartBear](https://editor.swagger.io){: external} also supports both v2 and v3 of OpenAPI.
* [{{site.data.keyword.apiconnect_short}}](https://cloud.ibm.com/catalog/services/api-connect){: external} provides a set of editors and tools for API modeling and creation.

### Generating the API implementation
{: #code-first}

You can use the open source [OpenAPI generator](https://github.com/OpenAPITools/openapi-generator){: external} to create a skeleton project for your service implementation from an OpenAPI definition. You can specify the language or framework for the skeleton from the command line. For example, to create a Java project for the sample PetStore API that uses generic JAX-RS method annotations, you specify the following command:

```bash
openapi-generator generate -g jaxrs-cxf-cdi -i ./petstore.yaml -o petstore --api-package=com.ibm.petstore
```
{: pre}

