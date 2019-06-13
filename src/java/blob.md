---
title: Blob
description: Handle blobs
review:
    comment: ''
    date: '2019-06-18'
    status: ok
labels:
    - java-client
tree_item_index: 400
section_parent: java
toc: true

---

This section is about how handling blobs with Nuxeo Java Client.

There are several ways in Nuxeo and so in Java Client to handle blobs.

## Blob Interface

In Java Client, we use `Blob` as the main interface to deal with blobs with server. In this page we mainly use `FileBlob` for clarity nevertheless every example are also working with `StreamBlob`.

For both implementation, we highly recommend to give `mimeType` if possible. This allows less processing for Nuxeo Server.

## Batch Upload

The batch upload manager leverage [batch upload endpoint]({{page space='nxdoc' page='batch-upload-endpoint'}}) server side. It is designed to upload a batch of files that can then be used in Automation or as a property of a document through the REST API. It also supports the [chunk mode]({{page space='nxdoc' page='batch-upload-endpoint#uploading-a-chunk'}}) if manager is configured to use it.

Let's first retrieve batch upload manager to handle our batch:
```java
BatchUploadManager batchUploadManager = client.batchUploadManager();
```

### Attach a Blob to a Document Using Automation

We're assuming `/file001` already exists. We're going to upload a blob with batch upload manager and then attach it to `file001`.

```java
// create a new batch
BatchUpload batchUpload = batchUploadManager.createBatch();
Blob fileBlob = new FileBlob(file);
batchUpload = batchUpload.upload("0", fileBlob);

// attach the blob
batchUpload.operation(Operations.BLOB_ATTACH_ON_DOCUMENT)
           .param("document", "/file001") // document to attach
           .execute();
```

### Attach a Blob to a Document Using REST API

We're assuming `/file001` already exists. We're gonna upload a blob with batch upload manager and then attach it to `file001`.

```java
// create a new batch
BatchUpload batchUpload = batchUploadManager.createBatch();
Blob fileBlob = new FileBlob(file);
batchUpload = batchUpload.upload("0", fileBlob);

// attach the blob
Document doc = nuxeoClient.repository().fetchDocumentByPath("/file001");
doc.setPropertyValue("file:content", batchUpload.getBatchBlob());
doc = doc.updateDocument();
```

### Create a Document from a Blob

We're gonna upload a blob with batch upload manager and then create a document referencing this blob.

```java
// create a new batch
BatchUpload batchUpload = batchUploadManager.createBatch();
Blob fileBlob = new FileBlob(file);
batchUpload = batchUpload.upload("0", fileBlob);

// create document with the blob
Document doc = Document.createWithName("file002", "File");
doc.setPropertyValue("file:content", batchUpload.getBatchBlob());
doc = nuxeoClient.repository().createDocumentByPath("/", doc);
```

## Automation (Multipart Upload)

You can upload blob when using automation. This will upload the blob using HTTP multipart.

### Attach a Blob to a Document

We're assuming `/file001` already exists. We're gonna upload and attach several blobs to `file001` using `Blob.AttachOnDocument`.

```java
nuxeoClient.operation(Operations.BLOB_ATTACH_ON_DOCUMENT)
           .voidOperation(true) // allows to not download blob in response
           .param("document", "/file001") // document to attach
           .param("xpath", "files:files") // xpath to store blobs
           .input(new Blobs(Arrays.asList(new FileBlob(file1), new FileBlob(file2))))
           .execute();
```

### Create a Document from a Blob

We're gonna upload, create `file002` and attach a blob using `FileManager.Import`:

```java
Document file = nuxeoClient.operation("FileManager.Import")
                           .context("currentDocument", "/") // parent document
                           .input(new FileBlob(file1, "file002"))
                           .execute();
```

## Download Blob

APIs below are available on repository manager to fetch blob:
- streamBlobById which takes a document id and a xpath
- streamBlobByPath which takes a document path and a xpath

You also have `streamBlob` on `Document` when object is connected.

In both case, you get a `StreamBlob` exposing an `InputStream` which must be closed because it is directly linked to the HTTP connection.

```java
StreamBlob blob = nuxeoClient.repository().streamBlobByPath("/file001", "file:content");
try (InputStream is = blob.getStream()) {
    String content = org.apache.commons.io.IOUtils.toString(is, StandardCharsets.UTF_8);
}
```
