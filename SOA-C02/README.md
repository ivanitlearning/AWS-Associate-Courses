These are notes for Cantrill's SOA-C02 course.

# 1. IAM-Accounts-AWS-Organizations

## 1.1. AWS Service Catalog

* Regional service
* Self-service portal for users to launch (pre-defined by admins) products
* Can control end-user permissions

### 1.1.1. How it works

1. Admins define products and portfolios using CF templates and Service Catalog config
2. Deploy portfolio to any service enabled regions
3. Service Catalog users review portfolios they have permissions on and launch product(s) into service enabled regions
4. Service catalog launches infrastructure using defined templates. Service catalog users don't need infrastructure permissions, only launch permissions.

### 1.1.2. Exam notes

* Need for end users/customers to deploy infrastructure with tight controls in a self-service way, use Service Catalog.

## 1.2. Cost Explorer

* Used for exploring costs for AWS account/organisation or any particular AWS user.
* Can be used to evaluate whether reserved instances or any other service options will benefit you cost wise
* Also show you cost breakdowns by cost tags.
* Can forecast future costs based on current utilisation
* Also recommend costs savings based on switching to Reserved Instances.

## 1.3. Cost Allocation Tags

* Enabled individually per account or master account for AWS Orgs.
* AWS-generated eg. **aws:createdBy** (which identity created a resource) or **aws:cloudformation:stack-name**
* Added to resources after enabled by AWS ie. **cannot** be enabled retroactively
* User-defined tags can also be enabled **user:something**
* Both visible in cost reports and can be used as filter
* Can take up to 24 hours to be visible and active

### 1.3.1 Tag examples

```
aws:createdBy
Root:12345678
```

## 1.4 SAML 2.0 Identity Federation

### 1.4.1. Introduction

* Security Assertion Markup Language (SAML) 2.0
* Open standard used by many identity providers (eg. Microsoft AD Federation Services)
* AWS services require AWS credentials
* Enterprise Identity Provider SAML 2.0 compatible.
* Uses IAM roles and AWS temporary credentials (12 hr validity)

#### Exam notes

* If question mentions Web identity logins (Google, FB, Apple, Twitter etc) or anything that suggests SAML 2.0 not supported, don't choose this.
* SAML 2.0 tends to be used in larger enterprises especially Windows-based identity providers.

### 1.4.2. SAML API

1. Application requests access with identity provider (eg. AD FS)
2. ID provider authenticates requests and identifies roles for app
3. ID provider gives a SAML assertion to the app
4. App uses SAML assertion with `STS:AssumeRoleWithSAML ` with STS.
5. STS accepts the role and passes temporary AWS credentials to Catagram.
6. App uses creds to access AWS resources eg. DynamoDB.

Note: Whole process requires bi-directional trust between ID-P and IAM

![](Pics/SAML-API.png)

### 1.4.3. SAML Identity Federation (Console)

Console access is similar

1. Bob browses to ID-P portal (with URL)
2. ID-P authenticates request and identifies roles for user
3. ID-P returns SAML assertion to Web client
4. Web client sends SAML assertion to sign-in URL to SAML endpoint.
5. SAML endpoint receives assertion to STS on your behalf and generates AWS temp creds.
6. Web client creates a console sign-in URL with credentials for user
7. User gets redirected to AWS management console with URL.

![](Pics/SAML-Console.png)

### 1.5 AWS Single Sign-On (SSO)

#### 1.5.1. Introduction

* Manages SSO access with AWS Accounts/Organisation and External Applications
* Flexible Identity Store
  * Once configured Identity Store, functionality of SSO same for all different types of ID store.
  * Also has in-built Identity Store
* Can use either AWS managed AD or on-prem AD with 2-way trust or AD connector
* Preferred by AWS for traditional "workforce" identity federation (**default choice** unless otherwise)
* Handles identity federation, all accounts in your organisation and also external applications.
* Can import identities and groups from the ID provider and use them within SSO to manage permissions across AWS resources.

![](Pics/AWS-SSO.png)

**Exam notes:**

* Customer identities -> Web app, Twitter, FB , Google or Web identity => AWS Cognito
* Enterprise/workplace identities -> AWS SSO

**Notes from demo:**

* No associated costs with using it, so leave it there

# 2. Simple Storage Service (S3)

## 2.1 Cross-Origin Resource Sharing (CORS)

* Either Simple requests or Preflight requests (more complicated requests)
  * Preflight request need to be done in advance to the other origins. Browser sends HTTP request to the other origin to check if request is safe to send.

### 2.1.1 Types of headers

* `Access-Control-Allow-Origin` - Indicates whether the response can be shared with requesting code from the given origin.
* `Access-Control-Max-Age` - Indicates how long the results of a [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request) (that is the information contained in the [`Access-Control-Allow-Methods`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Methods) and [`Access-Control-Allow-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers) headers) can be cached (and you need to do another preflight again)
* `Access-Control-Allow-Methods` -  Specifies the HTTP methods allowed when accessing the resource in response to a [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request).
* `Access-Control-Allow-Headers` - Used in response to a [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request) which includes the [`Access-Control-Request-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Headers) to indicate which HTTP headers can be used during the actual request.
  * Eg. `Access-Control-Allow-Headers: X-Custom-Header, Upgrade-Insecure-Requests`
* Eg

```json
{
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["PUT", "POST", "DELETE"],
    "AllowedOrigins": ["http://catagram.io"],
    "ExposeHeaders": []
}
```

## 2.2 S3 Access Points

* Allows access to S3 bucket with an endpoint with a custom bucket policy; can create multiple.
* Each access point has its own endpoint address
* Created with eg. `aws s3control create-access-point --name secretcats --account-id 1234567 --bucket catpics`
* Bucket policies can be used but become very large and unwieldy quickly.
* Based on eg below. Allows custom access for various teams.
* Each access point has a unique DNS address accessing it, and this address is passed to users.
* Can also be accessed via a VPC gateway endpoint.
* Needs to have either
  * Matching bucket policy
  * Permissions delegation of bucket policy to access point (bucket just takes whatever is specified in access point)

![](Pics/S3-Access-Pts.png)

## 2.3 S3 Inventory

* Generates storage inventory report of objects and various fields
* Fields include Encryption, Size, Last Modified, Storage Class, Version ID, Replication Status, Object lock etc.
* Can't use to generate reports on-demand; can only be scheduled either daily or weekly only.
  * Takes up to 48 hrs to generate first report.
* Available formats: CSV, ORC, Parquet
* Multiple inventories can be setup, and they go to a target bucket

### 2.3.1. Notes from demo

* Generate inventory configuration
* Can choose which bucket to store reports in
  * Destination bucket needs bucket policy to allow report to be stored; will be automatically added while setting up inventory configuration ie `s3:PutObject`

## 2.4 S3 Object Lock

### 2.4.1. Features

* Can be enabled only for new buckets (existing ones need support ticket)
* Versioning automatically enabled. This is irreversible.
* Store objects using a *write-once-read-many* (WORM) model. Object Lock can help prevent objects from being deleted or overwritten for a fixed amount of time or indefinitely. You can use Object Lock to help meet regulatory requirements that require WORM storage, or to simply add another layer of protection against object changes and deletion.

Object retention via 2 methods

* Retention period
* Legal hold

Can use both of these, 1 or none.

### 2.4.2. Retention period

* Specify duration in days, years.
* Two modes: Compliance and 

#### 2.4.2.1 Compliance mode

* Objects can't be adjusted, deleted or overwritten during the period.
* Retention period can't be changed, retention mode can't be adjusted, even for **root users**.
* Use cases: Financial, medical history records

#### 2.4.2.2. Governance mode

* Special permissions can be granted allowing lock settings to be adjusted
* Grant permissions `s3:BypassGovernanceRetention` to users and specify header `x-amz-bypass-governance-retention:true` (console default) to allow changes.

### 2.4.3. Legal hold

* Set on an object version: ON/OFF.
* No retention period; active until removed.
* Can't delete or modify object
* To add/remove legal hold, need `s3:PutObjectLegalHold` permission.
* Use cases:
  * Prevent accidental deletion of critical object versions.

# 3. Security

## 3.1. Policy Interpretation

Example of `NotAction` [statement](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws_deny-requested-region.html)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyAllOutsideRequestedRegions",
            "Effect": "Deny",
            "NotAction": [
                "cloudfront:*",
                "iam:*",
                "route53:*",
                "support:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringNotEquals": {
                    "aws:RequestedRegion": [
                        "eu-central-1",
                        "eu-west-1",
                        "eu-west-2",
                        "eu-west-3"
                    ]
                }
            }
        }
    ]
}
```

This denies actions to everything but those 4 global services in the list outside of the listed regions. These 4 global services run out of us-east-1 region. Will need an allow statement to mean anything.

## 3.2. Permissions Evaluation

### 3.2.1. Policy evaluation logic

Policies are evaluated in this order

1. Explicit deny
2. SCPs (deny only)
3. Resource policy (allow only)
4. Permissions boundary (deny only)
5. Session policy (deny only)
6. Identity policy (deny or allow)

### 3.2.2. Multi-account

Account A might allow something but account B also needs to allow it too or it gets denied (ie. A accessing a bucket in B)

## 3.2. AWS Inspector

* Scans EC2 instances and **instance OS**.
* Checks for vulnerabilities and deviations against best practices.
* Provides a report of findings ordered by priority.

### 3.2.1 Network Reachability package [[Ref](https://docs.aws.amazon.com/inspector/latest/userguide/inspector_network-reachability.html)]

* Network assessment works without agent, but can install agent for additional details.
* Checks reachability and accessibility of ports end-to-end. Works for EC2, ALB, DX, ELB, ENI, IGW, ACLs, RTs, SGs, subnets, VPCs, VGWs, & VPC Peering
  * **RecognizedPortWithListener** – A recognized port is externally reachable from the public internet through a specific networking component, and a process is listening on the port.
  * **RecognizedPortNoListener** – A port is externally reachable from the public internet through a specific networking component, and there are no processes listening on the port.
  * **RecognizedPortNoAgent** – A port is externally reachable from the public internet through a specific networking component. The presence of a process listening on the port can't be determined without installing an agent on the target instance.

* **Exam notes:** Look for keywords such as
  * *Host assessments*, *agent required*.
  * Common vulnerabilities and exposures (CVE) - will flag CVEs found on instance ports
  * Center for Internet Security (CIS) Benchmarks 
  * **Security best practices** for Amazon Inspector

## 3.3. GuardDuty

* Continuous security monitoring service.
* Learns patterns of what occurs normally within any managed accounts.
* Uses AI/ML with updated threat intelligence feeds 
* Configured to notify or event-driven protection/remediation
* Supports multiple accounts (**Master** and **Member**)
* Takes logs from
  * DNS logs
  * VPC flow logs
  * CloudTrail Event logs
  * CloudTrail Management Events
  * CloudTrail S3 Data Events
* Findings can trigger CW Events or EventBridge to send notifications to SNS or Lambda invocation

## 3.4. Trusted Advisor

* Compares your current settings with what you should have for best practices
* Account level service, needs no agents to work.
* Does cost optimisation, performance, security, fault tolerance and service limits.
* Mostly **not free**, except for **basic** or **developer** 7 core checks
* Anything more requires Business or Enterprise plans

### 3.4.1. Basic 7 core checks

1. S3 bucket permissions - **not** objects
2. Security groups - Specific ports for unrestricted access (ie. 0.0.0.0/0 or ::/0)
3. IAM use
4. MFA on Root Account - checks whether MFA enabled for root.
5. EBS public snapshots - checks for EBS snapshots marked for public access
6. RDS public snapshots - same but for RDS snapshot
7. 50 most common service limit - checks whether you're over 80% of these

### 3.4.2. Business and Enterprise support

* 115 further checks (covering costs, security, fault, tolerant, performance and service limit)
* Also covers AWS Support API programmatically
  * Get details of TA checks
  * Refresh TA checks
  * Checks status of TA checks, individually or all
  * Open a support ticket, search cases by dates and case identifiers
  * Add email comms to existing cases and resolve cases

* CloudWatch integration: Define event-driven actions if one TA checks surface issues

## 4. EC2

### 4.1. EC2 instance connect

* [List](https://ip-ranges.amazonaws.com/ip-ranges.json) of AWS address IP ranges for for EC2 instance connect.

### 4.2. EC2 savings plan

* Hourly commitment for 1 or 3 year term ($20 per hour for 3 years)
* General compute dollar amts
  * Eg. Compute products EC2, Fargate, Lambda have on demand rate but also have savings plan rate.
  * Service is billed at discounted savings plan rate until finished, then revert to normal on-demand rate.
  * Used to transition away from EC2 to Fargate, then to Lambda

### 4.3. EC2 termination shutdown protection and shutdown

* Protects EC2 instances from termination by sloppy admin.
* Right-click -> Instance settings -> Change termination protection -> Enable
* Now termination requires `disableApiTermination` attribute to terminate EC2.
* Can segregate permissions to allow only senior admins to terminate
* Can also change shutdown behaviour to either stop/terminate when shutdown from inside OS.
  * Right-click -> Instance settings -> Change shutdown behaviour

## 5. Monitoring, Logging & Auditing

### 5.1 CloudWatch - Architecture Concepts

* Public service - public space endpoints
* AWS service integration - management plane
* Agent integration - Provides richer metrics from within EC2 
* On-premises integration via Agent/API
* Application integration via Agent/API
* View data via console UI, CLI, API, dashboards & anomaly detection
* Alarms - react to metrics, can be used to notify or perform actions
* On-premises VMs and Internet apps, VPC applications

### 5.2 CloudWatch - Data

* Namespace = container for metrics eg. AWS/EC2 & AWS/Lambda
* Datapoint = Timestamp, Value, unit of measure
* Metric .. time ordered set of data points
* .. CPUUtilisation, Networking, DiskWriteBytes - EC2
* Every metric has a MetricName (CPUUtilisation) and a Namespace (AWS/EC2)
* Dimension .. 
* Key/value
* **Resolution** - Minimum time period you can get one particular data point for. Eg. Standard (60s), High (1s)
  * 60s - Data retained for 15 days
  * 5 min - Data retained for 63 days
  * 1 hr - Data retained for 455 days
* As data ages, its aggregated and stored for longer with lower resolution
* **Statistics** - Aggregation over a period (eg. Min, Max, Sum, Average...)
  * Percentile - Eg. 95th, 75th percentile

### 5.3 CloudWatch Alarm

* Alarm - watches a metric over a time period
* In two states: Alarm or OK.
* Set value of metric vs threshold over time
* One or more actions

### 5.4 CloudWatch Logs

* **Public service** - Store, monitor, access logging data
* Install CW Agent for system or custom application logging
* Sources: 
  * AWS, on-premises, IOT or any application
  * VPC flow logs
  * CloudTrail
  * Elastic beanstalk, ECS container logs, API GW, Lambda execution logs
  * Route53 - Log DNS requests
* **Exam note:** Default logging endpoint for AWS.

#### 5.4.1. CloudWatch Logs - Subscription filters

* Log stream is a sequence of log events that share the same source. Each separate source of logs in CloudWatch Logs makes up a separate log stream.
* Log group is a group of log streams that share the same retention, monitoring, and access control settings

* Real-time logging:
  * AWS managed Lambda function allows logging data to be delivered realtime to AWS Elasticsearch
* Near real-time logging:
  * Custom Lambda function can be used to export data 
  * Can use subscription filters - Eg. Kinesis data firehose allows *near realtime* delivery of logging to S3

* Can aggregate logs from various sources into Kinesis data streams, data firehose into S3

![](Pics/Cloudwatch-Logs-Aggregation.png)

#### 5.4.2 CloudWatch Logs - Exam notes

* Default for any log management scenarios
  * On-premises and AWS
* Export to S3 with `CreateExportTask` - 12 hours, not real time
* Near-realtime or persist logs - Kinesis firehose
* Firehose for any "firehose destinations"
* Realtime - Lambda (delivers to almost anything) or Kinesis Data stream (KCL consumers)
* Metric filter - scan log data, generate CloudWatch metric which you can set alarms on

### 5.5 AWS X-Ray

* Takes data from many AWS services and gives you single overview of session flow - Distributed tracing
* **Components:**
  * **Tracing header** - Generated by first service with unique trace ID, used to track request through application
  * **Segments** - Data blocks - host/IP, request, response work done, issues encountered
  * **Subsegments** - More granular version of above, calls to other services as part of a segment (endpoints etc)
  * **Service graph** - Generates a JSON doc detailing services and resources which make up the application
  * **Service map** - Visual representation of service graph (eg. below)

![](Pics/X-ray-diagram.png)

How it collects:

* EC2 - Installed X-Ray agent
* ECS - Agent installed as part of any tasks running in the service
* Lambda - Enable X-ray data collection option
* Beanstalk - Agent pre-installed
* API Gateway - Enabled on per-stage basis
* SNS & SQS - Can be enabled to send data to X-ray

* All of these require IAM permissions

## 6. Infrastructure as Code (CloudFormation)

### 6.1 Template and Pseudo Parameters

#### 6.1.1. Template parameters

* Template parameters accept input via console/CLI/API when a stack is created/updated
* Can be referenced from within Logical Resources 
* Can be configured with Defaults, AllowedValues, Min and Max length & AllowedPatterns, NoEcho & Type

![](Pics/Template-Parameters.png)

#### 6.1.2. Pseudo parameters

* Pseudo parameters are similar to template parameters only are provided by AWS based on environment when creating the stack.
  * **AWS::Region** will always reflect the region the stack is being created in.
  * **AWS::StackName** and **AWS::StackId** will match the specific Stack being created
  * **AWS::AccountId** will be set by AWS to the actual account ID the stack is being created within.

### 6.2. Intrinsic Functions

#### 6.2.1. **Ref** & Fn::**GetAtt**

Reference a resource you created, such as a VPC so that a subnet can be created inside.

* !Ref Instance points to the ID of the instance created. When used with logical resources, the physical ID is usually returned.
  * Eg. i-12345679acfgsd0
* !GetAtt LogicalResource.Attribute can be used to retrieve any attribute associated with the resource.
  * Eg. PublicIP 54.91.129.183, PublicDNSName ec2-54-91-129-183.compute-1.amazonaws.com

#### 6.2.2. Fn::**GetAZs** & Fn::**Select**

* GetAZs "us-east-1" or "" (current region) - Returns a list of AZs from a Region
  * ["us-east-1a", "us-east-1b", "us-east-1c", "us-east-1d"]
* Assuming default VPC or its subnets are not modified
* AvailabilityZone: !Select [ 0, !GetAZs, '' ] - Returns an object from a list of object. Lists start at index 0.

#### 6.2.3. Fn::**Join** & Fn::**Split**

* !Split [ "|", "roffle|truffles|penny|winkie"] returns a list ["roffle","truffles","penny", "winkie"] 

* Join can be used to concatenate a Web path to the DNS name of a public EC2 instance for people to use.
* !Join [delimiter, ["value1", "value2" ... "valueN"]]
  * Value: !Join [ '', [ 'http://', !GetAtt Instance.DNSName ] ] - Adds http:// to the DNS name of the EC2 instance

#### 6.2.4. Fn::Base64 & Fn::Sub

* Fn::**Base64** - Base64 used for providing user-data to EC2 instances for automated builds. 

* Fn::**Sub** - Allows you to substitute things within text based on runtime information

* Fn::Base64: !Sub | 

  ```yaml
  Fn::Base64: !Sub | 
    #!/bin/bash -xe
    yum -y update
    echo "something ${Instance.InstanceId}" >> /var/www/html/index.html
  ```
* ${Instance.InstanceId} can't do self-reference, only other instance IDs. Note the above is invalid.

#### 6.2.5. Fn::Cidr

* Used to generate a number of smaller CIDR ranges from a larger PC range

```yaml
VPC:
  Type: AWS::EC2::VPC
  Properties: 
    CidrBlock: "10.16.0.0/16"
Subnet1:
  Type: AWS::EC2::Subnet
  Properties:
    CidrBlock: !Select [ "0", !Cidr [ !GetAtt VPC.CidrBlock, "16", "12" ]]
    VpcId: !Ref VPC
```

* Here we are telling CF we want 16 subnets and the size (12) of each subnet. This outputs a list of subnets to use.

Conditions (Fn::**IF**, **And**, **Equals**, **Not** & **Or**) - Typical conditions, if X do Y else

### 6.3. CloudFormation Mappings

Structure: **!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]** 

Given this

```yaml
Mappings:
  RegionMap:
    us-east-1:
      HVM64: "ami-0ff8a915034f43254"
      HVMG2: "ami-0a3ed46234422bcee"
    us-east-2:
      HVM64: "ami-0ed3a4567c4f43423"
      HVMG2: "ami-066ee5fd4a9ef77f1"
```

**!FindInMap [ "RegionMap", !Ref 'AWS::Region',"HVM64"]** 

* Looks for RegionMap in CF template
* Finds the current region in the console
* Returns value in "HVM64"

### 6.4. CloudFormation Output

* Optional, may not be present
* Values declared in this section are visible using CLI or console UI
* Accessible from a parent stack when using **nesting**
* Can be exported allowing cross-stack references

```yaml
Outputs:
  WordpressURL:
    Description: "Instance Web URL"
    Value: !Join [ '', [ 'http://', !GetAtt Instance.DNSName ] ]
```

* Description visible from CLI and console UI and passed back to parent stack when nested stacks are used

### 6.5. CloudFormation Conditions

* Created in optional Conditions section of template
* Evaluated to TRUE or FALSE.
* Processed **before** resources are created, associated with logical resources to control if they're created or not.
  * Eg. Create different number of resources based on ONEAZ, TWOAZ
* Example:

Here if EnvType is not "prod", then only "Wordpress" EC2 instance is created. If EnvType is "prod" then in addition, Wordpress2, MyEIP, MyEIP2 is also triggered. Another EC2 instance is created, and both are assigned Elastic IPs.

![](Pics/CF-conditions.png)

### 6.6. CloudFormation DependsOn

* CF tries to efficient, tries to do things in parallel (create, update and delete resources)
* Determines a dependency order (VPC -> Subnet -> EC2)
* DependsOn lets you explicitly define dependencies so CF will not build that resource until dependencies are met.

```yaml
InternetGatewayAttachment:
  Type: 'AWS::EC2::VPCGatewayAttachment'
  Properties:
    VpcId: !Ref VPC
    InternetGatewayId: !Ref InternetGateway
```

* This creates an implicit dependency on `VPC` and `InternetGateway`. It works without specifying `DependsOn`.
* However Elastic IP doesn't work, there's no `!Ref` in the template:

```yaml
WPEIP:
  Type: AWS::EC2::EIP
```

* Will encounter errors if 
  * CF tries to create EIP before attaching IGW
  * CF tries to delete attached IGW before EIP

* Adding DependsOn fixes the issue

```yaml
WPEIP:
  Type: AWS::EC2::EIP
  DependsOn: InternetGatewayAttachment
  Properties:
    InstanceId: !Ref WordpressEC2
```

### 6.7 CloudFormation WaitCondition, CreationPolicy & `cfn-signal`

#### 6.7.1. CloudFormation Signal

