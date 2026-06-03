# Azure Functions - Complete Guide

## What is Azure Functions?

Azure Functions is a serverless compute service provided by Microsoft Azure that allows developers to run code without managing servers or infrastructure.

The developer focuses only on writing business logic, while Azure automatically handles:

* Server provisioning
* Scaling
* Availability
* Monitoring
* Patching
* Infrastructure management

### Traditional Approach

```
User Request
      ↓
Virtual Machine
      ↓
Application
      ↓
Database
```

Problems:

* VM runs 24/7
* Higher cost
* Manual scaling
* Infrastructure maintenance

### Serverless Approach

```
Event Occurs
      ↓
Azure Function Starts
      ↓
Execute Logic
      ↓
Azure Function Stops
```

Benefits:

* Pay only when code runs
* Automatic scaling
* No server management
* Faster development

---

# Why Azure Functions Came Into Existence

Many applications require code execution only when an event occurs.

Examples:

* File uploaded
* Order placed
* Database record inserted
* Queue message received
* HTTP request received

Running a VM continuously for these small tasks is inefficient.

Azure Functions solves this by executing code only when required.

---

# Core Concepts

## 1. Trigger

A trigger is an event that starts a function.

Without a trigger, a function cannot execute.

### Common Triggers

#### HTTP Trigger

```
Browser
   ↓
HTTP Request
   ↓
Azure Function
```

Use Cases:

* APIs
* Webhooks
* Backend services

---

#### Blob Trigger

```
File Upload
     ↓
Blob Storage
     ↓
Azure Function
```

Use Cases:

* Resume processing
* Image processing
* Document conversion

---

#### Queue Trigger

```
Queue Message
      ↓
Azure Function
```

Use Cases:

* Order processing
* Payment systems
* Background jobs

---

#### Timer Trigger

```
Every Day 9 AM
       ↓
Azure Function
```

Use Cases:

* Daily reports
* Cleanup jobs
* Scheduled tasks

---

#### Event Hub Trigger

```
IoT Devices
      ↓
Event Hub
      ↓
Azure Function
```

Use Cases:

* Telemetry processing
* Sensor analytics

---

# 2. Bindings

Bindings connect Azure Functions with Azure services.

They reduce boilerplate code.

## Input Binding

Azure automatically provides data.

Example:

```
Blob Storage
      ↓
myBlob
```

No storage SDK code required.

---

## Output Binding

Azure automatically writes data.

Example:

```
Function Output
       ↓
Blob Storage
```

No upload code required.

---

## Example

### Input Blob Binding

```javascript
module.exports = async function (context, myBlob) {
    context.log(myBlob.toString());
}
```

Azure automatically reads the blob.

---

### Output Blob Binding

```javascript
context.bindings.myOutputBlob = myBlob.toString();
```

Azure automatically writes the blob.

---

# Azure Functions Hosting Plans

## Consumption Plan

Characteristics:

* Serverless
* Pay per execution
* Auto scale
* Scale to zero

Best For:

* Learning
* Demos
* Event-driven workloads

---

## Premium Plan

Characteristics:

* Pre-warmed instances
* Reduced cold starts
* VNet support

Best For:

* Production applications

---

## App Service Plan

Characteristics:

* Dedicated resources
* Predictable performance

Best For:

* Existing App Service environments

---

# Scaling

Azure automatically scales functions.

Example:

```
1 Upload
  ↓
1 Function Instance

100 Uploads
  ↓
100 Function Instances
```

No manual scaling required.

---

# Azure Function Execution Flow

```
Event
   ↓
Trigger
   ↓
Function Runtime
   ↓
Execute Code
   ↓
Output Binding
   ↓
Result
```

---

# Durable Functions

Durable Functions allow stateful workflows.

## Normal Function

```
Upload
   ↓
Process
   ↓
Finish
```

---

## Durable Function

```
Step 1
   ↓
Step 2
   ↓
Wait
   ↓
Step 3
   ↓
Finish
```

Use Cases:

* Order workflows
* Approval processes
* Multi-step business processes

---

# Application Insights

Application Insights provides:

* Monitoring
* Logging
* Metrics
* Failure tracking
* Performance analysis

Example Logs:

```
Resume Uploaded
Metadata Created
Function Completed
```

Benefits:

* Observability
* Troubleshooting
* Performance monitoring

---

# Real-Time Use Cases

## Resume Processing

```
Resume Upload
      ↓
Blob Trigger
      ↓
Extract Metadata
      ↓
Store Result
```

---

## Image Processing

```
Image Upload
      ↓
Blob Trigger
      ↓
Resize Image
      ↓
Create Thumbnail
```

---

## Banking Notifications

```
Transaction
      ↓
Function
      ↓
SMS
      ↓
Email
```

---

## Order Processing

```
Order Created
      ↓
Function
      ↓
Inventory Update
      ↓
Invoice Generation
```

---

# Demonstration Project

## Smart Resume Processing System

### Architecture

```
User Uploads Resume
          ↓
Blob Storage (resumes)
          ↓
Blob Trigger Azure Function
          ↓
Extract Metadata
          ↓
Blob Storage (processed)
          ↓
Application Insights
```

---

# Azure Resources Required

## Resource Group

```
rg-smart-resume
```

Purpose:

Logical container for all resources.

---

## Storage Account

Purpose:

Store uploaded resumes and processed metadata.

Containers:

```
resumes
processed
```

---

## Function App

Purpose:

Runs serverless code.

Hosting:

```
Consumption Plan
```

Runtime:

```
Node.js
```

---

## Application Insights

Purpose:

Monitoring and logging.

---

# Demonstration Steps

## Step 1

Open Storage Account.

Show:

```
resumes
processed
```

---

## Step 2

Open Function App.

Show:

```
processResumeBlob
```

Blob Trigger.

---

## Step 3

Upload:

```
resume.pdf
```

to:

```
resumes
```

container.

---

## Step 4

Explain:

```
Upload
   ↓
Blob Trigger Fires
   ↓
Function Executes
```

---

## Step 5

Open Function Monitor.

Show:

```
Succeeded
```

execution.

---

## Step 6

Open Application Insights.

Run query:

```kusto
traces
| order by timestamp desc
```

Show logs.

---

# Interview Questions

## What is Azure Functions?

Azure Functions is a serverless compute service that executes code in response to events without requiring infrastructure management.

---

## What is a Trigger?

A trigger is an event that starts a function execution.

Examples:

* HTTP Trigger
* Blob Trigger
* Queue Trigger
* Timer Trigger

---

## What is a Binding?

Bindings connect Azure Functions with Azure services and reduce boilerplate code.

Types:

* Input Binding
* Output Binding

---

## What is a Blob Trigger?

A Blob Trigger executes a function automatically when a file is uploaded or modified in Azure Blob Storage.

---

## What is Serverless Computing?

Serverless computing allows developers to run code without managing servers while paying only for actual execution.

---

## What is Application Insights?

Application Insights provides monitoring, telemetry, logging, diagnostics, and performance tracking for applications.

---

# Conclusion

Azure Functions enables event-driven, serverless application development. It automatically scales, reduces infrastructure management, integrates with Azure services through triggers and bindings, and is widely used for file processing, automation, notifications, APIs, and workflow orchestration.
