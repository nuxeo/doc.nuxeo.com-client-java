---
title: Authentication
description: Handle authentication with Nuxeo Java Client
review:
    comment: ''
    date: '2019-06-18'
    status: ok
labels:
  - java-client
tree_item_index: 100
section_parent: java
toc: true
---

Nuxeo Java Client handles these authentication methods:
- basic authentication via `BasicAuthInterceptor`
- portal SSO authentication via `PortalSSOAuthInterceptor`
- token authentication via `TokenAuthInterceptor`
- OAuth2 authentication via `OAuth2AuthInterceptor`

## Basic Authentication

The default authentication method is the basic authentication. Client builder has a convenient method to configure Nuxeo Java Client with a basic authentication:

```java
new NuxeoClient.Builder().authentication("Administrator", "Administrator");
```

Whose equivalent is:

```java
new NuxeoClient.Builder().authentication(new BasicAuthInterceptor("Administrator", "Administrator"));
```

## Portal SSO Authentication

Once Portal SSO authentication has been activated on your Nuxeo server you can use this method to connect the Java client with:

```java
new NuxeoClient.Builder().authentication(new PortalSSOAuthInterceptor("Administrator", "nuxeo5secretkey"));
```

## Token Authentication

Token authentication is enabled by default on Nuxeo server, to use it in Java client:
```java
new NuxeoClient.Builder().authentication(new TokenAuthInterceptor("nuxeoToken"));
```
Token obtention is not implemented in java client and should be done by following this [documentation](https://github.com/nuxeo/nuxeo/tree/master/modules/platform/login/nuxeo-platform-login-token#implementation).

## JWT Authentication

JWT authentication can be enabled on Nuxeo server if you set up a secret with `nuxeo.jwt.secret` in your nuxeo.conf, to use it in java client:
```java
new NuxeoClient.Builder().authentication(new JWTAuthInterceptor("nuxeoToken"));
```
There's currently no way to obtain a JWT token through built-in REST API.

## OAuth 2 Authentication

### With An Authorization Flow

OAuth 2 authentication is enabled by default on Nuxeo server, you just need to create an OAuth 2 Client in Nuxeo, see [Nuxeo - OAuth2 Client Registration]({{page space='nxdoc' page='using-oauth2'}}#client-registration).

Once you have your `clientId` and `clientSecret` you need to implement an [OAuth 2 Grant Flow]({{page space='nxdoc' page='using-oauth2'}}#oauth-2-authorization-grant-flow) in your application to retrieve an authentication code. With this code you can setup the OAuth 2 authentication in the Java client with:

```java
new NuxeoClient.Builder().authentication(
    OAuth2AuthInterceptor.obtainAuthFromAuthorizationCode("http://localhost:8080/nuxeo", clientId, authCode));
// or if you have set up a client with a clientSecret
new NuxeoClient.Builder().authentication(
    OAuth2AuthInterceptor.obtainAuthFromAuthorizationCode("http://localhost:8080/nuxeo", clientId, clientSecret, authCode));
```

After that, your client will be able to access Nuxeo using OAuth 2 and it will refresh the OAuth 2 token if needed.

### With an Access Token

You can also use OAuth 2 authentication within the Java client with an access token. This is useful when the software using the client is responsible to obtain and refresh the OAuth Token.

```java
new NuxeoClient.Builder().authentication(OAuth2AuthInterceptor.createAuthFromToken(accessToken));
```

## Implement Your Own

By design, Nuxeo Java Client leverages `OkHttp` `Interceptor` interface to handle authentication between client and server.

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
