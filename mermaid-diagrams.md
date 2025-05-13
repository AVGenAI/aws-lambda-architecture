# AWS Lambda Architecture Diagrams

## Core Lambda Architecture

```mermaid
graph TD
    subgraph "Event Sources"
        API[API Gateway]
        S3[Amazon S3]
        DDB[DynamoDB]
        SNS[Amazon SNS]
        SQS[Amazon SQS]
        CW[CloudWatch Events]
        KDS[Kinesis Data Streams]
        EB[EventBridge]
    end

    subgraph "AWS Lambda"
        LS[Lambda Service]
        EE[Execution Environment]
        LS --> EE
    end

    subgraph "Your Function"
        Code[Your Code]
        Deps[Dependencies]
    end

    subgraph "AWS Services"
        DDBtarget[DynamoDB]
        S3target[S3]
        SNStarget[SNS]
        SQStarget[SQS]
    end

    subgraph "Monitoring"
        CWL[CloudWatch Logs]
        XR[X-Ray Tracing]
    end

    API --> LS
    S3 --> LS
    DDB --> LS
    SNS --> LS
    SQS --> LS
    CW --> LS
    KDS --> LS
    EB --> LS

    EE --> Code
    Code --> Deps
    
    Code --> DDBtarget
    Code --> S3target
    Code --> SNStarget
    Code --> SQStarget

    EE --> CWL
    EE --> XR
```

## VPC Integration Architecture

```mermaid
graph LR
    subgraph "VPC"
        subgraph "Lambda Function"
            LF[Function Code]
            ENI[Elastic Network Interface]
            LF --- ENI
        end
        
        subgraph "Private Resources"
            RDS[Amazon RDS]
            EC2[EC2 Instances]
            ES[ElastiCache]
        end
        
        ENI --> RDS
        ENI --> EC2
        ENI --> ES
    end
```

## API Backend Pattern

```mermaid
sequenceDiagram
    participant Client
    participant API as API Gateway
    participant Lambda
    participant DDB as DynamoDB
    
    Client->>API: HTTP Request
    API->>Lambda: Invoke Function
    Lambda->>DDB: Query/Update Data
    DDB-->>Lambda: Return Data
    Lambda-->>API: Response
    API-->>Client: HTTP Response
```

## Event Processing Pattern

```mermaid
graph LR
    S3[S3 Bucket] -->|Object Created Event| Lambda
    SQS[SQS Queue] -->|Message| Lambda
    SNS[SNS Topic] -->|Notification| Lambda
    
    Lambda -->|Process Data| Target[Target Service]
```

## Stream Processing Pattern

```mermaid
graph LR
    KDS[Kinesis Data Stream] -->|Records| Lambda
    DDBS[DynamoDB Stream] -->|Change Data| Lambda
    
    Lambda -->|Process Records| ES[Elasticsearch]
    Lambda -->|Store Data| S3[S3 Bucket]
    Lambda -->|Analytics| KDF[Kinesis Data Firehose]
```

## Lambda Scaling and Concurrency

```mermaid
graph TD
    Events[Incoming Events] --> Concurrency[Lambda Concurrency Manager]
    
    Concurrency --> E1[Execution Environment 1]
    Concurrency --> E2[Execution Environment 2]
    Concurrency --> E3[Execution Environment 3]
    Concurrency --> EN[Execution Environment N]
    
    subgraph "Account Concurrency Limits"
        RC[Reserved Concurrency]
        PC[Provisioned Concurrency]
        UC[Unreserved Concurrency]
    end
```

## Lambda Execution Environment Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Init
    Init --> Invoke: First Invocation
    Invoke --> WaitForNext: Execution Completed
    WaitForNext --> Invoke: New Invocation
    WaitForNext --> Shutdown: Idle Timeout
    Shutdown --> [*]
    
    note right of Init
        Cold Start - Download Code,
        Initialize Runtime, Run Init Code
    end note
    
    note right of WaitForNext
        Warm Environment - Faster Startup
    end note
```