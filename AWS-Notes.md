****
# AWS Developer

Table Of Content
- [AWS Developer](#aws-developer)
  - [Dynamo DB](#dynamo-db)
    - [Read Capacity Unit (RCU)](#read-capacity-unit-rcu)
    - [Options to backup the DB data to S3](#options-to-backup-the-db-data-to-s3)
  - [AWS RDS](#aws-rds)
    - [Replicas and backup](#replicas-and-backup)
  - [CloudFront (CDN):](#cloudfront-cdn)
  - [Lambda Function:](#lambda-function)
    - [There are 2 types of concurrency.](#there-are-2-types-of-concurrency)
    - [Triva](#triva)
  - [Elastic Beanstalk](#elastic-beanstalk)
    - [You provide: Application Code](#you-provide-application-code)
  - [Simple Queue Service (SQS)](#simple-queue-service-sqs)
    - [Extended Client](#extended-client)
  - [AWS Serverless Application Model (AWS SAM),](#aws-serverless-application-model-aws-sam)
  - [Amazon Kinesis](#amazon-kinesis)
    - [Use Cases](#use-cases)
    - [Shards](#shards)
    - [Kinesis Agent](#kinesis-agent)
    - [Data FireHouse](#data-firehouse)
  - [AWS Step Functions](#aws-step-functions)
  - [Amazon Elastic Container Service (Amazon ECS)](#amazon-elastic-container-service-amazon-ecs)
    - [Triva](#triva-1)
  - [Configure CloudWatch](#configure-cloudwatch)
    - [Logs](#logs)
    - [Custom Metrics](#custom-metrics)
    - [Use Cases](#use-cases-1)
  - [Load Balancer](#load-balancer)
    - [Stickiness Enabled](#stickiness-enabled)
  - [Cognito User Pool](#cognito-user-pool)
  - [Cognito Identity Pool](#cognito-identity-pool)
      - [How it works](#how-it-works)
      - [Connection between User and Identity Pool:](#connection-between-user-and-identity-pool)
    - [Triva:](#triva-2)
  - [AWS AppSync](#aws-appsync)
  - [AWS CloudFormation](#aws-cloudformation)
  - [Amazon EBS (Elastic Block Stroage)](#amazon-ebs-elastic-block-stroage)
  - [AWS S3](#aws-s3)
    - [Controlling Access to S3](#controlling-access-to-s3)
    - [Triva](#triva-3)
  - [AWS KMS (Key Management Service)](#aws-kms-key-management-service)
    - [Envelope Encryption](#envelope-encryption)
    - [Triva](#triva-4)
      - [Simple Architecture](#simple-architecture)
  - [AWS CodeBuild](#aws-codebuild)
  - [AWS CodePipeline](#aws-codepipeline)
  - [AWS CodeCommit](#aws-codecommit)
  - [AWS CodeDeploy](#aws-codedeploy)
      - [AWS CodeDeploy Agent](#aws-codedeploy-agent)
  - [Auto-Scaling](#auto-scaling)
  - [Types of Deployment](#types-of-deployment)
    - [All At Once](#all-at-once)
    - [Rolling Deployment](#rolling-deployment)
    - [Rolling with Additional Batch](#rolling-with-additional-batch)
    - [Immutable Deployment](#immutable-deployment)
    - [Traffic Spiltting / Canary Deployment](#traffic-spiltting--canary-deployment)
      - [Quick Comparsion:](#quick-comparsion)
  - [Amazon Athena](#amazon-athena)
  - [AWS Redshift](#aws-redshift)
  - [IAM](#iam)
    - [Policies](#policies)
    - [Permissions](#permissions)
    - [Simple Analogy](#simple-analogy)
- [AWS Glue](#aws-glue)
  - [AWS Security Token Service (STS)](#aws-security-token-service-sts)


## Dynamo DB

* You can use DynamoDB transactions to make coordinated all-or-nothing changes to multiple items both within and across tables. Transactions provide atomicity, consistency, isolation, and durability (ACID) in DynamoDB.
* TransactWriteItem or TransactReadItem, for write or read transaction. Dynamo db makes 2 write or 2 read operations, one for prepare and one of commit the transaction.
* DynamoDB Streams is a feature that captures every change made to items in a DynamoDB table and stores those changes in a stream for up to 24 hours. It enables event-driven architectures by letting other services react to database changes in near real time.
* Dynamo DB has 2 types of backup, on-demand(when we choose) and point-in-time(continuous backup),**BOTH STORE TO S3, BUT S3 IS NOT ACCESS TO ANY USERS.**

### Read Capacity Unit (RCU)

Formula of RCUs = (Item Size / 4 KB) × Consistency Factor (1 for strongly and 0.5 for eventually).

- Rules:
  - 1 strongly consistent read per second for an item up to 4 KB.
  - 2 eventually consistent reads per second for an item up to 4 KB.
  - Transactional reads consume 2 RCUs per 4 KB.


- RCU and WCU allocations

  | Feature | Adds Capacity? | Needs Configuration? |
  |----------|---------------|----------------------|
  | Adaptive Capacity | ❌ No | ❌ No |
  | Auto Scaling | ✅ Yes | ✅ Yes |
  | On-Demand | ✅ Yes | ❌ No capacity settings |

- Easy memory trick:
  - Adaptive Capacity = Move capacity between partitions.
  - Auto Scaling = Increase/decrease provisioned RCUs/WCUs.
  - On-Demand = DynamoDB does everything automatically; no RCUs/WCUs to manage.

### Options to backup the DB data to S3
- Use AWS Data Pipeline to export your table to an S3 bucket in the account of your choice and download locally
  - This is the easiest method. This method is used when you want to make a one-time backup using the lowest amount of AWS resources possible. Data Pipeline uses Amazon EMR to create the backup, and the scripting is done for you. You don't have to learn Apache Hive or Apache Spark to accomplish this task.
- Use Hive with Amazon Elastic MapReduce (EMR) to export your data to an S3 bucket and download locally.
  - Use Hive to export data to an S3 bucket. Or, use the open-source emr-dynamodb-connector to manage your own custom backup method in Spark or Hive. These methods are the best practice to use if you're an active Amazon EMR user and are comfortable with Hive or Spark. These methods offer more control than the Data Pipeline method.
  - EMR is a service that provides support for all the pipeline creation frameworks, like Hive, Spark etc.
- Use **Glue** to copy your table to Amazon S3 and download locally
  - Use Glue to copy your table to Amazon S3. This is the best practice to use if you want automated, continuous backups that you can also use in another service, such as Amazon Athena.

**** 
## AWS RDS
- RDS POSTGRESQL and RDS MYSQL can be configured with IAM database Authentication.
-  IAM database authentication works with MySQL and PostgreSQL engines for Aurora as well as MySQL, MariaDB and RDS PostgreSQL engines for RDS.

### Replicas and backup
- Automated backup feature of the AWS RDS is a MULTI-AZ deployment that creates backup in SINGLE REGION.

****

## CloudFront (CDN):

- Signed URL:
    student -> buys a course -> Udemy (uses a private key and generate a URL valid till 1 hour) -> share it with student
    student -> cloudFront -> (Udemy has already uploaded the public key for the private key)
    -> cloudFront checks if the URL is valid using the public, if not valid return ERROR.

- Signed Cookies:
  - CloudFront Signed Cookies are similar to Signed URLs, but instead of putting the access credentials in the URL, CloudFront stores them in cookies sent by the user's browser.
  - They are used when you want to give access to multiple protected files without generating a separate signed URL for every file.

- Trusted Key Group: cloudFront support a key groups, where we can store N number of public-keys
- CloudFront Key Pair (legacy, root-user based): only able to store 2 active public-keys
- example: https://d123.cloudfront.net/movie.mp42?Expires=17800000003&Signature=ABCXYZ4&Key-Pair-Id=K123456 // used to determine the public in the key-group.

****

## Lambda Function:
- Memory == CPU == Network (if you want to increase a lambda function's CPU or Network, the you need to increase its Memory)
- Auto-scaling is possible, like increasing the Provisioned concurrency on the go.

- Due to a spike in traffic, when Lambda functions scale, this causes the portion of requests that are served by new instances to have higher latency than the rest. To enable your function to scale without fluctuations in latency, use provisioned concurrency. By allocating provisioned concurrency before an increase in invocations, you can ensure that all requests are served by initialized instances with very low latency.
- Execution Context: In AWS Lambda, the execution context is the runtime environment that Lambda creates to execute your function code. Think of it as a container (or execution environment) that contains:
  - Your function code.
  - Runtime (Java, Python, Node.js, etc.).
  - Allocated memory and CPU.
  - Temporary storage (/tmp).
  - Initialized variables and objects.
- Lamdba  Aliases:
  -  A Lambda alias is like a pointer to a specific Lambda function version. You can create one or more aliases for your AWS Lambda function.
  - An alias can only point to a function version, not to another alias.

### There are 2 types of concurrency.

- Reserved concurrency guarantees the maximum number of concurrent instances for the function. When a function has reserved concurrency, no other function can use that concurrency. There is no charge for configuring reserved concurrency for a function.

- Provisioned concurrency initializes a requested number of execution environments so that they are prepared to respond immediately to your function's invocations. Note that configuring provisioned concurrency incurs charges to your AWS account.

### Triva
- The total size of the env variables should not exceed 4 KB.

****

## Elastic Beanstalk
- When dealing with SQS/SNS events, and longer tasks, use **Dedicated Worker Enviroment**.

### You provide: Application Code

- Elastic Beanstalk manages: EC2, Auto Scaling, Load Balancer, Monitoring, Health Checks, Deployments.
- .ebextension folder (at the root of the project), inside this folder we can add all the config file with ".config".

****
## Simple Queue Service (SQS)

- Standard queue: offer maximum throughput, best-effort ordering, and at-least-once delivery.
- FIFO queues: are designed to guarantee that messages are processed exactly once, in the exact order that they are sent.
- Delay Queue: when sending a event to SQS, we can add a delay to it min of 0 and max of 15 mins, this will be in the delay queue.
- Visibility timeout: It is the time for which the event is not visible to other consumers, meaning when a event is being processed by a consumer, other consumers can not be able to see this event, min = 0, max = 12 hours, default = 30 sec.
- **in-flight messages**: received from a queue by a consumer, but not yet deleted from the queue. MAX of 120,000 in-flight messages allowed.
- "no limit": There are no message limits for storing in SQS.
- By default SQS uses SHORT-POLLING, With short polling, Amazon SQS sends the response right away, even if the query found no messages.
- **LONG POLLING**: Helps reduce the cost of AWS SQS by elimating the number of empty responses.

### Extended Client
- It helps in handling events, whose size is more than 256kb.
- Working:
  - producer uploads a 2 GB(max-limit) file / payload.
  - extended client than uploads this to S3.
  - S3 pointer / reference is sent to SQS.
  - consumer recevies the SQS message.
  - extended client, automatically retrieves the file / payload from the S3

****

## AWS Serverless Application Model (AWS SAM), 
(serverless-framework - tryFuse)

- It is a simplified layer on top of AWS CloudFormation. Instead of writing hundreds of lines of CloudFormation YAML, you can use SAM's shorter syntax to define resources such as: Lambda, EC2 and so no.

- sam deploy - Full deployment - Slow - Updates CloudFormation stack

- sam sync - Fast deployment - Updates only what's changed, Great during development. (acts like nodemon).

****
## Amazon Kinesis

- It is AWS's real-time data streaming service.
- Think of it as a system that can continuously collect, process, and analyze massive streams of data as they are generated, instead of waiting for batch uploads.
- Data Streams, incoming data is called streams, it can be of any type lets, stock-data, gaming or logs.
- partition key is a value that Kinesis uses to determine which shard a record will be written to.

### Use Cases
- For example, you have a billing application and an audit application that runs a few hours behind the billing application. By default, records of a stream are accessible for up to 24 hours from the time they are added to the stream. You can raise this limit to a maximum of 365 days. For the given use-case, Amazon Kinesis Data Streams can be configured to store data for up to 7 days and you can run the audit application up to 7 days behind the billing application.

### Shards
- A shard is the basic unit of throughput in Kinesis Data Streams.

```
Producer
   |
   v
Kinesis Stream
   |
   +--> Shard 1 ----\
   +--> Shard 2 -----+--> Consumer
   +--> Shard 3 ----/
```

### Kinesis Agent
- Kinesis Agent works with data producers.
- Lets say we are sending the EC2 logs to kinesis, so here I can install the kinesis at the EC2 instance, which keeps an eye on the log file and when there is update, it pushes that data to the kinesis.

### Data FireHouse
- Kinesis Data Firehose, we don't need to write applications or manage resources. we configure our data producers to send data to Kinesis Data Firehose, and it automatically delivers the data to the destination that you specified.
- With FireHouse, we can send data to 2 S3 bucket, or to S3 bucket and SQS or a Elasticsearch cluster or to redshift via S3-bucket.

****

## AWS Step Functions
- AWS Step Functions enables you to implement a business process as a series of steps that make up a workflow. The individual steps in the workflow can invoke a Lambda function or a container that has some business logic, update a database such as DynamoDB or publish a message to a queue once that step or the entire workflow completes execution.

- Activities in the step-functions are like third party API calls, When a Step Function reaches an activity task state, the workflow waits for an activity worker to poll for a task. For example, an activity worker can be an application running on an Amazon EC2 instance or an AWS Lambda function.
- Types of step functions:
  - Standard step functions (1 Year excution time): More suitable for long-running, durable, and auditable workflows.
  - Express step function (5 Mins): Used for workloads with high event rates and short durations. Express Workflows support event rates of more than 100,000 per second. DO NOT SUPPORT HUMAN APPROVAL STEP (.waitForTaskToken).

![alt text](<Screenshot 2026-07-20 143831.png>)

****

## Amazon Elastic Container Service (Amazon ECS)

- It is a highly scalable, fast, container management service that makes it easy to run, stop, and manage Docker containers on a cluster. You can host your cluster on a serverless infrastructure that is managed by Amazon ECS by launching your services or tasks using the Fargate launch type.

- For more control over your infrastructure, you can host your tasks on a cluster of Amazon Elastic Compute Cloud (Amazon EC2) instances that you manage by using the EC2 launch type. one EC2 instance can have multiple containers, and this are managed by ECS.  

- When you deploy your services using Amazon Elastic Container Service (Amazon ECS), you can use dynamic port mapping to support multiple tasks from a single service on the same container instance. Amazon ECS manages updates to your services by automatically registering and deregistering containers with your target group using the instance ID and port for each container.

### Triva
- When a container is TERMINATED while running, the container is still registered to the ECS, so it will be de-reigstered.
- When a conatiner is TERMINATED when it is in STOPPED state, the container will not be removed from ECS, because it is not registered, need to manually de-register it.

****
## Configure CloudWatch

### Logs

- CloudWatch Logs Insights enables you to interactively search and analyze your log data in Amazon CloudWatch Logs. You can perform queries to help you more efficiently and effectively respond to operational issues.

### Custom Metrics
-  We can publish your own metrics, known as custom metrics, to CloudWatch using the AWS CLI or an API.
- Each metric is one of the following:
  - Standard resolution, with data having a one-minute granularity.
  - High resolution, with data at a granularity of one second **[Real Time Monitorning]**.

### Use Cases
- We can create composite alerts over a custom-metrices, to watch for error-rate, latency over a period of 1 hour and send notification to SNS, SQS or SES and so on.

****

## Load Balancer

- To use an HTTPS listener, you must deploy at least one SSL/TLS server certificate on your load balancer. 

- Application LB works on the application layer (network layers: APS-TN-DP), with HTTP and HTTPS, it also supports the dynamic port mapping with the EC2 instance.
- Classic LB does not support dynamic port mapping.
- **X-Forwarded-For**, to get the client address in EC2 or lambda, LB attaches the client IP in the headers.X-Forwarded-For key.

### Stickiness Enabled
- Sticky sessions are a mechanism to route requests to the same target in a target group. This is useful for servers that maintain state information to provide a continuous experience to clients. To use sticky sessions, the clients must support cookies.
- Working:
  - When a load balancer first receives a request from a client, it routes the request to a target.
  - generates a cookie named AWSALB that encodes information about the selected target, encrypts the cookie, and includes the cookie in the response to the client.
  - The client should include the cookie that it receives in subsequent requests to the load balancer.
  - When the load balancer receives a request from a client that contains the cookie.
  - if sticky sessions are enabled for the target group and the request goes to the same target group, the load balancer detects the cookie and routes the request to the same target.


****

## Cognito User Pool

- Authorization, Sign-up, Sign-in.

- Built-in, customixable web UI to sign-in.

- Social sign-in with FB, Google, Apple sign-in with SAML.

- MFA, Phone and email verification.



****

## Cognito Identity Pool

- Used to obtain temporary access to AWS service.
- AWS Cognito Identity Pools can provide temporary AWS credentials to anonymous (guest/unauthenticated) users.

#### How it works

- A user visits your application without logging in.

- Cognito Identity Pool recognizes the user as an unauthenticated (guest) identity.

- Cognito assumes a special IAM role configured for guest users.

- AWS returns temporary credentials (Access Key, Secret Key, Session Token).

- The guest user can access only the AWS services/actions allowed by that IAM role.

#### Connection between User and Identity Pool:

- client A, sign-ups with the help of User-pool.

- User-pool returns client A, a valid tokens.

- client A uses this token and obtains a temporary access to AWS from Identity-pool.

### Triva:
- Identity Pools (Amazon Cognito Identity Pools) provide temporary AWS credentials, but they do so by using [AWS STS](#aws-security-token-service-sts) behind the scenes.

****

## AWS AppSync

- AWS AppSync is a managed GraphQL service designed for building real-time, collaborative applications with features like **offline data access, conflict resolution, and device synchronization**.
- With AppSync, mobile apps can continue to work **offline**, queue changes locally, and then sync seamlessly across users and devices when connectivity returns.
- It also integrates with Cognito for authentication and DynamoDB or Aurora for storage, making it ideal for the portal’s real-time group study and Q&A features.

****

## AWS CloudFormation 

-It gives developers and businesses an easy way to create a collection of related AWS and third-party resources and provision them in an orderly and predictable fashion.

How CloudFormation Works:
AWS CloudFormation gives developers and businesses an easy way to create a collection of related AWS and third-party resources and provision them in an orderly and predictable fashion.

**CloudFormation currently supports the following parameter types:**
```
String – A literal string
Number – An integer or float
List<Number> – An array of integers or floats
CommaDelimitedList – An array of literal strings that are separated by commas
AWS::EC2::KeyPair::KeyName – An Amazon EC2 key pair name
AWS::EC2::SecurityGroup::Id – A security group ID
AWS::EC2::Subnet::Id – A subnet ID
AWS::EC2::VPC::Id – A VPC ID
List<AWS::EC2::VPC::Id> – An array of VPC IDs
List<AWS::EC2::SecurityGroup::Id> – An array of security group IDs
List<AWS::EC2::Subnet::Id> – String – A literal string
Number – An integer or float
List<Number> – An array of integers or floats
CommaDelimitedList – An array of literal strings that are separated by commas
AWS::EC2::KeyPair::KeyName – An Amazon EC2 key pair name
AWS::EC2::SecurityGroup::Id – A security group ID
AWS::EC2::Subnet::Id – A subnet ID
AWS::EC2::VPC::Id – A VPC ID
List<AWS::EC2::VPC::Id> – An array of VPC IDs
List<AWS::EC2::SecurityGroup::Id> – An array of security group IDs
List<AWS::EC2::Subnet::Id> – An array of subnet IDs array of subnet IDs
```

How CloudFormation Works:
![alt text](image.png)

****

## Amazon EBS (Elastic Block Stroage)

- An Amazon EBS volume is a durable, block-level storage device that you can attach to your instances.
- After you attach a volume to an instance, you can use it as you would use a physical hard drive.
EBS volumes are flexible.
- When you create an EBS volume, it is automatically replicated within its Availability Zone to prevent data loss due to the failure of any single hardware component. **You can attach an EBS volume to an EC2 instance in the same Availability Zone**.
- Encrypt Rules:
  - Data at rest inside the volume
  - All data moving between the volume and the instance
  - All snapshots created from the volume
  - All volumes created from those snapshots

- **Provisioned IOPS** SSD is a high-performance EBS storage type designed for applications that need a guaranteed number of I/O operations per second (IOPS), such as databases.
  - GP2 IOPS = 3 * volumn_size.
  - GP3 IOPS = custom IOPS, regardless of the size (costier than GP2).
  - IO1 / IO2 = 50 * volumn_size, And IO1 / IO2 are dedicated SSD.

****
## AWS S3

- **pre-signed URLs** All objects by default are private, with the object owner having permission to access the objects. However, the object owner can optionally share objects with others by creating a pre-signed URL, using their own security credentials, to grant time-limited permission to download the objects.
- **CROS-Config**: For allowing cross-origin on the S3 objects (webhosting), upload a XML file, with allowed origin and allowed HTTP methods.
- **bucket-owner-full-control**, when passed while uploading the object to bucket, the object owner is now the bucket owner.

### Controlling Access to S3
- Identity and Access Mangement (IAM) policies.
- Bucket Policies.
- Access Control Policies.
- Query String Authentication (Signed URLS).

### Triva
1. Even in case of bucket being Public, objects are private, unless there is bucket-policy at a folder like,
```
{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/public/*"
}
```
Here all the objects in the public folder are public.

****
## AWS KMS (Key Management Service)

- AWS KMS is a managed service that helps you create, store, manage, and use encryption keys to protect your data in AWS services and your applications.

### Envelope Encryption

Envelope Encryption is a technique AWS KMS uses to efficiently encrypt large amounts of data. Instead of using the KMS key directly to encrypt a large file, AWS uses a two-key approach:
- A Data Key encrypts the actual data.
- The KMS Key (CMK) encrypts the Data Key.

Encryption Flow:
```
KMS Key
   |
   | Generates
   v
Encryted Data Key
   |
   | Encrypts
   v
100 GB File
```

Decryption Flow:
```
100 GB encryted file + encryted data key
   |
   | KMS -> encryted data key and provides a decrted key.
   v
Decryted Data Key
   |
   | Decrypts data key to decryt the file.
   v
Decryted 100 GB File
```

### Triva
- KMS = Key Store, only manges encryption keys MAX of 4kb.
- Secrets Manager = Password Store, Used to store ENV vars
- Parameter Store = Configuration Store, Used to different env, like "/dev/db/url", "/qa/db/url".

#### Simple Architecture
```
Developer
    |
    v
Git Push / Pull
    |
    v
AWS CodeCommit Repository
    |
    +--> AWS CodeBuild
    |
    +--> AWS CodeDeploy
    |
    +--> AWS CodePipeline
```

In Prod Example:

| Service | Purpose |
|----------|----------|
| CodeCommit | Source code repository |
| CodeBuild | Compile/Test |
| CodeDeploy | Deploy application |
| CodePipeline | Orchestrate entire CI/CD flow |

****

## AWS CodeBuild
- buildspec.yml file is required for the codeBuild.
- CodeBuild scales automatically.
- CodeBuild is responsible for:
  - Compiling code.
  - Running unit tests.
  - Running integration tests.
  - Creating build artifacts (JAR, WAR, Docker image, ZIP, etc.).

Example devloper push code:
```
Java Source Code
       |
       v
CodeBuild
       |
       +--> mvn clean install
       +--> Run tests
       +--> Create JAR

```

****
## AWS CodePipeline

CodePipeline coordinates all CI/CD stages.

- CodePipeline supports manual approvals in the pipeline process.

Pipeline stages:
- Source.
- Build.
- Test.
- Approval (optional).
- Deploy.

***CodePipeline itself does not build code.*** Instead it calls services such as:
- CodeBuild.
- CodeDeploy.
- Lambda.
- ECS.
- CloudFormation.

Example:
```
CodeCommit/GitHub
        |
        v
   CodePipeline
        |
   +----+----+
   |         |
Source      Build
              |
          CodeBuild
              |
          Deploy
              |
         CodeDeploy
```


****
## AWS CodeCommit

AWS CodeCommit is AWS's managed Git repository service.
Think of it as AWS's version of GitHub, GitLab, or Bitbucket, where you can store your source code and track changes using Git.


****
## AWS CodeDeploy

AWS CodeDeploy is an service, that manages all the deployment tasks.

#### AWS CodeDeploy Agent
- The CodeDeploy agent is a software package that, when installed and configured on an instance, makes it possible for that instance to be used in CodeDeploy deployments.
- The CodeDeploy agent archives revisions and log files on instances. The CodeDeploy agent cleans up these artifacts to conserve disk space.
- You can use the **max_revisions** option in the agent configuration file to specify the number of application revisions to the archive by entering any positive integer.

****
## Auto-Scaling

To maintain the same number of instances, Amazon EC2 Auto Scaling performs a periodic health check on running instances within an Auto Scaling group. When it finds that an instance is unhealthy, it terminates that instance and launches a new one. Amazon EC2 Auto Scaling creates a new scaling activity for terminating the unhealthy instance and then terminates it. Later, another scaling activity launches a new instance to replace the terminated instance.

****

## Types of Deployment

### All At Once
- The quickest deployment, All instances are updated simultaneously.
```
Before:
v1 v1 v1 v1

Deploy

After:
v2 v2 v2 v2
```

### Rolling Deployment
- Updates a few instances at a time.

```
Batch 1:
v2 v1 v1 v1

Batch 2:
v2 v2 v1 v1

Batch 3:
v2 v2 v2 v1

Batch 4:
v2 v2 v2 v2
```

### Rolling with Additional Batch
- AWS creates additional temporary instances first.

```
Current: v1 v1 v1 v1
AWS adds: v2 v2 v2 v2

Deploy

Batch 1:
v2 v2 v1 v1

Batch 2:
v2 v2 v2 v2
```

### Immutable Deployment
- AWS launches a completely new fleet with the new version.

```
Old Fleet:
v1 v1 v1 v1

New Fleet:
v2 v2 v2 v2

if Health checks run on the new fleet:
    Terminate old fleet.
    Keep new fleet.
else:
    Delete new fleet.
    Keep old fleet.
```
### Traffic Spiltting / Canary Deployment

- Similar to Immutable, creates a separate fleet, then:
  1. small amount of traffic 10% is routes to new fleet.
  2. Monitor the metrices, if healthy then 50% to new fleet.
  3. Eventually 100% to new fleet.

#### Quick Comparsion:

| Strategy | Downtime | Extra Instances | Risk |
|-----------|-----------|-----------|-----------|
| All-at-Once | Possible | No | Highest |
| Rolling | No | No | Medium |
| Rolling with Additional Batch | No | Temporary | Low |
| Immutable | No | Yes | Very Low |
| Traffic Splitting | No | Yes | Lowest |

****
## Amazon Athena 

- It is an interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL.

- Athena is serverless, so there is no infrastructure to manage, and you pay only for the queries that you run.

****

## AWS Redshift

- It is data-warehouse.

****

## IAM

### Policies

![Policies](<Screenshot 2026-07-20 130106.png>)

### Permissions
- **Sigv4**
  - The request must be signed using AWS credentials (IAM user, IAM role, temporary credentials, etc.) with the SigV4 signing process so AWS can evaluate the IAM permissions attached to that identity.
  - Without SigV4, AWS doesn't know who is making the request.
  - Working:
    - The client sends an S3 request.
    - The request is signed using SigV4 and AWS credentials.
    - AWS verifies:
      - Is the signature valid?
      - Who is the caller?
      - Does the caller have the required IAM permissions?
    - Access is allowed or denied.

### Simple Analogy
1. IAM permissions = What you're allowed to do.
2. SigV4 = Your identity card proving who you are.

****
# AWS Glue
- It is a serverless **ETL** (Extract, Transform, Load) service that helps you move and transform data between different data sources and targets without managing servers.

****
## AWS Security Token Service (STS)
- AWS STS (Security Token Service) is an AWS service that provides temporary security credentials to access AWS resources. These credentials include an Access Key ID, Secret Access Key, and Session Token, and automatically expire after a configured time.
- Use cases:
  - AssumeRole: Used to temporarily assume an IAM role.
  - GetCallerIdentity: Find out who you are currently authenticated as "aws sts get-caller-identityShow more lines".
  - GetSessionToken.
  - DecodeAuthorizationMessage.