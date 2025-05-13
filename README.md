# AWS Lambda Architecture Diagram

## Overview

AWS Lambda is a serverless compute service that lets you run code without provisioning or managing servers. Below is a comprehensive architecture diagram of how AWS Lambda works within the AWS ecosystem.

```
                                 +---------------------+
                                 |                     |
                                 |   Event Sources     |
                                 |                     |
                                 +----------+----------+
                                            |
                                            ▼
+------------------------------------------------------------------+
|                                                                  |
|  +----------------+      +----------------+      +-------------+ |
|  |                |      |                |      |             | |
|  | API Gateway    +----->+ AWS Lambda     +----->+ Your Code   | |
|  | S3             |      | Service        |      | Function    | |
|  | DynamoDB       |      |                |      |             | |
|  | SNS/SQS        |      | +------------+|      +------+------+ |
|  | CloudWatch     |      | |Execution   ||             |        |
|  | Kinesis        |      | |Environment ||             |        |
|  | EventBridge    |      | +------------+|             |        |
|  |                |      |                |             |        |
|  +----------------+      +----------------+             ▼        |
|                                                                  |
|                                                  +-----------+  |
|                                                  | AWS       |  |
|                                                  | Services  |  |
|                                                  | (DynamoDB,|  |
|                                                  |  S3, etc) |  |
|                                                  +-----------+  |
|                                                                  |
+------------------------------------------------------------------+
                     |                            |
                     ▼                            ▼
           +-----------------+           +----------------+
           |                 |           |                |
           | CloudWatch Logs |           | X-Ray Tracing |
           |                 |           |                |
           +-----------------+           +----------------+
```

## Components of AWS Lambda Architecture

### 1. Event Sources

Lambda functions can be triggered by various event sources:

- **AWS Services**: S3, DynamoDB, Kinesis, SNS, SQS, API Gateway, etc.
- **Custom Applications**: Through SDK or API calls
- **HTTP Endpoints**: Via API Gateway or Application Load Balancer
- **Scheduled Events**: Using CloudWatch Events/EventBridge

### 2. Lambda Service

The core AWS Lambda service that manages:

- **Function Deployment**: Packaging and distributing your code
- **Scaling**: Automatically scaling based on incoming requests
- **Invocation**: Processing triggers and invoking your function
- **Execution Environment**: Isolated containers to run your code
- **Resource Management**: CPU, memory, and network allocations

### 3. Execution Environment

The containerized environment where your code runs:

- **Language Runtime**: Supports Node.js, Python, Java, Go, .NET, Ruby
- **Memory Allocation**: 128MB to 10GB
- **Execution Duration**: Up to 15 minutes
- **Lifecycle**: Init phase and invoke phase
- **Temporary Storage**: /tmp directory with up to 10GB

### 4. Function Code

Your business logic that processes events:

- **Handler Function**: Entry point for execution
- **Dependencies**: Libraries and modules packaged with your code
- **Context Object**: Information about the invocation, function, and execution environment
- **Event Object**: Input data from the event source

### 5. Monitoring and Logging

Tools to track performance and troubleshoot issues:

- **CloudWatch Logs**: Capture function logs
- **CloudWatch Metrics**: Track invocations, duration, errors, etc.
- **X-Ray**: Trace requests across services
- **CloudWatch Alarms**: Alert on function metrics

## Key Architectural Features

### Stateless Execution

Lambda functions are stateless by design. Any state that needs to be persisted must be stored in external services like S3 or DynamoDB.

### Cold Starts vs. Warm Starts

- **Cold Start**: When a new execution environment is created (higher latency)
- **Warm Start**: When an existing environment is reused (lower latency)

### Concurrency and Scaling

- **Concurrency**: Number of function instances running simultaneously
- **Reserved Concurrency**: Guarantees a certain level of concurrency
- **Provisioned Concurrency**: Pre-initialized environments to reduce cold starts

### VPC Integration

Lambda functions can be configured to access resources within a VPC:

```
+------------------------------------------+
|                   VPC                    |
|                                          |
|  +-------------+      +---------------+  |
|  |             |      |               |  |
|  | Lambda      +----->+ Private       |  |
|  | Function    |      | Resources     |  |
|  |             |      | (RDS, EC2)    |  |
|  +-------------+      +---------------+  |
|                                          |
+------------------------------------------+
```

### Security

- **IAM Roles**: Control what your function can access
- **Resource Policies**: Control who can invoke your functions
- **Encryption**: In-transit and at-rest encryption of code and environment variables
- **Security Groups**: Control network traffic when in a VPC

## Common Architecture Patterns

### API Backend

```
   +-----------+      +--------+      +---------+
   |           |      |        |      |         |
   | API       +----->+ Lambda +----->+ DynamoDB|
   | Gateway   |      |        |      |         |
   |           |      |        |      |         |
   +-----------+      +--------+      +---------+
```

### Event Processing

```
   +-----------+      +--------+      +---------+
   |           |      |        |      |         |
   | S3 Event  +----->+ Lambda +----->+ S3      |
   | SNS/SQS   |      |        |      | DynamoDB|
   |           |      |        |      |         |
   +-----------+      +--------+      +---------+
```

### Stream Processing

```
   +-----------+      +--------+      +---------+
   |           |      |        |      |         |
   | Kinesis   +----->+ Lambda +----->+ Firehose|
   | DynamoDB  |      |        |      | Elastic |
   | Streams   |      |        |      | Search  |
   +-----------+      +--------+      +---------+
```

## Best Practices

1. **Function Size**: Keep functions small and focused on a single task
2. **Execution Time**: Design for short-lived executions
3. **Error Handling**: Implement proper error handling and retries
4. **Environment Variables**: Use for configuration and secrets
5. **Dead Letter Queues**: Capture failed invocations
6. **Layers**: Share common code across functions
7. **Least Privilege**: Grant minimal permissions needed

## Limitations and Considerations

- **Execution Duration**: Maximum of 15 minutes
- **Deployment Package Size**: Up to 50MB zipped, 250MB unzipped
- **Memory**: 128MB to 10GB (affects CPU allocation)
- **Concurrent Executions**: Default limit of 1,000 per region
- **Cold Start Latency**: Particularly impacts functions in VPCs