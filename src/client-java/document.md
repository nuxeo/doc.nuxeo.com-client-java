---
title: Document
description: Handle documents
review:
    comment: ''
    date: '2019-06-18'
    status: ok
labels:
    - java-client
tree_item_index: 300
section_parent: java
toc: true

---

This section is about how handling documents with Nuxeo Java Client.

Almost all APIs have an asynchronous equivalent, asynchronous APIs share the same name than synchronous APIs but they return nothing and have a `retrofit2.Callback` parameter in addition.

Let's first retrieve repository manager to handle our documents:
```java
Repository repository = client.repository();
```

## Create

APIs below are available to create a document:
- createDocumentByPath which takes parent path and a `Document`
- createDocumentById which takes parent id and a `Document`

Let's create a folder:
```java
Document folder = Document.createWithName("folder", "Folder");
folder = repository.createDocumentByPath("/", folder);
```

Now we can create a file and a note in it:
```java
Document note = Document.createWithName("note001", "Note");
note = repository.createDocumentByPath("/folder", note);

Document file = Document.createWithName("file001", "File");
file = repository.createDocumentById(folder.getId(), file);
```

We can also provide some property at creation:
```java
Document note = Document.createWithName("note002", "Note");
note.setPropertyValue("note:note", "Some note for my second note document")
note.setPropertyValue("dc:description", "Description of my document");
note = repository.createDocumentByPath("/folder", note);
```

## Fetch

APIs below are available to fetch document:
- fetchDocumentRoot
- fetchDocumentById which takes document id to fetch
- fetchDocumentByPath which takes document path to fetch
- fetchChildrenById which takes parent id of children to fetch
- fetchChildrenByPath which takes parent path of children to fetch

Let's fetch our folder:
```java
Document folder = repository.fetchDocumentByPath("/folder");
String title = folder.getPropertyValue("dc:title"); // equals to folder
```

We can retrieve its children with either `fetchChildrenById` or `fetchChildrenByPath` which take the parent id or path:
```java
Documents children = repository.fetchChildrenById(folder.getId());
int childrenSize = children.size() // equals to 3
List<Document> childrenList = children.getEntries();
Document note = children.streamEntries()
                        .filter(doc -> "note001".equals(doc.getName()))
                        .findFirst()
                        .orElseThrow(() -> new RuntimeException("Unable to find 'note001'"));
```

As `Document` is a [connectable entity]({{page page='configuration#objects'}}) and we get `folder` from client we can do the following:
```java
Documents children = folder.fetchChildren();
```

{{#> callout type='note' heading='fetchChildren behavior'}}
`fetchChildren*`  returns a subset of all children. Because they leverage `ChildrenAdapter` server side which leverages `CURRENT_DOC_CHILDREN` page provider.

By default this page provider returns only 50 elements per page.

If you want to retrieve all children you should use [page provider APIs](#page-provider) instead.
{{/callout}}

## Update

APIs below are available to update document:
- updateDocument which takes a document (with an id)

During the update, only dirty properties detected on client side are sent to the server.

Let's update our note:
```java
Document note = repository.fetchDocumentByPath("/folder/note001");
note.setPropertyValue("note:note", "Some update in content");
 // previous note is not updated, you need to get returned document for further operation
note = repository.updateDocument(note);
```

As `Document` is a [connectable entity]({{page page='configuration#objects'}}) we can also do the following:
```java
Document note = repository.fetchDocumentByPath("/folder/note001");
note.setPropertyValue("note:note", "Some update in content");
 // previous note is not updated, you need to get returned document for further operation
note = note.updateDocument();
```

You can also update document without fetching it first, if and only if you know its id:
```java
Document note = Document.createWithId("UUID", "OPTIONAL_TYPE");
note.setPropertyValue("note:note", "Some update in content");
note = repository.updateDocument(note);
```

Id is mandatory because this is how Nuxeo Server resolved the document to update. There's currently no way to update a document with only its path.
Also, with this way, you can't call `Document#updateDocument` because your object is not connected with client, but the one returned from `repository#updateDocument` is connected.

## Delete

APIs below are available to delete document:
- deleteDocument which takes document id to delete
- deleteDocument which takes document to delete

Just simple as:
```java
repository.deleteDocument("UUID");
```

## Adapters

You can use the Adapters API to access the [Nuxeo Web Adapter]({{page space='nxdoc' page='rest-api-web-adapters'}}) of your document with the method `Document#adapter`.

The Java client provides the following adapters:
- AnnotationAdapter
- CommentAdapter

You can use it like below:

```java
Comment comment = new Comment();
comment.setText("A comment");
Document note = repository.fetchDocumentByPath("/folder/note001");
note.adapter(CommentAdapter::new).create(comment);
```

## Manipulate Document Properties

Once you have retrieved your document you can get/set property values with the API below:
- getPropertyValue
- setPropertyValue

Example:

```java
// getPropertyValue
Document note = repository.fetchDocumentByPath("/folder/note001");
String noteText = note.getPropertyValue("note:note");
// getPropertyValue with XPath
Document note = repository.fetchDocumentByPath("/folder/note001");
String firstContributor = note.getPropertyValue("dc:contributors/0");
// setPropertyValue
Document note = repository.fetchDocumentByPath("/folder/note001");
note.setPropertyValue("note:note", "Some update in content");
// setPropertyValue with list/complex
Document note = repository.fetchDocumentByPath("/folder/note001");
note.setPropertyValue("dc:subjects", List.of("art/architecture", "sciences/astronomy"));
```

## Enrichers / Fetch Properties

As seen in [configuration]({{page page='configuration'}}), you can configure enrichers and fetch properties for your manager, here the repository.

Depending on enricher or property to fetch your use, you will either get a proper Java Client object or a JSON representation in Map and List. Deserialization into proper object leverage entities registered during client creation, see [marshalling configuration]({{page page='configuration'}}#marshalling) for more information.

For instance, `breadcrumb` enricher adds a `Documents` JSON representation in the context parameter:
```java
Document note = repository.enrichersForDocument("breadcrumb").fetchDocumentByPath("/folder/note001");
Documents breadcrumb = note.getContextParameter("breadcrumb");
```

Whereas `thumbnail` enricher writes an URL:
```java
Document file = repository.enrichersForDocument("thumbnail").fetchDocumentByPath("/folder/file");
// as thumbnail just returned an url we can cast to Map<String, String>
Map<String, String> thumbnail = note.getContextParameter("thumbnail");
String thumbnailUrl = thumbnail.get("url");
```

Or `renditions` enricher which writes several fields:
```java
Document file = repository.enrichersForDocument("renditions").fetchDocumentByPath("/folder/file");
// as renditions returned a list of objects containing string we can cast to List<Map<String, String>>
List<Map<String, String>> renditions = note.getContextParameter("renditions");
for (Map<String, String> rendition : renditions) {
    String name = rendition.get("name");
    String kind = rendition.get("kind");
    String icon = rendition.get("icon");
    String url = rendition.get("url");
}
```

Same behavior applies to fetch properties.

For instance, on `dc:creator` we have:
```java
Document file = repository.fetchPropertiesForDocument("dc:creator")
                          .fetchDocumentByPath("/folder/file");
User creator = file.getPropertyValue("dc:creator");
```

You can request several enrichers and/or properties to fetch as it:
```java
Document file = repository.enrichersForDocument("renditions", "thumbnail", "breadcrumb")
                          .fetchPropertiesForDocument("dc:creator", "dc:contributors")
                          .fetchDocumentByPath("/folder/file");
```

## Query and Pagination

APIs below are available to query documents:
- query which takes an NXQL query
- query which takes an NXQL query, page size, current page index, max results, sort by, sort order and query params (except query, others are optional)

Query method taking only an NXQL query is driven by server side configuration, see query endpoint [documentation]({{page space='nxdoc' page='query-endpoint'}}).

Let's see how retrieving a paginated query result and loop over pages:
```java
Documents documents = null;
do {
    String pageIndex = documents == null ? "0" : String.valueOf(documents.getCurrentPageIndex() + 1);
    documents = repository.query("SELECT * FROM Document", "50", pageIndex, null,
            "dc:title,dc:description", "ASC,DESC");
    for (Document document : documents.getEntries()) {
        // do something with document
    }
} while (documents.isNextPageAvailable());
```

If you want to use query params, you can use the API below:
```java
repository.query("SELECT * FROM Document WHERE dc:title= ?", "50", pageIndex, null, 
    "dc:title,dc:description", "ASC,DESC", "My Document");
```

## Page provider

API below is available to query documents with page provider:
- queryByProvider which takes a page provider name, page size, current page index, max results, sort by, sort order and query params (except page provider name, others are optional)

Let's see how retrieving a paginated page provider result and loop over pages:
```java
Documents documents = null;
do {
    String pageIndex = documents == null ? "0" : String.valueOf(documents.getCurrentPageIndex() + 1);
    documents = repository.queryByProvider("CURRENT_DOC_CHILDREN", "50", pageIndex, null,
            "dc:title,dc:description", "ASC,DESC");
    for (Document document : documents.getEntries()) {
        // do something with document
    }
} while (documents.isNextPageAvailable());
```

If your page provider declares query params, you can use for instance:
```xml
<coreQueryPageProvider name="search_with_params">
  <pattern>SELECT * From Note WHERE dc:title = ?</pattern>
  <pageSize>50</pageSize>
</coreQueryPageProvider>
```
```java
repository.queryByProvider("search_with_params", "50", pageIndex, null,
    "dc:title,dc:description", "ASC,DESC", "My Note");
```

## Permissions

APIs below are available to fetch permissions:
- fetchACPById which takes a document id
- fetchACPByPath which takes a document path

Let's fetch ACP of our note:
```java
ACP acp = repository.fetchACPByPath("/folder/note001");
ACL acl = acp.getAcls()
             .stream()
             .filter(acl -> ACL.LOCAL_ACL.equals(acl.getName()))
             .findFirst()
             .orElseThrow(() -> new RuntimeException("Unable to find 'local' acl"));
List<ACE> aces = acl.getAces();
```

As `Document` is a [connectable entity]({{page page='configuration'}}#objects) we can fetch ACP directly from it but also handle them:
```java
Document note = repository.fetchDocumentByPath("/folder/note001");
// fetch ACP
ACP acp = note.fetchPermissions();
// remove all local permissions for john
note.removePermission("john"); // does a call to nuxeo server
// add a new permission for john
ACE ace = new ACE();
ace.setCreator("Administrator");
ace.setUsername("john");
ace.setPermission("READ");
ace.setBegin(new GregorianCalendar(2015, Calendar.JUNE, 20));
ace.setEnd(new GregorianCalendar(2015, Calendar.JUNE, 27));
note = note.addPermission(ace); // does a call to nuxeo server
// send an external invitation to jean@dev.null
ACE invitation = new ACE();
invitation.setCreator("Administrator");
invitation.setPermission("READ");
invitation.setBegin(new GregorianCalendar(2015, Calendar.JUNE, 20));
invitation.setEnd(new GregorianCalendar(2015, Calendar.JUNE, 27));
note = note.addInvitation(invitation, "jean@dev.null"); // does a call to nuxeo server
```
