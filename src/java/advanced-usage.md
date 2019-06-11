---
title: Advanced Usages
description: How to extend Nuxeo Java Client
review:
    comment: ''
    date: '2018-01-02'
    status: ok
labels:
    - java-client
tree_item_index: 900
section_parent: java
toc: true
---

This section is about how to extend Nuxeo Java Client.

## Deserialize Your POJO

Let's say that you have added writer and reader server side for a new kind of object. Now you want to have proper Java object within the client. The new entity has `custom-json-object` entity type, it will help Nuxeo Java Client to deserialize it.

First, we will create a Java class for this entity:
```java
public class CustomJSONObject extends Entity {

    public static final String ENTITY_TYPE = "custom-json-object";

    @JsonProperty("id")
    private String id;

    @JsonProperty("token")
    private String token;

    public CustomJSONObject() {
        super(ENTITY_TYPE);
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }

}
```

Now that we have this object, we can register it during client creation:
```java
NuxeoClient client = new NuxeoClient.Builder()
                                    .url("http://localhost:8080/nuxeo")
                                    .authentication("Administrator", "Administrator")
                                    .schemas("*")
                                    .registerEntity(CustomJSONObject.ENTITY_TYPE, CustomJSONObject.class)
                                    .connect();
```

Now we can call an operation returning this entity and get a proper Java object:
```java
CustomJSONObject custom = client.operation("Custom.MyOperation").execute();
```

## Make Your POJO a Connectable Entity

We want to change our POJO to make it connectable, this will allow the convenient method to send requests to the Nuxeo Server on the object.

We bind our POJO to `RepositoryAPI` in order to do requests, let's change it to:
```java
public class CustomJSONObject extends ConnectableEntity<RepositoryAPI, CustomJSONObject> {

    public static final String ENTITY_TYPE = "custom-json-object";

    ...

    public CustomJSONObject() {
        super(ENTITY_TYPE, RepositoryAPI.class);
    }

    /**
     * Constructor to instantiate a connected object.
     */
    public CustomJSONObject(NuxeoClient client) {
        super(ENTITY_TYPE, RepositoryAPI.class, client);
    }

    ...

}
```

As it, each time we will get this object from client, we will be able to call methods doing request to Nuxeo Server.

For example, we can have methods to download different kind of blob:
```java
public class CustomJSONObject extends ConnectableEntity<RepositoryAPI, CustomJSONObject> {

    ...

    public StreamBlob streamMainBlob() {
        return fetchResponse(api.streamBlobById(id, "file:content"));
    }

    public StreamBlob streamMainThumbnail() {
        return fetchResponse(api.streamBlobById(id, "thumbnail:thumbnail"));
    }

    ...

}
```

In a [connectable entity]({{page page='configuration#objects'}}) you also have access to `NuxeoClient`, which allows to use other managers from your POJO:
```java
public class CustomJSONObject extends ConnectableEntity<RepositoryAPI, CustomJSONObject> {

    ...

    public CustomJSONObject refresh() {
        return nuxeoClient.operation("Custom.Refresh").input(this).execute();
    }

    ...

}
```

## Create a New Retrofit API

You can also create new Retrofit API in order to use your custom Nuxeo endpoint. [Retrofit APIs](https://square.github.io/retrofit/#api-declaration) defines endpoint to hit, their path starts after the base url, after `http://localhost:8080/nuxeo` for instance.

Let's create a basic API with at least `streamBlobById` as we've used it in previous example:
```java
public interface CustomJSONAPI {

    @GET("id/{documentId}/@blob/{fieldPath}")
    Call<StreamBlob> streamBlobById(@Path("documentId") String documentId,
            @Path(value = "fieldPath", encoded = true) String fieldPath);

    @PUT("custom/json/api/{id}")
    Call<CustomJSONObject> refresh(@Path("id") String id, @Body CustomJSONObject object);
}
```

In our POJO, we make these changes:
```java
public class CustomJSONObject extends ConnectableEntity<CustomJSONAPI, CustomJSONObject> {

    ...

    public CustomJSONObject() {
        super(ENTITY_TYPE, CustomJSONAPI.class);
    }

    /**
     * Constructor to instantiate a connected object.
     */
    public CustomJSONObject(NuxeoClient client) {
        super(ENTITY_TYPE, CustomJSONAPI.class, client);
    }

    ...

    public CustomJSONObject refresh() {
        return fetchResponse(api.refresh(id, this));
    }

    ...

}
```

* * *

**POJO**: Plain Old Java Object
