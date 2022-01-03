---
title: Java Client SDK 3.12
description: Nuxeo Client SDK - Java
review:
    comment: ''
    date: '2019-06-18'
    status: ok
labels:
  - java-client
toc: true
is_overview: true
---

The Nuxeo Java Client is a Java client library for Nuxeo REST API.

It is compatible with all Nuxeo versions as of LTS 2015.

## Getting Started

Follow this [tutorial]({{page space='nxdoc' page='setting-up-your-nuxeo-environment'}}) to have a running Nuxeo instance on your local machine.

## Installation

To use nuxeo-java-client, you can download it from our Nexus: [Nuxeo Client Library 3.12.0](https://packages.nuxeo.com/#browse/search/maven=attributes.maven2.artifactId%3Dnuxeo-java-client%20AND%20version%3D3.12.0).

If you use Maven, you need to have nuxeo-java-client as dependency:

```xml
<dependency>
  <groupId>org.nuxeo.client</groupId>
  <artifactId>nuxeo-java-client</artifactId>
  <version>3.12.0</version>
</dependency>

<repository>
  <id>public-releases</id>
  <url>https://packages.nuxeo.com/repository/maven-public</url>
</repository>
```

## Client Creation

Nuxeo Java Client is created with help of its `Builder`. Once every options are submitted, last step is to build the client and test the connection to Nuxeo Server:

```java
NuxeoClient client = new NuxeoClient.Builder()
                                    .url("http://localhost:8080/nuxeo")
                                    .authentication("Administrator", "Administrator")
                                    .schemas("*") // fetch all document schemas
                                    .connect();
```

Client is now ready to execute requests to Nuxeo Server. After its creation, it will contain the current Nuxeo user and the Nuxeo Server version.

More documentation about client [options]({{page page='configuration'}}).

{{#> callout type='note' heading='just build the client'}}
You can use `build` instead of `connect` to not perform any request to check the authentication.
{{/callout}}

## Fetch Your First Document

Below is how we can retrieve default domain from a stock Nuxeo Server:

```java
Document domain = client.repository().fetchDocumentByPath("/default-domain");
String title = domain.getPropertyValue("dc:title"); // should be equal to "Domain"
```

## Compatibility

The 3.12 Nuxeo Java Client version is compatible with Java 11 and greater.

&nbsp;

<div class="row" data-equalizer data-equalize-on="medium"><div class="column medium-6">{{#> panel heading='Related sections in this documentation'}}

- [Nuxeo - Getting Started]({{page space='nxdoc' page='getting-started'}})
- [Nuxeo - Setting up your environment]({{page space='nxdoc' page='setting-up-your-nuxeo-environment'}})

{{/panel}}</div></div>

&nbsp;
