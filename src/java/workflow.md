---
title: Workflow
description: Handle workflows
review:
    comment: ''
    date: '2018-01-02'
    status: ok
labels:
    - java-client
tree_item_index: 700
section_parent: java
toc: true

---

This section is about how handling workflows and tasks with Nuxeo Java Client.

There's no dedicated manager for workflows. We can handle them through repository manager, document or user manager.

Let's get our repository manager:
```java
Repository repository = client.repository();
```

Let's create a Note too:
```java
Document note = Document.createWithName("note", "Note");
note = repository.createDocumentByPath("/", note);
```

## Model Retrieval

APIs below are available on repository manager to fetch workflow model:
- fetchWorkflowModels which takes nothing
- fetchWorkflowModel which takes the workflow model name
- fetchWorkflowModelGraph which takes the workflow model name

Let's retrieve all workflow models on Nuxeo Server:
```java
Workflows workflows = repository.fetchWorkflowModels();
```

Next examples will use the `SerialDocumentReview` workflow which is present on a bare Nuxeo Server, let's retrieve it:
```java
Workflow serialWorkflow = repository.fetchWorkflowModel("SerialDocumentReview");
String name = serialWorkflow.getName(); // equals to SerialDocumentReview
String title = serialWorkflow.getTitle(); // equals to wf.serialDocumentReview.SerialDocumentReview
String modelName = serialWorkflow.getWorkflowModelName(); // equals to SerialDocumentReview
```

## Create - Start

APIs below are available on repository manager to create/start a workflow instance:
- startWorkflowInstanceWithDocId which takes a document id and a workflow model
- startWorkflowInstanceWithDocPath which takes a document path and a workflow model

Let's start serial workflow on our note:
```java
Workflow serialWorkflowInstance = repository.startWorkflowInstanceWithDocPath("/note", serialWorkflow);
String name = serialWorkflow.getName(); // equals to SerialDocumentReview
String title = serialWorkflow.getTitle(); // equals to wf.serialDocumentReview.SerialDocumentReview
String modelName = serialWorkflow.getWorkflowModelName(); // equals to SerialDocumentReview
String state = serialWorkflowInstance.getState(); // equals to running
String initiator = serialWorkflowInstance.getInitiator(); // equals to username who started it
List<String> attachedDocIds = serialWorkflowInstance.getAttachedDocumentIds(); // contains note id
```

As `Document` is a [connectable entity]({{page page='configuration#objects'}}) and we get `note` from client we can do the following:
```java
Workflow serialWorkflowInstance = note.startWorkflowInstance(serialWorkflow);
```

## Fetch

APIs below are available on repository manager to fetch workflow instance:
- fetchWorkflowInstance which takes a workflow instance id
- fetchWorkflowInstancesByDocId which takes a document id
- fetchWorkflowInstancesByDocPath which takes a document path

Let's retrieve workflow instances on our note:
```java
// contains the previous started workflow
Workflows workflows = repository.fetchWorkflowInstancesByDocPath("/note");
```

As `Document` is a [connectable entity]({{page page='configuration#objects'}}) and we get `note` from client we can do the following:
```java
Workflows workflows = note.fetchWorkflowInstances();
```

You can also retrieve tasks on a document:
```java
Tasks tasks = note.fetchTasks();
```

## Update - Work with Task

There's task manager to handle tasks for a workflow.

APIs below are available on task manager to handle tasks:
- fetchTasks which take an user id, a workflow instance id and a workflow model name
- fetchTask which takes a task id
- reassign which takes a task id, a list of actors and a comment
- reassign which takes a task id, a list of actors serialized in string and a comment
- delegate which takes a task id, a list of actors and a comment
- delegate which takes a task id, a list of actors serialized in string and a comment
- complete which takes a task id, an action and a task completion request object

Let's get our task manager:
```java
TaskManager taskManager = client.taskManager();
```

Assuming our current user is Administrator, let's retrieve its tasks for our workflow:
```java
Tasks tasks = taskManager.fetchTasks("Administrator", serialWorkflowInstance.getId(),
                                     serialWorkflowInstance.getWorkflowModelName());
Task task = tasks.getEntry(0);
String name = task.getName(); // equals to wf.serialDocumentReview.chooseParticipants
List<String> actors = task.getActors(); // contains Administrator
List<String> targetDocIds = task.getTargetDocumentIds(); // contains note id
TaskInfo taskInfo = task.getTaskInfo();
List<TaskInfoItem> taskActions = taskInfo.getTaskActions(); // contains two actions
String actionName = taskActions.get(0).getName(); // equals to cancel or start_review
```

Let's start the review now:
```java
TaskCompletionRequest taskCompletionRequest = new TaskCompletionRequest();
Map<String, Object> variables = new HashMap<>();
variables.put("comment", "Please review");
variables.put("participants", Collections.singletonList("user:Administrator"));
taskCompletionRequest.setVariables(variables);
task = nuxeoClient.taskManager().complete(task.getId(), "start_review", taskCompletionRequest);
```

Let's approve the review:
```java
TaskCompletionRequest taskCompletionRequest = new TaskCompletionRequest();
taskCompletionRequest.setComment("Looks good!");
task = nuxeoClient.taskManager().complete(task.getId(), "approve", taskCompletionRequest);
```

Now we can end the workflow by validating it:
```java
TaskCompletionRequest taskCompletionRequest = new TaskCompletionRequest();
taskCompletionRequest.setComment("Looks good!");
task = nuxeoClient.taskManager().complete(task.getId(), "validate", taskCompletionRequest);
```

## Delete - Cancel Workflow

At each step we can cancel the workflow:
```java
repository.cancelWorkflowInstance(serialWorkflowInstance.getId());
```
