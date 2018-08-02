---
title: Authentication
description: Handle authentication with Nuxeo Java Client
review:
    comment: ''
    date: '2018-01-02'
    status: ok
labels:
    - java-client
tree_item_index: 100
section_parent: java
toc: true

---

Nuxeo Java Client handles these authentication method:
- basic authentication via `BasicAuthInterceptor`
- portal SSO authentication via `PortalSSOAuthInterceptor`
- token authentication via `TokenAuthInterceptor`

### Basic authentication

The default authentication method is the basic authentication. Client builder has a convenient method to configure Nuxeo Java Client with a basic authentication:
```java
new NuxeoClient.Builder().authentication("Administrator", "Administrator");
```

Whose equivalent is:
```java
new NuxeoClient.Builder().authentication(new BasicAuthInterceptor("Administrator", "Administrator"));
```

### Portal SSO authentication

Once Portal SSO authentication has been activated on your Nuxeo server you can use this method to connect the java client with:
```java
new NuxeoClient.Builder().authentication(new PortalSSOAuthInterceptor("Administrator", "nuxeo5secretkey"));
```

### Token authentication
Token authentication is enabled by default on Nuxeo server, to use it in java client
```java
new NuxeoClient.Builder().authentication(new TokenAuthInterceptor("nuxeoToken"));
```

### Implement your own

By design, Nuxeo Java Client leverage `OkHttp` `Interceptor` interface to handle authentication between client and server.

This is what expect the method: `NuxeoClient.Builder#authentication(Interceptor)`.

You just need to implement one method which will be responsible to inject the appropriate header:
```java
public class MyAuthInterceptor implements Interceptor {

    protected String token;

    public MyAuthInterceptor(String token) {
        this.token = token;
    }

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request() // get the request
                               .newBuilder() // create a new builder from it
                               .addHeader("MY-HEADER", token) // add header
                               .build(); // finally build request
        return chain.proceed(request);
    }

}
```

Then you can use it:
```java
new NuxeoClient.Builder().authentication(new MyAuthInterceptor("mySuperToken"));
```

&nbsp;

<div class="row" data-equalizer data-equalize-on="medium"><div class="column medium-6">{{#> panel heading='Related sections in this documentation'}}

- [Nuxeo - REST Authentication]({{page space='nxdoc' page='request-authentication'}})
- [Nuxeo - Portal SSO Authentication]({{page space='nxdoc' page='using-sso-portals'}})

{{/panel}}</div></div>

&nbsp;
