---
title: Java Client SDK
description: Nuxeo Client SDK - Java
review:
    comment: ''
    date: '2018-01-02'
    status: ok
labels:
    - java-client
toc: true
is_overview: true

---

The Nuxeo Java Client is a  Java client library for Nuxeo REST API.

It is compatible with all Nuxeo versions as of LTS 2015.

## Getting started

Follow this [tutorial]({{page space='nxdoc' page='setting-up-your-nuxeo-environment'}}) to have a running Nuxeo instance on your local machine.

### Installation

To use nuxeo-java-client, you can download it from our Nexus: [Nuxeo Client Library 3.1.1-SNAPSHOT](https://maven.nuxeo.org/nexus/#nexus-search;gav%7Eorg.nuxeo.client%7Enuxeo-java-client%7E3.1.1-SNAPSHOT%7Ejar%7E).

If you use Maven, you need to have nuxeo-java-client as dependency:
```xml
<dependency>
  <groupId>org.nuxeo.client</groupId>
  <artifactId>nuxeo-java-client</artifactId>
  <version>3.1.1-SNAPSHOT</version>
</dependency>

<repository>
  <id>public-releases</id>
  <url>http://maven.nuxeo.com/nexus/content/repositories/public-releases/</url>
</repository>
<repository>
  <id>public-snapshots</id>
  <url>http://maven.nuxeo.com/nexus/content/repositories/public-snapshots/</url>
</repository>
```

### Client creation

Nuxeo Java Client is created with help of its `Builder`. Once every options are submitted, last step is to build the client and test the connection to Nuxeo Server:
```java
NuxeoClient client = new NuxeoClient.Builder()
                                    .url("http://localhost:8080/nuxeo")
                                    .authentication("Administrator", "Administrator")
                                    .schemas("*") // fetch all document schemas
                                    .connect();
```

Client is now ready to execute requests to Nuxeo Server. After its creation, it will contain the current Nuxeo user and the Nuxeo Server version.

More documentation about client [options]({{page page=''}}).

### Fetch your first document

Below is how we can retrieve default domain from a stock Nuxeo Server:
```java
Document domain = client.repository().fetchDocumentByPath("/default-domain");
String title = domain.getPropertyValue("dc:title"); // should be equal to "Domain"
```

&nbsp;

<div class="row" data-equalizer data-equalize-on="medium"><div class="column medium-6">{{#> panel heading='Related sections in this documentation'}}

- [Nuxeo - Getting Started]({{page space='nxdoc' page='getting-started'}})
- [Nuxeo - Setting up your environment]({{page space='nxdoc' page='setting-up-your-nuxeo-environment'}})

{{/panel}}</div></div>

&nbsp;
