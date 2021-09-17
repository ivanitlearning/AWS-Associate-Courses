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

* Account level service, needs no agents
* Does cost optimisation, performance, security, fault tolerance and service limits.
* Mostly **not free**, except for **basic** & **developer** 7 core checks
* Anything more requires Business or Enterprise plans

### 3.4.1. 7 core checks

1. S3 bucket permissions - **not** objects
2. Security groups - Specific ports for unrestricted access (ie. 0.0.0.0/0 or ::/0)
3. IAM use
4. MFA on Root Account - checks whether MFA enabled for root.
5. EBS public snapshots - checks for EBS snapshots marked for public access
6. RDS public snapshots - same but for RDS snapshot
7. 50 most common service limit - checks whether you're over 80% of these

