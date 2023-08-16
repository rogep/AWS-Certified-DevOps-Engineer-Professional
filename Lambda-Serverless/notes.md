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

### Lambda Layers

Layers allow new runtimes and allow libraries to be externalised (/opt). Deployment ZIPs are smaller,
with shared libraries reused between functions.

### Lambda & ALB Integration

Using mutli-value headers, the load balancer uses both key values sent by the client and sends you an event that includes query string parameters
using `multiValueQueryStringParameters`

- without multi-value headers, the ALB sends the last value sent by the client
- `queryStringParameters` vs `multiValueQueryStringParameters`

### Lambda Resource Policy

Every lambda functions have two forms of security

- Execution Role

  - Assumed by the function
  - determines **WHAT** the lambda function **CAN DO**

- Resource Policy
  - **WHO** can do **WHAT** with the lambda function
  - A use case can be multi account invocations
  - Cross account requires Identity (OUT) **AND** Resource Policy (IN)
  - When a service needs to invoke a lambda

## API Gateway

A service that lets you create and manage APIs.

- Acts as an endpoint/entry-point for applications
- Sits between applications & integrations (services)
- Can connect to services/endpoints in AWS or on-premises
- HTTP, REST, Websocket APIs

### Authentication

- Cognito user pools
- Lambda-based authorisation (bearer token)
  - A lambda authoriser is called
  - return an IAM policy and a principal identifier
  - returns 403 ACCESS denied to client if invalid

### Endpoint types

- Edge optimised
- Routed to the nearest cloudfront POP
- regional - clients in the same region
- Private - endpoint accessible only within a VPC interface endpoint

### Stages

APIs are deployed to stages -- each stage has one deployment

- customers, dev, prod etc
- can rollback
- canary deployments (certain % of traffic is sent to the canary and can be adjusted over time or promote)

### Error codes

- 4XX Client Error
  - invalid request on client side
  - 400 Bad request - Generic
  - 403 Access denied - Authoriser denies - WAF Filtered
  - 429 API gateway can throttle - exceeded throttle limit
- 5XX Server error
  - Valid request, backend issue
  - 502 Bad gateway exception - bad output returned by lambda
  - 503 Service unavailable - backing endpoint offline? Major service issues
  - 504 Integration failure/timeout - 29s limit

### Caching

- Configured per stage
- Without a cache, services are triggered each request
- Cache TTL default is 300 seconds
  - Configurable min 0 and max 3600s
  - Can be encrypted
  - Cache size 500MB to 237GB
- Calls are only made to backend integrations if a request is a cache miss

### Methods and Resources

Methods are where integrations are configured which provide the functionality of an API e.g. lambda, HTTP, & AWS service (GET, POST, etc).
Resources are points in the API tree `/route`

### Integrations

Three phases

1. Request (authorise, validate, transform)
2. Integrations -- backend (lambda, http, aws services...)

- proxy implies just passing the data through

3. Response (Transform, prepare, return)

- API methods (client) are integrated with a backend endpoint
  Types:
- MOCK: Used for testing -- no backend involvement
- HTTP: Backend HTTP Endpoint
- mapping templates required
- HTTP Proxy: Pass through to integration unmodified, return to the client unmodified (backend need to use supported format)
- AWS - Lets an API expose AWS service actions
  - need to manually configure mappings for method, int, response (complex)
  - mapping templates required
- AWS_PROXY (Lambda) - Low admin overhead lambda endpoint
  - Lambda needs to be able to interpret the event for data conversion

Mapping templates

- used for AWS and HTTP (non-proxy) integrations
- modify and rename parameters
- modify the body or headers of the request
- filtering - removing anything which isnt needed
- uses VTL (velocity template language)
- common exam scenario
  - REST API (on API gateway) to a SOAP API .. transform the request using a mapping template

### Deployments

Like lambda, changes are not live until you publish the deployment

- unlike lambda, they are not immutable and can be rolled back
