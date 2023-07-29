# Lambda, Serverless & Applications

## CloudWatch EventBridge (Previously Events)

Delivers a near real\*time stream of events (systems started, stopped, performed actions)

- If X happens, or at Y times(s)... do Z
- A **default** Event bus for the account (multiple for EB, 1 for Events)
- Rules match incoming events, or scheduled based/pattern matching rules (think UNIX cron)
- Events are just JSON structures

## Lambda

- Deployment package 50mb zipped, 250mb unzipped
- Docker is an anti pattern for lambda (not container images with lambda)
- Custom runtimes such as rust are possible using layers and docker (bootstrap)
- vCPU scales with memory (1vCPU = 1769MB)

### Networking

#### Public

By default, lambda functions are given public networking. Accessing AWS services and public internet.
This offers the best performance as no customer specific VPC network is required -- but cannot access VPCs.

#### Private

Lambda functions running in a VPC obey all VPC networking rules. Typically use a VPC endpoint (gateway endpoint) to access DB's/S3. Or use a NAT gateway.

Lambdas (and Fargate) are ran inside of a separate AWS Lambda Service VPC and interact with your VPC through through a single elastic network interface. One interface per VPC to enable scalability.

#### Security

- Lambda requires an **execution role** with a trust policy (same as an EC2 instance role).
- Lambda resource policy controls WHAT services and accounts can INVOKE lambda functions
  - Can only be changed through the API or CLI (not in the console)

### Invocations

- Synchronous
  - CLI / API invoke a lambda and wait for a response
  - API gateways while client waits for a response
  - Result (success/failure) returned during the request
  - Errors or retries have to be handled within the client
- Asynchronous
  - Typically used when AWS services invoke lambda functions
  - The lambda function needs to be idempotent (rerunning should have the same end state)
  - Events can be sent to a dead letter queue after repeated failed processing
  - Lambda supports destinations (SQS, SNS, Lambda & EventBridge) where successful or failed events can be sent
- Event Source Mapping
  - Typically used on streams or queues which don't support event generation to invoke lambda (Kinesis, DynamoDB, SQS)
  - Sources batches and sends events as event batches
  - Need to ensure that they don't exceed the 15 minute timeout
  - Permissions from the lambda execution role are used by the event source mapping to interact with the event source
  - DLQ can be used for this again

#### Versions

- A version is the code + the configuration of the lambda function
- Immutable (code and settings fixed) and has its own ARN
  - Qualified ARN points at a specific version (needs to be published)
  - Unqualified ARN points at the function ($LATEST) - not a specific version (just deployed)
- Aliases (DEV, PROD) point at a version - can be changed
  - Pointer to a function version
  - Useful for BLUE/GREEN and A/B testing

#### Execution Context

- An execution context is the environment a lambda function runs in. A cold start is a full creation and configuration included function code downloaded.
- A warm start is if future invocations reuse the execution contest.
- Lambda's should assume they can't reuse contexts.
  - If used infrequently, contests will be removed
  - Concurrent executions will use multiple (potentially new) contexts
- Provisioned concurrency can be used
  - AWS will create and keep X contexts warm and ready to use -- improving start speeds

### Lambda Function Handler Architecture

- Lambda executions have lifecycles (lambda execution environment)
- Steps
  - INIT - Creates or unfreezes the execution environment
  - INVOKE - Runs the function Handler (Cold start)
  - NEXT INVOKE - WARM START - same environment
  - SHUTDOWN - Terminate the environment
