---
title: Directory
description: Handle directories
review:
    comment: ''
    date: '2018-01-02'
    status: ok
labels:
    - java-client
tree_item_index: 600
section_parent: java
toc: true

---

This section is about how handling directories (and vocabularies) with Nuxeo Java Client.

Let's first retrieve directory manager to handle them:
```java
DirectoryManager directoryManager = client.directoryManager();
```

Directory manager only haves two APIs:
- fetchDirectories to fetch all directories configured on Nuxeo Server
- directory which takes a directory name and returns the directory entity

## Create

```java
DirectoryEntry entry = new DirectoryEntry();
entry.putIdProperty("foo");
entry.putLabelProperty("Foo");
directoryManager.directory("nature").createEntry(entry);
```

## Fetch

APIs below are available to fetch directory entry:
- fetchEntries which takes nothing
- fetchEntries which takes currentPageIndex, pageSize, maxResult, sortBy and sortOrder
- fetchEntry which takes an entry id

Let's fetch all entries:
```java
DirectoryEntries entries = directoryManager.directory("nature").fetchEntries();
DirectoryEntry foo = entries.getDirectoryEntry("foo");
```

Let's fetch our entry:
```java
DirectoryEntry foo = directoryManager.directory("nature").fetchEntry("foo");
```

## Update

APIs below are available to update directory entry:
- updateEntry which takes a directory entry

Let's update our entry:
```java
Directory nature = directoryManager.directory("nature");
DirectoryEntry foo = nature.fetchEntry("foo");
foo.putLabelProperty("New Foo");
nature.updateEntry(foo);
```

As `DirectoryEntry` is a [connectable entity]({{page page='configuration#objects'}}) and we get `foo` from client we can do the following:
```java
DirectoryEntry foo = directoryManager.directory("nature").fetchEntry("foo");
foo.putLabelProperty("New Foo");
foo.update();
```

## Delete

APIs below are available to delete directory entry:
- deleteEntry which takes a directory entry id

Let's delete foo:
```java
directoryManager.directory("nature").deleteEntry("foo");
```

As `DirectoryEntry` is a [connectable entity]({{page page='configuration#objects'}}) and we get `foo` from client we can do the following:
```java
DirectoryEntry foo = directoryManager.directory("nature").fetchEntry("foo");
foo.delete();
```
