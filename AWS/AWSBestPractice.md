# AWS Cloud Best Practice
## Difference between Traditional and Cloud Computing Environment
### IT Assets as Provisioned Resources
Provision capacity based on an estimate of a theoretical maximum peak > access as much or as little capacity as you need and dynamically scale to meet actual demand

### Global, Available and Scalable Capacity
reduce latency to end users around the world by using the Amazon CloudFront content delivery network

### Higher-level Managed Services

### Built-in Security
AWS Cloud provides governance capabilities that enable continuous monitoring of configuration changes to your IT resources.
embed your security policy within the design of your infrastructure.

### Architecting for Cost
optimizing for cost is fundamental design tenant for architects.

### Operation on AWS
* Infrastructure as Code through API
* automation of the operational processes as the supporting services.
* DevOps processes for delivery pipeline.




## Design Principles
### Scalability
support growth in users, traffic or data size with no drop-in performance.

1. Stateless application

2. Distribute load to multiple nodes
to distribute the workload to multiple nodes in your environment, you can choose either a push or a pull model.
* **push** model: **ELB** can routes incoming requests across multiple EC2 instances.
* **pull** model: **SQS** for asynchronous, event-driven workloads

3. Stateless Components
* by not storing anything that needs to persist for more than a single request in the local file system.
* web application can use HTTP cookies to store session information in the web client cache.
==> 1) treat it as untrusted data that must be validated, 2) keep their size to minimum to avoid unnecessary latency.
* store the session in Amazon DynamoDB
* if larger files are engaged—user uploads, interim results of batch result—goes to S3 or EFS.
* complex multi-step workflow ==> AWS Step Function

4. Stateful Components
* distribute the load to multiple nodes with session affinity.
* existing sessions do not directly benefit from the introduction of newly launched compute nodes.

5. Implement Session Affinity
* sticky sessions feature of an Application Load Balancer to bind a user’s session to a specific instance.
* use DNS, or build a simple discovery API to provide that information to the software running on the client.

6. Distributed Processing
processing of very large amounts of data requires a distributed processing approach.

7. Implement Distributed Processing
For real-time processing of streaming data, Amazon Kinesis partitions data in multiple shards that can then be consumed by multiple Amazon EC2 or AWS Lambda resources to achieve scalability.


### Disposable Resources Instead of Fixed Servers
* fixed resources lead the upfront cost and lead time of introducing new hardware.
* software patches applied through time can result in untested and heterogeneous configuration across different environment.
==> fixed servers to temporary resources.

#### Instantiating Compute Resources
1. Bootstrapping
* simple scripts and configuration management tools such as Chef of Puppet.
* custom scripts and the AWS APIs, or with AWS CloudFormation support.

2. Golden Images
* customize EC2 instance and save its configuration by creating an AMI.
* VM import / export from AWS to convert a variety of virtualization formats to an AMI.

3. Containers
* Docker allows you to package a piece of software in a Docker image—standardized unit for software development, containing everything the software need.
* AWS Elastic Beanstalk, ECS, Fargate let you deploy and manage multiple containers across a cluster of EC2 instances.

4. Hybrid
* AWS Beanstalk follows the hybrid model: provide preconfigured run time environment, but also allows bootstrapping through extensions.

#### Infrastructure as Code
CloudFormation templates.


### Automation
#### Serverless Management and Deployment 
serverless patterns, the operational focus is on the automation of the deployment pipeline 

#### Infrastructure Management and Deployment
1. AWS Beanstalk: upload their application code, and the service automatically handles all the details, such as resource provisioning, load balancing, auto scaling, and monitoring. 

2. EC2 Auto Recovery: CloudWatch alarm and recovers it if it becomes impaired.

3. System Manager: automatically collect software inventory, apply OS patches, create a system image to configure Windows and Linux operating systems, and execute arbitrary commands.


#### Alarms and Events
1. Amazon CloudWatch alarm by SNS.
2. Amazon CloudWatch Events: a near real-time stream of system events that describe changes in AWS resources.
3. AWS Lambda scheduled events: Lambda can execute a function on a regular schedule.
4. AWS WAF security automation: create custom, application-specific rules that block common attack patterns.

### Loose Coupling
#### Well Defined Interface
* allow the various components to interact with each other only through specific, technology-agnostic interfaces, such as RESTful APIs 
* Amazon API Gatewayis a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale 


#### Service Discovery
Because each of those services can be running across multiple compute resources, there needs to be a way for each service to be addressed. 

1. Elastic Load Balancing (ELB). Because each load balancer gets its own hostname, you can consume a service through a stable endpoint. 

2. Another option is to use a service registration and discovery method to allow retrieval of the endpoint IP addresses and port number of any service .

3. If there is not LB, Amazon Route 53 supports auto naming to make it easier to provision instances for microservices .


#### Asynchronous Integration
The two components do not integrate through direct point-to-point interaction but usually through an intermediate durable storage layer, such as an SQS queue or a streaming data platform such as Amazon Kinesis, cascading Lambda events, AWS Step Functions, or Amazon Simple Workflow Service. 

* A front-end application inserts jobs in a queue system such as Amazon SQS. A back-end system retrieves those jobs and processes them at its own pace. 

* An API generates events and pushes them into Kinesis streams. A back-end application processes these events in batches to create aggregated time-series data stored in a database. 

* Multiple heterogeneous systems use AWS Step Functions to communicate the flow of work between them without directly interacting with each other. 



#### Distributed Systems Best Practices
provide alternative or cached content instead of failing completely when  your database server becomes unavailable. 

The Amazon Route 53 DNS failover feature also gives you the ability to monitor your website and automatically route your visitors to a backup site if your primary site becomes unavailable. Y



### Services, Not Servers
By using Amazon API Gateway, you can develop virtually infinitely scalable synchronous APIs powered by AWS Lambda. 

Amazon Cognito provides temporary AWS credentials to your users, allowing the mobile application running on the device to interact directly with AWS Identity and Access Management (IAM)-protected AWS services 


### Databases
1. RDS
* Relational databases can scale vertically by upgrading to a larger Amazon RDS DB instance or adding more and faster storage.
* horizontally scale beyond the capacity constraints of a single DB instance by creating one or more read replicas 
* write capacity ? data partitioning or sharing.
* no need for joins or complex transactions ? consider a NoSQL.

2. NoSQL
* NoSQL database engines will typically perform data partitioning and replication to scale both the reads and the writes in a horizontal fashion. 
==> no data partitioning logic implemented in the data access layer of your application 
* Amazon DynamoDB synchronously replicates data across three facilities in an AWS Region, 

3. Data Warehouse
* optimized for analysis and reporting of large amounts of data 
* transactional data from disparate sources ==> available for analysis and decision making.
* you deploy production workloads in multi- node clusters, so that data that is written to a node is automatically replicated to other nodes within the cluster. 
* backed up to S3.

4. Search
* A search service can be used to index and search both structured and free text format and can support functionality that is not available in other databases, such as customizable result ranking, faceting for filtering, synonyms, and stemming. 

* Amazon CloudSearch is a managed service that requires little configuration and will scale automatically 

* Amazon CloudSearch and Amazon ES use data partitioning and replication to scale horizontally 


5. Graph Database
enable data in the store to be linked together directly, which allows for the fast retrieval of complex hierarchical structures in relational systems. 


### Managing Increasing Volumes of Data
A data lake is an architectural approach that allows you to store massive amounts of data in a central location so that it’s readily available to be categorized, processed, analyzed, and consumed by diverse groups 


### Removing Single Point of Failures
#### Detect Failure
* ELB and Amazon Route 53 to configure health checks and mask failure by routing traffic to healthy endpoints. 
* replace unhealthy nodes automatically using Auto Scaling or by using the Amazon EC2 auto-recovery feature or services such as AWS Elastic Beanstalk 

* Configuring the right health checks for your application helps determine your ability to respond correctly and promptly to a variety of failure scenarios. 
==> you should assess whether the web server can return an HTTP 200 response for some simple request. 

* *deep health check*, which is a test that depends on other layers of your application to be successful, because false positives can result 
==> might be appropriate to implement at the Amazon Route 53 level 


#### Durable Data Storage
1. Synchronous replication
S3 ==> With versioning, you can recover from both unintended user actions and application failures. 

2. Asynchronous replication
decouples the primary node from its replicas at the expense of introducing replication lag 

#### Automated Multi-Data Center Resilience
Because of the long distance between the two data centers, latency makes it impractical to maintain synchronous cross-data center copies of the data. 

Amazon RDS provides high availability and automatic failover support for DB instances using Multi-AZ deployments, while with Amazon S3 and Amazon DynamoDB your data is redundantly stored across multiple facilities. 

#### Fault Isolation and Traditional Horizontal Scaling
instead of spreading traffic from all customers across every node, you can group the instances into shards. 


### Optimize for Cost
benchmark your application environment and select the right instance type depending on how your workload uses CPU, RAM, network, storage size, and I/O. 

define and implement a tagging policy for your AWS resources. 


### Caching
#### Application Data Caching
ached information might include the results of I/O- intensive database queries, or the outcome of computationally intensive processing. 

* Amazon ElastiCache 
* Amazon DynamoDB Accelerator (DAX) is a fully managed, highly available, in-memory cache for DynamoDB that delivers performance improvements from milliseconds to microseconds, for high throughput. 


#### Edge Caching
Copies of static content (images, CSS files, or streaming pre-recorded video) and dynamic content (responsive HTML, live video) can be cached at an Amazon CloudFront edge location, which is a CDN with multiple points of presence around the world. 
Amazon CloudFront reuses existing connections between the Amazon CloudFront edge and the origin server 

### Security
#### Reduce Privileged Access
* Amazon CloudWatch Logs to collect this information. 
* AWS Systems Manager 55 to take a unified view and automate actions on groups of resources. 
* use IAM roles to grant permissions to applications running on EC2 instances through the use of short-term credentials 

#### Security as a Code
CloudFormation and deploys your resources in alignment with your security policy. 


#### Real-Time Auditing
* AWS Config, Amazon Inspector, and AWS Trusted Advisor continually monitor for compliance or vulnerabilities, giving you a clear overview of which IT resources are in compliance, and which are not. 

* AWS CloudTrail is a web service that records API calls to supported AWS services in your AWS account and delivers a log file to your S3 bucket 

* AWS Lambda, Amazon EMR, Amazon ES, Amazon Athena, or third-party tools from AWS Marketplace to scan log data to detect events such as unused permissions, privileged account overuse, key usage, anomalous logins, policy violations, and system abuse 
