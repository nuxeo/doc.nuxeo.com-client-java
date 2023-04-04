---
title: Automation
description: Run Operation and Automation Chain
review:
    comment: ''
    date: '2019-06-18'
    status: ok
labels:
    - java-client
tree_item_index: 800
section_parent: java
toc: true

---

This section is about how to run automation with Nuxeo Java Client.

Operation manager is instantiated for each automation to run, this gives us high isolation for each request to automation.

Automation input can be anything you want, Blob, Blobs, Document, Documents or primitive type. To give a document reference as input to automation, you need to use `DocRef` or `DocRefs`.

Automation output is dynamic and leverage [marshaller]({{page page='configuration'}}#marshalling) capabilities in order to return proper Java Client object, if marshaller is not found client will return a plain string.

## Document.Create

Let's create a file:
```java
Document file = client.operation("Document.Create")
                      .param("name", "file001")
                      .param("type", "File")
                      .param("properties", "dc:title=My File\ndc:description=A short description")
                      .input(new DocRef("/"))
                      .execute();
```

## Document.Update

Let's update it:
```java
file = client.operation("Document.Update")
             .param("properties", "dc:nature=card")
             .input(new DocRef("/file001"))
             .execute();
```


## Blob.Attach

Let's attach a blob to our file:
```java
client.operation("Blob.Attach")
      .param("document", "/file001")
      .param("xpath", "file:content")
      .input(new FileBlob(File))
      .voidOperation(true) // allows to not download blob in response
      .execute();
```

## Blob.Get

Let's download our blob:
```java
Blob blob = client.operation("Blob.Get")
                  .param("xpath", "file:content")
                  .input(new DocRef("/file001"))
                  .execute();
```

## Document.GetVersions

Let's get versions of our file:
```java
Documents versions = client.operation("Document.GetVersions")
                           .input(new DocRef("/file001"))
                           .execute();
```
