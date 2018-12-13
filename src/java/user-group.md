---
title: User & Group
description: Handle users & groups
review:
    comment: ''
    date: '2018-01-02'
    status: ok
labels:
    - java-client
tree_item_index: 500
section_parent: java
toc: true

---

This section is about how handling users and groups with Nuxeo Java Client.

User is highly configurable in Nuxeo. Java Client allows such configurability but also offer common methods to get name, password, email...
This page presents examples on basic configuration. If you need to get/set more properties you can use method below:
- getProperties
- setProperties

Let's first retrieve user manager to handle them:
```java
UserManager userManager = client.userManager();
```

## Create

APIs below are available to create user / group:
- createUser which takes an user
- createGroup which takes a group

Let's first create a user:
```java
User user = new User();
user.setUserName("john");
user.setPassword("MY_STRONG_PASSWORD");
user.setFirstName("John");
user.setLastName("Doe");
user.setCompany("Nuxeo");
user = userManager.createUser(user);
```

Now we want to create a new group and add John to it. Furthermore we want this group to be part of `members` group present on stock Nuxeo Server:
```java
Group group = new Group();
group.setGroupName("myGroup");
group.setGroupLabel("My Group");
group.setMemberUsers(Arrays.asList("john"));
group.setParentGroups(Arrays.asList("members"));
group = userManager.createGroup(group);
```

## Fetch

APIs below are available to fetch user / group:
- fetchUser which takes an user name
- fetchCurrentUser which takes nothing
- fetchGroup which takes a group name

Let's fetch our user:
```java
User user = userManager.fetchUser("john");
String firstName = user.getFirstName(); // equals to John
String lastName = user.getLastName(); // equals to Doe
String password = user.getPassword(); // equals to null / empty

User currentUser = userManager.fetchCurrentUser(); // get the log-in user
```

Let's fetch our group:
```java
Group group = userManager.fetchGroup("myGroup");
String label = group.getGroupLabel(); // equals to My Group
```

As `Group` is a [connectable entity]({{page page='configuration#objects'}}) and we get `myGroup` from client we can do the following:
```java
Users users = group.fetchMemberUsers();
Groups subGroups = group.fetchMemberGroups();
```

You can also retrieve them during the fetch with help of fetch properties:
```java
Group group = userManager.fetchPropertiesForGroup("memberUsers", "memberGroups", "parentGroups")
                         .fetchGroup("myGroup");
List<String> memberUsers = group.getMemberUsers();
List<String> memberGroups = group.getMemberGroups();
List<String> parentGroups = group.getParentGroups();
```

## Update

APIs below are available to update user / group:
- updateUser which takes an user
- updateUser which takes an user name and a, user
- updateGroup which takes a group
- updateGroup which takes a group name and a group
- addUserToGroup which takes an user name and a group name
- attachGroupToUser which takes a group name and an user name

Let's update our user:
```java
User john = userManager.fetchUser("john");
john.setLastName("Peper");
userManager.updateUser(john);
```

Let's update our group:
```java
User myGroup = userManager.fetchUser("myGroup");
myGroup.setGroupLabel("Company Group");
userManager.updateGroup(myGroup);
```

Now we want to add another user to our group, let's say `jean`:
```java
userManager.addUserToGroup("jean", "myGroup");
```

## Delete

APIs below are available to delete user / group:
- deleteUser which takes an user name
- deleteGroup which takes a group name

Let's delete our entities:
```java
userManager.deleteUser("john");
userManager.deleteGroup("myGroup");
```

## Search

APIs below are available to search user / group:
- searchUser which takes a pattern
- searchUser which takes a pattern, current page index and page size
- searchGroup which takes a pattern
- searchGroup which takes a pattern, current page index and page size

These APIs leverage a page provider server side, depending on your configuration you can have different behavior than the regular one leveraged in these examples.

Examples are available for both users and groups.

Let's retrieve all groups:
```java
Groups groups = userManager.searchGroup("*"); // retrieve maxResult groups
for (Group group : groups.getEntries()) {
    // do something with group
}
```

Let's see how retrieving a paginated query result and loop over pages:
```java
Users users = null;
do {
    int pageIndex = users == null ? 0 : users.getCurrentPageIndex() + 1;
    users = userManager.searchUser("*", pageIndex, 50);
    for (User user : users.getEntries()) {
        // do something with user
    }
} while (users.isNextPageAvailable());
```

Depending on you directory configuration, pattern will act differently, see `substringMatchType` in your directory configuration.

Let's search for john:
```java
Users users = userManager.searchUser("jo");
List<User> userList = users.getEntries(); // contains john user
```
