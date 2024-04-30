# Conductor OSS Javascript/Typescript SDK

Javascript SDK for working with https://github.com/conductor-oss/conductor.

[Conductor](https://www.conductor-oss.org/) is the leading open-source orchestration platform allowing developers to build highly scalable distributed applications.

Check out the official documentation for [Conductor](https://orkes.io/content).

## ⭐ Conductor OSS

Show support for the Conductor OSS.  Please help spread the awareness by starring Conductor repo.

[![GitHub stars](https://img.shields.io/github/stars/conductor-oss/conductor.svg?style=social&label=Star&maxAge=)](https://GitHub.com/conductor-oss/conductor/)


## Content

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Install Conductor Javascript SDK](#install-conductor-javascript-sdk)
  - [Get Conductor Javascript SDK](#get-conductor-javascript-sdk)
    - [NPM](#npm)
    - [YARN](#yarn)
- [Hello World Application Using Conductor](#hello-world-application-using-conductor)
  - [Step 1: Create Workflow](#step-1-create-workflow)
    - [Creating Workflows by Code](#creating-workflows-by-code)
    - [(Alternatively) Creating Workflows in JSON](#alternatively-creating-workflows-in-json)
  - [Step 2: Write Task Worker](#step-2-write-task-worker)
  - [Step 3: Write *Hello* World Application](#step-3-write-hello-world-application)
- [Running Workflows on Conductor Standalone (Installed Locally)](#running-workflows-on-conductor-standalone-installed-locally)
  - [Setup Environment Variable](#setup-environment-variable)
  - [Start Conductor Server](#start-conductor-server)
  - [Execute Hello World Application](#execute-hello-world-application)
- [Running Workflows on Orkes Conductor](#running-workflows-on-orkes-conductor)
- [Learn More about Conductor Javascript/Typescript SDK](#learn-more-about-conductor-javascripttypescript-sdk)
  - [Create and Run Conductor Workers](#create-and-run-conductor-workers)
  - [Create Conductor Workflows](#create-conductor-workflows)
  - [Using Conductor in your Application](#using-conductor-in-your-application)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

   
## Install Conductor Javascript SDK

### Get Conductor Javascript SDK

#### NPM

For NPM based projects, get the package from:

```shell
npm i @io-orkes/conductor-javascript
```

#### YARN

For yarn based projects, get the package from:

```shell
yarn add @io-orkes/conductor-javascript
```

## Hello World Application Using Conductor

In this section, we will create a simple "Hello World" application that executes a "greetings" workflow managed by Conductor.

### Step 1: Create Workflow

#### Creating Workflows by Code

Create [workflow/greetings_workflow.js] with the following:

```javascript
const {
  simpleTask,
  workflow
} = require("@io-orkes/conductor-javascript")

async function createAndRegisterWorkflow(client, workflow_name) {
  const greetingsTask = simpleTask("greetings", "greetings", {
    name: "${workflow.input.name}",
  });
  const wf = workflow(workflow_name, [
    greetingsTask
  ]);
  wf.inputParameters = ['name']
  client.metadataResource.create(wf, true);
  return wf;
}

module.exports = {
  createAndRegisterWorkflow: createAndRegisterWorkflow
}
```

#### (Alternatively) Creating Workflows in JSON

Create **greetings_workflow.json** with the following:

```json
{
  "name": "greetings",
  "description": "Sample greetings workflow",
  "version": 1,
  "tasks": [
    {
      "name": "greet",
      "taskReferenceName": "greet_ref",
      "type": "SIMPLE",
      "inputParameters": {
        "name": "${workflow.input.name}"
      }
    }
  ],
  "timeoutPolicy": "TIME_OUT_WF",
  "timeoutSeconds": 60
}
```

Workflows must be registered to the Conductor server. Use the API to register the greetings workflow from the JSON file above:

```shell
To Do
```

> [!note]
> To use the Conductor API, the Conductor server must be up and running ([see Running over Conductor standalone (installed locally)](#running-over-conductor-standalone-installed-locally)).

### Step 2: Write Task Worker

Using Javascript, a worker represents a function with the `worker_task` decorator. Create [workers/greetings_worker.js] file as illustrated below:

>[!note]
> A single workflow can have task workers written in different languages and deployed anywhere, making your workflow polyglot and distributed!

```javascript
const greetings = () => {
  return {
    taskDefName: "greetings",
    execute: async ({ inputData }) => {
      const name = inputData?.name;
      return {
        outputData: {
          result: "Hello "+ name,
        },
        status: "COMPLETED",
      };
    },
  };
};

module.exports = {
  greetings: greetings
}
```

Now, we are ready to write our main application, which will execute our workflow.

### Step 3: Write *Hello* World Application

Let's add [helloworld.js] with a `main` method:

```javascript
const { createAndRegisterWorkflow} = require("./workflow/greetings_workflow")
const { greetings } = require("./worker/greetings_worker")
const { orkesConductorClient,  WorkflowExecutor, TaskManager } = require("@io-orkes/conductor-javascript");
const workflow_name = "Greetings_Workflow";
const uuid = require('uuid');

const serverSettings = {
    keyId: process.env.KEY,
    keySecret: process.env.SECRET,
    serverUrl: process.env.CONDUCTOR_SERVER_URL,
};

async function main() {
    const clientPromise = await orkesConductorClient(serverSettings);
    //This is a one-time process, this line can be commented after
    const wf = await createAndRegisterWorkflow(clientPromise, workflow_name);
    const taskManager = new TaskManager(clientPromise, [
        greetings()
    ], { logger: console, options: { concurrency: 5, pollInterval: 100 } });
    taskManager.startPolling();
    await run_workflow(clientPromise);
    await taskManager.stopPolling();
    process.exit(0);
}

async function run_workflow(client) {
    const workflowExecutor = new WorkflowExecutor(client);
    const workflowRun = await workflowExecutor.executeWorkflow(
        {
            name: workflow_name,
            version: 1,
            input: {
                name: "Orkes",
            },
        },
        workflow_name,
        1,
        uuid.v4(),
    )
    if (workflowRun.status != 'COMPLETED') {
        throw new Error(`workflow not completed, workflowId: ${workflowRun.workflowId}`);
    }
    console.log(`Workflow Output: ${workflowRun.output}`);
    const baseUrl = `${process.env.CONDUCTOR_SERVER_URL}`;
    const execUrl = `${baseUrl.substring(0, baseUrl.length - 4)}/execution/${workflowRun.workflowId}`;
    console.log(execUrl);
}

main()
```

## Running Workflows on Conductor Standalone (Installed Locally)

### Setup Environment Variable

Set the following environment variable to point the SDK to the Conductor Server API endpoint:

```shell
export CONDUCTOR_SERVER_URL=http://localhost:8080/api
```

### Start Conductor Server

To start the Conductor server in a standalone mode from a Docker image, type the command below:

```shell
docker run --init -p 8080:8080 -p 5000:5000 conductoross/conductor-standalone:3.15.0
```

To ensure the server has started successfully, open Conductor UI on http://localhost:5000.

### Execute Hello World Application

To run the application, type the following command:

```javascript
node src/helloworld.js
```

Now, the workflow is executed, and its execution status can be viewed from Conductor UI (http://localhost:5000).

Navigate to the **Executions** tab to view the workflow execution.

## Running Workflows on Orkes Conductor

For running the workflow in Orkes Conductor,

- Update the Conductor server URL to your cluster name.

```shell
export CONDUCTOR_SERVER_URL=https://[cluster-name].orkesconductor.io/api
```

- If you want to run the workflow on the Orkes Conductor Playground, set the Conductor Server variable as follows:

```shell
export CONDUCTOR_SERVER_URL=https://play.orkes.io/api
```

- Orkes Conductor requires authentication. [Obtain the key and secret from the Conductor server](https://orkes.io/content/how-to-videos/access-key-and-secret) and set the following environment variables.

```shell
export CONDUCTOR_AUTH_KEY=your_key
export CONDUCTOR_AUTH_SECRET=your_key_secret
```

Run the application and view the execution status from Conductor's UI Console.

>[!note]
> That's it - you just created and executed your first distributed Javascript app!

## Learn More about Conductor Javascript/Typescript SDK

There are three main ways you can use Conductor when building durable, resilient, distributed applications.

1. Write service workers that implement business logic to accomplish a specific goal - such as initiating payment transfer, getting user information from the database, etc.
2. Create Conductor workflows that implement application state - A typical workflow implements the saga pattern.
3. Use Conductor SDK and APIs to manage workflows from your application.

### [Create and Run Conductor Workers](workers_sdk.md)

### [Create Conductor Workflows](workflow_sdk.md)

### [Using Conductor in your Application](conductor_apps.md)
