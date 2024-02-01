---
title: Concepts & Configuration
description: Configure the Nuxeo Java Client
review:
    comment: ''
    date: '2019-06-18'
    status: ok
labels:
    - java-client
tree_item_index: 200
section_parent: java
toc: true

---

## Concepts

Nuxeo Java Client uses several libraries:
- [Retrofit](https://square.github.io/retrofit/) and [OkHttp](https://square.github.io/okhttp/) to perform HTTP requests.
- [Jackson](https://github.com/FasterXML/jackson) to serialize/deserialize requests/responses
- [Guava](https://github.com/google/guava) to cache responses (optional)

The basic flow when you perform a request to Nuxeo Server from Nuxeo Java Client looks like this:
{{!--     ### nx_asset ###
    path: /default-domain/workspaces/Product Management/Documentation/Documentation Screenshots/Clients/Java/Concepts and Configuration/Java Client Request flow
    name: Nuxeo Java Client - Basic Flow.png
    addins#diagram#up_to_date
--}}
![Java Client Request flow](/nx_assets/13f368cc-2faf-43b7-b011-c619b9594977.png ?border=true)

To ease usage and configuration of client we added several notions/objects.

### Builder

Builder is the entry point of Nuxeo Java Client. It is used to create the client, validate the needed configuration and test connection to Nuxeo Server. After calling `Builder#connect()`, you'll get a working `NuxeoClient` owning the current user used for further requests.

### NuxeoClient

NuxeoClient is the base client's class. It allows to get managers and to perform arbitrary HTTP request with client configuration. It also owns the cache, the converter factory, the server version and the current user performing requests.

Managers are created each time you get them, we're gonna see why in [configuration isolation](#configuration-isolation) section.

Cache is given to NuxeoClient during creation with `Builder#cache(NuxeoResponseCache)`.

Converter factory is created during `Builder` instantiation.

The server version is fetched from Nuxeo Server in a lazy way. Client will get this information by requesting `$URL/json/cmis`.

Current user is retrieved during `Builder#connect()`.

### Managers

Managers own a set of Nuxeo Server API, for example `UserManager` manager owns API to handle user and group. Every manager offers a sync and an async version for each API, the async version leverages `Callback` Retrofit class.

Under the hood, they hold the bridge between Nuxeo Java Client API and Retrofit API.

### Objects

Every sent/received object in Nuxeo Java Client extends one of these types:
- `Entity` for objects without connectivity needs
- `ConnectableEntity` for objects with connectivity needs

They both offer a `getEntityType` method and handle `entity-type` read/write in JSON (used for serialization/deserialization needs).

There're other types for object consume/produce by the client:
- `Connectable` interface for objects with connectivity needs or holding sub objects with connectivity needs
- `Entities` class for objects holding a list of entities
- `PaginableEntity` class for objects with pagination capabilities

**Let's see each type in detail:**

- `Entity` is a basic java class with entity type capabilities.

- `ConnectableEntity` has the entity type capabilities, implement `Connectable` interface and provide access to client and Retrofit API objects (in order to execute requests). A connectable entity is able to execute requests if object is handle by client. For example, `Document` is a connectable entity, which gives convenient methods:
```
Document root = nuxeoClient.repository().fetchDocumentRoot();
// here we leverage ConnectableEntity's capabilities
Documents children = root.fetchChildren();
```

- `Connectable` interface is mainly used to handle connectable entity. Every class implementing this interface and returned by the client will receive a call to `reconnectWith(NuxeoClient)` after post treatments. This allows to give client to connectable entity, to reconnect children in collection objects (such as Documents, Users, ...).

- `Entities` is an entity with some useful API to handle container such as directories, tasks, workflows, comments...

- `PaginableEntity` is an entity with extra fields for pagination information (such as totalSize, pageIndex, pageCount, isNextPageAvailable, ...). This is mainly used for documents on query API for instance.

### Constants

Nuxeo Java Client uses some constants internally, we make some of them free to use:
- `ConstantsV1` owning default values for Nuxeo REST API v1
- `HttpHeaders` owning HTTP headers for Nuxeo REST API usage
- `MediaTypes` owning common media types
- `Operations` owning common Nuxeo operation ids

### Marshalling

All serialization/deserialization are made by `NuxeoRequestConverter`/`NuxeoResponseConverter` leveraging Jackson. They are managed by Retrofit and created for each request/response.

Serialization is pretty simple and use Jackson marshalling capabilities.

Deserialization first checks if the return type is given by Retrofit, if so we unmarshall response with this type. If not, we read `entity-type` field in the response, get type from registered types and finally unmarshall response. You can register your own entity types with `Builder#registerEntity(String, Class<?>)`.

### Cache

Nuxeo Java Client offers a built-in cache mechanism relying on `NuxeoResponseCache` interface. We provide one implementation based on Guava in `nuxeo-java-client-cache` artifact.

Only GET requests are cached.

You can clean the cache with `NuxeoClient#refreshCache`.

## Configuration

Depending on configuration type, you can configure your clients at different moment:
- during creation with `Builder`
- during use with `NuxeoClient`
- during use with managers - access specific manager configuration

The three kinds of configuration holders all have the common part of configuration.

Builder owns connection, authentication, cache, serialization and common configuration because they are not supposed to change during client life.

NuxeoClient owns only common configuration.

Managers own common and specific configuration, for instance, `Operation` owns the `voidOperation` configuration besides common ones.

{{> anchor 'configuration-isolation'}}
### Configuration Isolation
The manager configuration is isolated from the client one.

This means that each configuration made on a manager belongs to this manager instance.
For instance:
```java
Repository repository1 = client.repository().schemas("*");
Repository repository2 = client.repository().schemas("dc");

// document containing all schemas
Document doc1 = repository1.fetchDocumentRoot();
// document containing only dublincore
Document doc2 = repository2.fetchDocumentRoot();
```

### Common Configuration

| Method                                                 | Description                                                                                 |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| timeout(long)                                          | Configure connect, read and write timeout, unit is seconds                                  |
| connectTimeout(long)                                   | Configure connect timeout, unit is seconds                                                  |
| readTimeout(long)                                      | Configure read timeout, unit is seconds                                                     |
| writeTimeout(long)                                     | Configure write timeout, unit is seconds                                                    |
| retryOnConnectionFailure(boolean)                      | Configure to retry or not when a connectivity problem is encountered                        |
| header(String, boolean)                                | Replace header of a given name with given value                                             |
| header(String, int, int...)                            | Replace header of a given name with given values                                            |
| header(boolean, String, int, int...)                   | Replace or append header of a given name with given values                                  |
| header(String, Serializable, Serializable...)          | Replace header of a given name with given values                                            |
| header(boolean, String, Serializable, Serializable...) | Replace or append header of a given name with given values                                  |
| header(String, String, String...)                      | Replace header of a given name with given values                                            |
| header(boolean, String, String, String...)             | Replace or append header of a given name with given values                                  |
| transactionTimeout(long)                               | Configure transaction timeout on Nuxeo Server                                               |
| enrichers(boolean, String, String, String...)          | Replace or append enricher of given type with given values                                  |
| enrichers(String, String, String...)                   | Replace enricher of given type with given values                                            |
| enrichersForDocument(String, String...)                | Replace enricher for document with given values                                             |
| fetchProperties(boolean, String, String, String...)    | Replace or append fetch properties of given type with given values                          |
| fetchProperties(String, String, String...)             | Replace fetch properties of given type with given values                                    |
| fetchPropertiesForDocument(String, String...)          | Replace fetch properties for document with given values                                     |
| fetchPropertiesForGroup(String, String...)             | Replace fetch properties for group with given values                                        |
| depth(String)                                          | Replace the depth header with given value, possible values are `root`, `children` and `max` |
| version(String)                                        | Replace the set versioning option header with given value                                   |
| schemas(boolean, String, String...)                    | Replace or append schemas to fetch with given values, `*` is allowed to fetch them all      |
| schemas(String, String...)                             | Replace schemas to fetch with given values, `*` is allowed to fetch them all                |

### Builder Configuration

| Method                           | Description                                             |
| -------------------------------- | ------------------------------------------------------- |
| authentication(String, String)   | Configure basic authentication with user/password       |
| authentication(Interceptor)      | Configure another kind of authentication                |
| cache(NuxeoResponseCache)        | Configure a cache for client                            |
| interceptor(Interceptor)         | Add a new OkHttp interceptor                            |
| registerEntity(String, Class<?>) | Register a new entity type in client marshalling system |
| url(String)                      | The Nuxeo Server URL                                    |

You must configure authentication and URL at least.

### Managers Configuration

#### Operation

| Method          | Description                                                        |
| --------------- | ------------------------------------------------------------------ |
| voidOperation() | Set void operation header for next operation calls on this manager |

#### Examples

Basic example on client:
```java
NuxeoClient client = client.schemas("*")
                           .connectTimeout(30)
                           .readTimeout(60)
                           .header(HttpHeaders.NX_ES_SYNC, true)
                           // same effect than transactionTimeout(long)
                           .header(HttpHeaders.NUXEO_TX_TIMEOUT, 300)
                           .header("source", "FROM_JAVA_CLIENT");
```

Example on repository manager, configuration will affect only this repository instance:
```java
Repository repository = client.repository()
                              .enrichersForDocument("thumbnail", "breadcrumb")
                              .fetchPropertiesForDocument("dc:creator");
Document doc = repository.fetchDocumentByPath("/default-domain");
User creator = doc.getPropertyValue("dc:creator"); // we get an User object because of fetchProperties
Documents breadcrumb = doc.getContextParameter("breadcrumb");
```

&nbsp;

<div class="row" data-equalizer data-equalize-on="medium">
<div class="column medium-6">
{{#> panel heading='Related sections in this documentation'}}

- [Nuxeo - Special HTTP Headers]({{page space='nxdoc' page='special-http-headers'}})

{{/panel}}
</div>
</div>

&nbsp;
