# Conductor Workflows

Workflow can be defined as the collection of tasks and operators that specify the order and execution of the defined tasks. This orchestration occurs in a hybrid ecosystem that encircles serverless functions, microservices, and monolithic applications.

This section will dive deeper into creating and executing Conductor workflows using Javascript/Typescript SDK.

[![GitHub stars](https://img.shields.io/github/stars/conductor-oss/conductor.svg?style=social&label=Star&maxAge=)](https://GitHub.com/conductor-oss/conductor/)

## Content

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Creating Workflows](#creating-workflows)
  - [Execute Dynamic Workflows Using Code](#execute-dynamic-workflows-using-code)
  - [Kitchen-Sink Workflow](#kitchen-sink-workflow)
- [Executing Workflows](#executing-workflows)
  - [Execute Workflow Asynchronously](#execute-workflow-asynchronously)
  - [Execute Workflow Synchronously](#execute-workflow-synchronously)
- [Managing Workflow Executions](#managing-workflow-executions)
  - [Get Execution Status](#get-execution-status)
  - [Update Workflow State Variables](#update-workflow-state-variables)
  - [Terminate Running Workflows](#terminate-running-workflows)
  - [Retry Failed Workflows](#retry-failed-workflows)
  - [Restart Workflows](#restart-workflows)
  - [Rerun Workflow from a Specific Task](#rerun-workflow-from-a-specific-task)
  - [Pause Running Workflow](#pause-running-workflow)
  - [Resume Paused Workflow](#resume-paused-workflow)
- [Searching for Workflows](#searching-for-workflows)
- [Handling Failures, Retries and Rate Limits](#handling-failures-retries-and-rate-limits)
  - [Retries](#retries)
  - [Rate Limits](#rate-limits)
    - [Task Registration](#task-registration)
    - [Update Task Definition:](#update-task-definition)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Creating Workflows

Conductor lets you create the workflows using either Javascript or JSON as the configuration.

Using Javascript as code to define and execute workflows lets you build extremely powerful, dynamic workflows and run them on Conductor.

When the workflows are relatively static, they can be designed using the Orkes UI (available when using Orkes Conductor) and APIs or SDKs to register and run the workflows.

Both the code and configuration approaches are equally powerful and similar in nature to how you treat Infrastructure as Code.

### Execute Dynamic Workflows Using Code

For cases where the workflows cannot be created statically ahead of time, Conductor is a powerful dynamic workflow execution platform that lets you create very complex workflows in code and execute them. It is useful when the workflow is unique for each execution.

```javascript
To Do
```

See [dynamic_workflow.js] for a fully functional example.

### Kitchen-Sink Workflow

For a more complex workflow example with all the supported features, see [kitchensink.js].

## Executing Workflows

The [WorkflowClient] interface provides all the APIs required to work with workflow executions.

```javascript
To Do
```

### Execute Workflow Asynchronously

Useful when workflows are long-running.

```javascript
To Do
```

### Execute Workflow Synchronously

Applicable when workflows complete very quickly - usually under 20-30 seconds.

```javascript
To Do 
```

## Managing Workflow Executions

>[!note]
>See [workflow_ops.js] for a fully working application that demonstrates working with the workflow executions and sending signals to the workflow to manage its state.

Workflows represent the application state. With Conductor, you can query the workflow execution state anytime during its lifecycle. You can also send signals to the workflow that determines the outcome of the workflow state.

[WorkflowClient] is the client interface used to manage workflow executions.

```javascript
To Do
```

### Get Execution Status

The following method lets you query the status of the workflow execution given the id. When the `include_tasks` is set, the response also includes all the completed and in-progress tasks.

```javascript
To Do
```

### Update Workflow State Variables

Variables inside a workflow are the equivalent of global variables in a program.

```javascript
To Do
```

### Terminate Running Workflows

Used to terminate a running workflow. Any pending tasks are canceled, and no further work is scheduled for this workflow upon termination. A failure workflow will be triggered but can be avoided if  `trigger_failure_workflow` is set to False.

```javascript
To Do
```

### Retry Failed Workflows

If the workflow has failed due to one of the task failures after exhausting the retries for the task, the workflow can still be resumed by calling the retry.

```javascript
To Do
```

When a sub-workflow inside a workflow has failed, there are two options:

1. Re-trigger the sub-workflow from the start (Default behavior).
2. Resume the sub-workflow from the failed task (set  `resume_subworkflow_tasks` to True).

### Restart Workflows

A workflow in the terminal state (COMPLETED, TERMINATED, FAILED) can be restarted from the beginning. Useful when retrying from the last failed task is insufficient, and the whole workflow must be started again.

```javascript
To Do
```

### Rerun Workflow from a Specific Task

In the cases where a workflow needs to be restarted from a specific task rather than from the beginning, rerun provides that option. When issuing the rerun command to the workflow, you can specify the task ID from where the workflow should be restarted (as opposed to from the beginning), and optionally, the workflow's input can also be changed.

```javascript
To Do
```

>[!tip]
>Rerun is one of the most powerful features Conductor has, giving you unparalleled control over the workflow restart.

### Pause Running Workflow

A running workflow can be put to a PAUSED status. A paused workflow lets the currently running tasks complete but does not schedule any new tasks until resumed.

```javascript
To Do
```

### Resume Paused Workflow

Resume operation resumes the currently paused workflow, immediately evaluating its state and scheduling the next set of tasks.

```javascript
To Do
```

## Searching for Workflows

Workflow executions are retained until removed from the Conductor. This gives complete visibility into all the executions an application has - regardless of the number of executions. Conductor has a powerful search API that allows you to search for workflow executions.

```javascript
To Do
```

- **free_text**: Free text search to look for specific words in the workflow and task input/output.
- **query**: SQL-like query to search against specific fields in the workflow.

Here are the supported fields for **query**:

| Field | Description |
| ----- | ----------- |
| status	| The status of the workflow. |
| correlationId	| The ID to correlate the workflow execution to other executions. |
| workflowType | The name of the workflow. |
| version	| The version of the workflow. |
| startTime | The start time of the workflow is in milliseconds. |

## Handling Failures, Retries and Rate Limits

Conductor lets you embrace failures rather than worry about the complexities introduced in the system to handle failures.

All the aspects of handling failures, retries, rate limits, etc., are driven by the configuration that can be updated in real time without re-deploying your application.

### Retries

Each task in the Conductor workflow can be configured to handle failures with retries, along with the retry policy (linear, fixed, exponential backoff) and maximum number of retry attempts allowed.

See [Error Handling](https://orkes.io/content/error-handling) for more details.

### Rate Limits

What happens when a task is operating on a critical resource that can only handle a few requests at a time? Tasks can be configured to have a fixed concurrency (X request at a time) or a rate (Y tasks/time window).

#### Task Registration

```javascript
To Do
```

```json
{
  "name": "task_with_retries",
  
  "retryCount": 3,
  "retryLogic": "LINEAR_BACKOFF",
  "retryDelaySeconds": 1,
  "backoffScaleFactor": 1,
  
  "timeoutSeconds": 120,
  "responseTimeoutSeconds": 60,
  "pollTimeoutSeconds": 60,
  "timeoutPolicy": "TIME_OUT_WF",
  
  "concurrentExecLimit": 3,
  
  "rateLimitPerFrequency": 0,
  "rateLimitFrequencyInSeconds": 1
}
```

#### Update Task Definition:

```shell
POST /api/metadata/taskdef -d @task_def.json
```

See [task_configure.js] for a detailed working app.