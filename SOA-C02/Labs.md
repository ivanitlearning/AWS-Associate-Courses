# Lambda functions

### Steps to create Lambda function

1. Create Lambda execution role
   1. IAM -> Create role -> AWS Service, Lambda -> Next, Permissions
   2. Custom role (JSON policy) or search from list
   3. Select role from list -> Next
   4. Name the role -> Create role
2. Go to AWS Lambda -> Create function
   1. Select code runtime (eg Python 3.8)
   2. Change default execution role -> Use an existing role -> Select existing role -> Create Function
3. Select lambda_function.py -> Edit your code there -> Deploy
4. If environment variables needed, go to Configuration -> Environment variables -> Edit them in
5. Test by going to Test tab, test with event (manual invocation)
6. Should show **Execution result: succeeded**

### Allow Lambda function to access EFS

1. Create roles allowing Lambda function to access EFS with these permissions policies
   1. `AWSLambdaVPCAccessExecutionRole` - Allows ENI injection into VPC
   2. `AmazonElasticFileSystemClientFullAccess` - Mount, write access to EFS mount points
2. Create access point for Lambda to access EFS
   1. EFS -> Click on file systems -> Access points tab -> Create access point
      * Fill in user ID and group ID
      * Set POSIX permissions for root directory
   2. Create access point
3. Create Lambda function
   1. Select IAM role
   2. Edit timeout (max 15 min)
4. Go to Configuration tab -> VPC tab
   1. Select VPC, subnets, SG
   2. Ensure that the SG the Lambda function is in has access to the SG the EFS has.
   3. Lambda will create 2 ENI inside selected subnets
   4. Save
5. Go to Configuration tab -> File systems -> Add file system
   1. Select target EFS, access point
   2. Specify mount path **/mnt/efs**
6. Update code and test

# EventBridge

## Creating a rule to auto-start stopped instances

1. EventBridge -> Create rule
2. Event-driven -> Choose Event pattern. Otherwise choose Schedule
   1. For event pattern (auto-starting stopped instances)
      - Service provider: AWS
      - Service name: EC2
      - Event type: EC2 Instance state-change notification
      - Specific states: Stopped
      - Specific instance IDs (if needed)
3. Select targets (what to do if event matches pattern)

# Simple Email Service

1. Whitelist sending email
   1. Email Addresses -> Verify a New Email Address

# Step Functions

1. State machines -> Create state machines
2. [Demo] Write workflow in code
   1. Copy and paste the JSON code
3. Next
   1. Choose existing role
   2. Logging: Select ALL or as required
4. Select create state machine

# API Gateway

1. API Gateway -> APIs

2. Choose REST API -> Build

3. Select New API

   1. Select endpoint type Regional or as required

4. Select Create API

5. Select Actions -> Create Resource

   1. Enable API Gateway CORS if needed

6. Select created resource -> Create Method

   1. Select HTTP method (here POST)

   2. Integration Point (what the API-GW does when it gets POST request) -> Lambda function
   3. Type in Lambda function name

7. Confirm AWS edit of Lambda function permissions to allow API-GW to invoke it

8. Actions -> Deploy API

   1. Create stage with [New Stage]
   2. Deploy

9. Copy URL of API Gateway at the top for use

# S3

## Static website hosting

1. S3 -> Create bucket
   1. Uncheck protect bucket from public access -> Create bucket
2. Click on bucket name -> Permissions tab
   1. Bucket policy -> Edit
   2. Add in bucket policy
   3. Save
3. Scroll all the way down, EDIT for static website hosting
   1. Check Enable
   2. Specify index and error document (eg. index.html)
   3. Save changes
4. Click Upload
   1. Upload all the files for static website hosting (include index and error docs)
   2. Click Upload
5. Properties tab, scroll down to static website hosting -> Copy bucket website endpoint

## S3 Versioning

1. S3 -> Create bucket
2. Enable bucket versioning, Create bucket
3. Click on bucket -> Enable S3 static site hosting if needed
4. Permissions tab -> Edit bucket policy
5. Add in policy or use policy generator
6. Upload files
7. Objects tab -> Check version ID
8. Upload new files to overwrite old ones
9. The website should now show new images
   * Note that deleted objects are instead given deleted markers.

## S3 object encryption

1. Create bucket
2. When uploading objects, expand Properties
3. Specify encryption key
   1. SSE-KMS or SSE-S3
   2. Select either AWS managed key or CMK for KMS
   3. Note you can click Properties tab and specify default encryption settings for new objects (doesn't prevent selecting otherwise)
4. Upload object
5. Verify that user without access to specific KMS CMK can't open objects encrypted with blocked key

## Cross-region replication of S3 static website

1. Create two buckets, one source, one destination, make both public and enable versioning
2. Edit bucket policy on both buckets to allow * principal to access /*
3. Go to source bucket -> Management tab -> Create replication rule
4. Enable, specify destination bucket
5. Choose IAM role which allows destination bucket to read from source bucket or "Create role" to let  AWS do it for you
6. Create

## S3 presigned URLs

This demo creates S3 time-limited presigned URLs for outsiders to access with AWS permissions

1. Create bucket, upload objects

2. Open CloudShell and type

   ```text
   aws s3 presign s3://animals4lifemedia4324124/all5.jpg --expires-in 180
   ```

   Get the S3 URL from the S3 object properties

 3. You'll see this returned

    ```text
    https://animals4lifemedia4324124.s3.us-east-1.amazonaws.com/all5.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAT3SQ5UUXECTVJZ75%2F20211003%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20211003T100041Z&X-Amz-Expires=180&X-Amz-SignedHeaders=host&X-Amz-Security-Token=IQoJb3JpZ2luX2VjELL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIEA5JD8RIqMuZCz0fYgHd8XkJDihsNdXc02qB7I7D%2FcXAiAi1KyFphPkI4FjpWjnEHhk8BqBLEc7TKBnL1J6umxbSCqeAwgrEAAaDDI2NTM4Mzk0NTUxOCIMexQg%2Fw98z28pcd31KvsCiqhpQSX8YtC2yhR8n8bxxulOYJ5%2FmT4ZXWIAlRjMeszc2hRNYTQcxFjNnWm3mq%2BqWoSSVHlNse5sUbojiyRN6mA3Gjs7VenpmlTW6fjkN9lxIIwAA3kZhqhhHE2iP9vOYuwx2anTbD7muimoyB%2FmysLsVfjefK%2FPghRwrCDNY%2Ffk5E55FsPtmqVFHt8HXUv6uI2GSIzHfKh39Xcx2FfU6mZry9eOuLonFWyy3PGFvRN%2BD3vGIjzQppU8hFQOubBKoqtuoIs4QEKu%2FFahb8XbLZ302kdExORHfhIUF8W0v9VA0pJ1Z%2FbGqP0ZCcDZgH8QvepqX3XhgCRMxBTVrEo1Be8N78VE4Ok6bBUpEMD9SRLuU9VBM2V7Rv1fWwxhatpIn2BLKgtUyGuDrsH9UYZtHXiRHNWtu3kgz4FGeaEqVnUu1rS63lD4DnrsdkDwUWrXwK%2FAPWtHsuR45IfdoiToH3ALWygMam2n9wYKUzeHRa8jaPfsUZeEtIy00TDO5OWKBjq0Ap0HIIU0s5fXYr0CrGzDRoU7s3MxxDMdz0RsGhXG0p30z7yukG%2Bhvvos4AqGYDutVaum0bwrKY3Lb7NBc0z8mTVmUfB85AiDGFhyPEsReff%2BJqiWbDUwLIdP2E90E8iLRypfthR%2BG0M6kEs4Pfjmt%2BDokzy36WI7FTPYkqq1k10GFJpeXEqIXaNn5DfLeoxwqh%2ByhQdtCwTqi0jTIkYKltxNhVNq%2BpjRl%2FtFBqghWh6J8li3dgR6eNrOyxY8z7Jm6RPw7YmHR9UOUq6I2nXEaKiYOdKIYpURK2MMZoGOQvucTcBmAFAJQEaGG3SkG8oNtht%2BvDsli8fPC%2FC3dlx1TqzGXx3ys0tXYG6fTN29DY2hMFEOcWK8WiuJLzPxrP3CIkIogW%2FrKh6Jskf5HRyF0us%2F9cdC&X-Amz-Signature=db709700fdbcd92df41488b92ba83cc4ecb5d1994bca548de5d49bc72ef7bbac
    ```

## S3 Inventory

Needs two buckets, one destination bucket for reports to be stored and the source to have inventory list created.

1. Go to source bucket -> Management tab -> Create inventory configuration
   1. Specify prefix "/" if needed, object versions
   2. Also frequency and report format
   3. Create
2. Check that target bucket has policy applied and 48 hrs, then check for reports.

# Elastic Beanstalk

## Creating new application

1. Enter name
2. Select Platform, Platform branch, and platform version
3. Upload code

## Deploying new app version

1. Elastic Beanstalk -> Applications -> Select application
2. Click environment name
3. Upload and deploy
4. Select deployment policy and health check
5. Click Deploy
6. Wait for Events to finish
7. Click link to see new environment

### Cloning

* Elastic Beanstalk -> Select environment -> Actions -> Clone environment
  * Can change URL
  * Can't change platform branch (eg. Python 3.7 on x64 AMZN Linux 2)
  * Can change platform version

# IAM

## AWS Organizations

### Join AWS organization as member

These steps assume the account to be made member of AWS Org already exists

**In account to be made master:**

1. AWS Organizations -> Create Organization
2. May need to click confirmation link in email
3. Click 'Add an AWS account'
4. Enter account ID of AWS account to invite

**In account to be made member:**

1. AWS Organizations -> Accept invitation

**In account to be made master:**

1. Refresh AWS Organizations, verify that the member account has been added

### Create member role that can be assumed by master

**In member account:**

1. IAM -> Roles -> Create role
   1. Select 'Another AWS account'
   2. Enter account ID of master account
   3. Click 'Next: Permisions'
2. Select appropriate Permissions policy for role (demo **AdministratorAccess**)
3. Next -> Enter role name -> Create role
4. Verify Role in IAM by going to role in IAM -> Trust relationships

### Use master account to role switch to member account role

1. Click top-right dropdown -> Switch Role
2. Enter 
   1. Account ID of member account
   2. Exact name of role created by member account (demo OrganizationAccountAccessRole)
3. Click Switch Role
4. Will enter role, check top right corner. Check that it creates Role History in dropdown menu

### Join AWS organization as *created* member account

1. AWS Organizations -> Add AWS account
2. Create an AWS account
   1. Enter Account name
   2. Email address for new account
   3. Name of IAM role for master account to assume in created member account
3. Click Create Account
4. Wait for account to be created

5. Do the same above to create Role History

### Move member accounts into OUs

1. Check box next to Root container in Organizations -> Actions -> Create new OU
2. Enter name -> Create OU
3. Once OU created, select member to be moved
4. Actions -> Move -> Select destination OU

### Attach service control policies

First create the policy, then attach it to certain member accounts

**In master account:**

1. AWS Organizations -> Policies -> Service control policies
2. Enable Service control policies
3. Create policy
4. Enter name and edit in JSON/YAML policy below -> Create policy
5. Once created go to Accounts in AWS Organizations -> Click on account to attach to
6. Policies tab -> Attach -> Select policy to attach
7. Policies tab -> Detach old policy if needed
8. Verify that member account has no access to restricted resource (eg. S3)

### Attach IAM policy to IAM user

1. IAM -> Select user
2. Permissions tab -> Add permissions or 'Add inline policy'
3. Select 'Attach existing policies directly'
   1. Create policy if non-existent, copy-paste JSON
4. Select policy -> Next: Review -> Add permissions

Similarly you can remove policy from IAM user

### Attach IAM policy to IAM group

1. IAM -> User groups -> Create group
   1. Add specific users to group
   2. Attach permissions policies to group
   3. Create group
2. Verify in user account

# CloudTrail

## Create Organizational Trail 

1. Login to master account

2. CloudTrail -> Create trail

3. Enter

   1. Name
   2. Select **Enable for all accounts in organization**
   3. Specify S3 bucket target for logging
   4. Enable/disable SSE-KMS encryption
   5. Enable/disable CW Logs to monitor CloudTrail logs to notify if specific activity occurs
      1. Specify new/existing log group for CW Logs
      2. Specify new/existing IAM role (new role will have policy document attached)
   6. Click Next
   7. Select Event types: Management, Data, Insights (unusual activity in account)
   8. Click Next -> Review and create trail

4. Click on CloudTrail and go to S3 bucket which stores trail to view

5. In CW Logs -> log group you'll see several streams with the name format **[org-name]-[acc-no]_CloudTrail_us-east-1** eg. `o-96izg9oguo_753355385669_CloudTrail_us-east-1`


# CloudWatch

### EC2 monitoring

1. Enable detailed monitoring when launching EC2 instance
2. Go to Monitoring tab for EC2 instance

### Create CW alarm for EC2 monitoring

This alarms triggers when your EC2 monitoring metric for CPU utilization exceeds a certain value

1. CloudWatch -> All alarms -> Create alarm
2. Select metric
3. Locate EC2 -> Per-Instance metrics
4. Look for CPUUtilization for the specific Instance name/ID -> Select metric
5. Set 
   1. Threshold: Static
   2. Greater (>threshold)
   3. 15 (threshold value)
   4. Next
6. Configure notification if required (skipped)
7. Next, enter name and Create alarm
8. Check that alarm is created

# VPC with private and public subnets

Create a VPC with with one public and one private subnet in Singapore. The private subnet should be able to connect to the internet but not allow outside to come in.

1. Create a non-default VPC located in `ap-southeast-1` that is configured to host resources with a CIDR block of `10.0.0.0/16` and name it `td-vpc-lab`.

2. Create a public subnet called `td-public-subnet` in `us-west-1a` and assign the `10.0.0.0/24` CIDR block. Then, create a private subnet called `td-private-subnet` in `us-west-1b` and assign the `10.0.1.0/24` CIDR block.

3. Create a NAT gateway and assign an Elastic IP address.

4. Ensure that all EC2 instances launched in the `td-vpc-lab` will always be associated with a DNS hostname.

### VPC steps

1. Create a VPC with name and CIDR block
2. Actions -> Edit DNS hostnames -> Enable

### Subnets

1. Create both subnets
   1. Specify AZ
   2. CIDR block for subnet
2. Enable auto-assign public IPv4 address for public subnet

### IGW

1. Create IGW
2. Associate IGW with VPC

### Security Groups

1. 1 SG for instances in public subnet
   1. Allow ICMP from private subnet
   2. Allow SSH from my IP
2. 1 SG for instances in private subnet
   1. Allow ICMP from public subnet
   2. Allow SSH from public subnet

### IAM

1. Create role
2. Select AWS service -> EC2 -> Next
3. Search for policy `AmazonSSMManagedInstanceCore` -> Next
4. Enter name tags if needed -> Next -> Enter name

### NAT Gateway

1. VPC -> NAT Gateway -> Create NAT gateway
   1. Select public subnet
   2. Connectivity type: Public
   3. Click 'Allocate Elastic IP'
   4. Create

### Route tables

#### Public subnet route table

1. Enter name, VPC -> Create RT
2. Edit routes
   1. Point 0.0.0.0/0 to IGW
3. Associate with public subnet

#### Private subnet

1. Enter name, VPC -> Create RT
2. Edit routes
   1. Point 0.0.0.0/0 to NAT gateway

### EC2

1. Launch instances
2. For instance in private subnet, attach the IAM role for SSM

# EFS

### Creating EFS target mounts

1. EFS -> Create file system -> Customize

2. Select required options, then Next

3. Create mount target

   1. Choose subnets to create mount targets in
   2. Choose SG to be applied

4. File system policy (skip if already have policy to be applied)

5. Click Create

6. Go to Network tab, wait until Mount targets are Available

7. EFS -> File systems -> Get file system ID

8. Mount them within the Linux instances in `/etc/fstab` or click on Attach button in EFS to get Linux command to attach within EC2. Make sure the final argument, which is the mount point exists.


# KMS

Using customer-managed keys to encrypt data

### Create CMKs

1. KMS -> Customer-managed keys -> Choose symmetric (demo) or asymmetric
   1. Key material origin: KMS, External or CloudHSM
   2. Regionality: Single or multi (same key in multi regions)
   3. Next
2. Key administrative permissions: Determine which users can administer the key
3. Key usage permissions: Determine who can use the key (to decrypt/encrypt)
4. Review key policy and create key
5. Click on key and go to Key rotation tab, enable if needed

# EBS

This lab creates, mounts EBS volumes, unmounts them and mounts them on another target

## Creating EBS volumes

1. EC2 -> Volumes -> Create volume
2. Select specs as required, AZ
3. Create volume

## Attach to instances

Only can attach to instances in same AZ.

1. Right click -> Attach Volume
2. Enter `/dev/sdf` or path to attach within instance
3. Volume will be present in device path `/dev/xvdf`
4. To detach just right-click -> Detach volume

To mount:

```text
# Create XFS filesystem at /dev/xvdf
sudo mkfs -t xfs /dev/xvdf
# Check that it exists
sudo file -s /dev/xvdf
# Mount it at mount point
sudo mount /dev/xvdf /ebstest
```

## Attach to instance in different AZ

1. Right click on EBS volume in origin AZ -> Create snapshot
2. After creation 
   1. Right-click snapshot -> Create volume
   2. Select specs (can be different) and target AZ
3. Attach volume to instance
4. Also can copy snapshot to different region

## Create EBS encrypted volume

1. Same as creating EBS volume, just check EBS encryption
2. Snapshots taken same way, also encrypted (can't be changed)
3. Volumes created from snapshots also encrypted but can change CMK used

# EC2 AMI

## Creating AMI and launch instance from it

1. Shut down instance (stop in EC2)
2. Select instance -> Image and templates -> Create image
3. Select options -> Create image
   1. Snapshot of boot volume created first
   2. AMI created next
4. Right-click AMI -> Launch
5. Follow same steps as launching instance

## Copying and sharing AMI

1. Right-click AMI -> Copy AMI
2. Select destination region, specify encryption
3. Select Copy AMI

# AWS Fargate

Create a container of cats using Fargate

1. ECS -> Clusters -> Create cluster -> Networking only
2. Specify VPC -> Create
3. Click on created cluster
4. Task definition -> New task definition -> Fargate
   1. Specify name, task size (memory and vCPU)
5. Container definitions -> Add container
   1. Specify image URL (dockerhub), ports to expose -> Create
6. Click on cluster again -> Tasks tab -> Run new task
   1. Specify Fargate as launch type
   2. Select VPC and subnets to deploy in (ENI to be deployed in subnets)
   3. Edit SG to be applied to ENIs
   4. Run task
7. Wait till Last status becomes Running (same as Desired status)
8. Click on task -> Access it via public IP

## EC2 user-data

Bootstrapping with EC2 user-data

1. Launch EC2 instance
2. When specifying Instance details (eg. VPC etc.), scroll down to Advanced Details
3. Paste in user-data downloaded
4. Create instance

## EC2 instance roles

1. Before configuring, EC2 instance has no role configured

   ```text
   [ec2-user@ip-10-16-49-196 ~]$ aws s3 ls
   Unable to locate credentials. You can configure credentials by running "aws configure".
   ```

2. IAM -> Create role -> AWS service -> Next: Permissions

3. Search for policy **AmazonS3ReadOnlyAccess** and assign it to role.

4. Once created, right-click target EC2 instance -> Security -> Modify IAM role -> Select created role

5. Verify in Security tab of instance that the role is present

6. Go back to shell in EC2 instance, now `aws s3 ls` works

7. To detach IAM role redo step 4 but don't specify a role

# SM Parameter Store

1. Go to Systems Manager -> Parameter Store -> Create parameter

   ```text
   /my-cat-app/dbstring        db.allthecats.com:3306
   /my-cat-app/dbuser          bosscat
   /my-cat-app/dbpassword      amazingsecretpassword1337 (encrypted)
   /my-dog-app/dbstring        db.ifwereallymusthavedogs.com:3306
   /rate-my-lizard/dbstring    db.thisisprettyrandom.com:3306
   ```

2. For passwords that need encryption specify SecureString and KMS key to use

3. To interact at the command line do either top to specify exact name or an entire path (add decryption)

   ```text
   [cloudshell-user@ip-10-1-35-252 ~]$ aws ssm get-parameters --names /rate-my-lizard/dbstring
   {
       "Parameters": [
           {
               "Name": "/rate-my-lizard/dbstring",
               "Type": "String",
               "Value": "db.thisisprettyrandom.com:3306",
               "Version": 1,
               "LastModifiedDate": "2021-10-04T15:38:42.348000+00:00",
               "ARN": "arn:aws:ssm:us-east-1:265383945518:parameter/rate-my-lizard/dbstring",
               "DataType": "text"
           }
       ],
       "InvalidParameters": []
   }
   [cloudshell-user@ip-10-1-35-252 ~]$ aws ssm get-parameters-by-path --path /my-cat-app/ --with-decryption
   ```

# CloudWatch Agent

1. Install CW Agent on EC2 instance if not already present

   ```text
   wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
   sudo rpm -U ./amazon-cloudwatch-agent.rpm
   ```

2. Next we create an IAM instance role for the CW Agent to push to CW Logs

   1. IAM -> Create role -> AWS Service role (EC2)
   2. Select policies `CloudWatchAgentServerPolicy` and `AmazonSSMFullAccess`
   3. Create role

3. Attach IAM role to instance (right-click Security -> Modify IAM role

4. Enter instance to configure CW agent (a lot of steps here)

   ```text
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
   ```

5. Go to CloudWatch -> Log Groups, look for **/var/log/httpd/error_log** or other monitored logs configured above. Look for stream name as per instance ID

6. Also note you can find metrics in CWAgent that are available because of the installed CW Agent

# Route 53

## Configure R53 health checks

1. Route 53 -> Health check -> Create health check
2. Enter protocol (HTTP), IP address, host name (not required) and path to check (index.html)
3. When created, go to Health checkers tab to look at status of health checks

## Create Route 53 failover records

This creates a failover R53 record to a static S3 bucket website hosting

1. Route 53 -> Hosted zones -> Select hosted zone
2. Create record -> Failover record 
3. Specify record name (subdomain), DNS TTL
4. Click Define failover record
   5. Specify **IP address or another value**
   2. Elastic IP of primary resource
   3. **Primary record** type, health check (created above) 
5. Define another failover record,
   1. Specify **Alias to S3 website endpoint**, enter DNS address of bucket
   2. Select region, and S3 bucket
   3. Choose **Secondary record**

6. Check by failing the primary instance, check that health checks fail and it fails over

## Create Route 53 private hosted zone

1. Route -> Create hosted zone
   1. Enter private domain eg. somesite.sg
   2. Select private hosted zone
   3. Associate with VPC
2. Click created hosted zone to create 'A' record
   1. Create record
   2. Specify subdomain
   3. IP address to direct to, TTL and R53 routing policy
   4. Create record, now wait 5 min
3. Check within instance in VPC you can resolve private DNS query

# Databases

Migrate MariaDB running on EC2 instance to RDS instance

## Provision RDS instance

1. RDS -> Create DB subnet group
2. Choose VPC and its respective AZs
3. Pick the subnets the instance will be in, reference VPC
4. Databases -> Create database
   1. Specify engine type, version, DBMS credentials to use
   2. Specify storage (enable storage autoscaling if required)
   3. Specify subnet group created above, SG, database name
   4. Create DB
   5. Wait till complete
5. Edit assigned SG to allow EC2 instances on another SG inbound permissions to 3306
   1. Inbound rules: Type: MySQL/Aurora
   2. Select SG the EC2 instances belong to

## Migrate data to RDS instance

1. Click on RDS instance -> Connectivity & security tab. Note the DNS endpoint
2. Run command to migrate data on EC2 instance to RDS endpoint
3. Edit the app config so it points to the RDS endpoint

## Migrate from RDS to Aurora

1. Create snapshot of RDS instance
2. Click snapshot -> Actions -> Migrate snapshot
3. Specify
   1. DB engine version
   2. VPC, subnet group (create if non-existent), SG
   3. Create

Can also create from scratch like RDS

## Add read replicas for Aurora

1. RDS -> Select Aurora cluster (not endpoints) -> Actions -> Add reader
2. Create reader

## Migrate from Aurora to Aurora serverless

1. Take snapshot of Aurora instance (not RDS instance!). Migrate from RDS to Aurora first, then take screenshot using writer endpoint

2. Select Aurora snapshot -> Actions -> Restore snapshot

3. Select Serverless

   1. Also need specify VPC, subnet group
   2. Specify Aurora capacity units min, max
   3. And whether can pause if there's no load


## Database Migration Service from on-prem to RDS instance

Create the AWS RDS instance (target of DMS)

1. Create RDS subnet group
2. Provision RDS instance with subnet group and SG (see above)

Migrate the on-prem DB to RDS

1. Create the DMS subnet group (DMS -> Subnet groups -> Create subnet groups)
2. Select target VPC and subnets
3. Create Replication instance
   1. Select target VPC, replication subnet group
   2. Select single or multi AZ
   3. Create RI
4. Create source endpoint
   1. Endpoints -> Source endpoint
   2. Select source engine (source DBMS engine)
   3. Check 'Provide access information manually'
   4. Server name: IP address of the source on-prem DB
5. Create destination endpoint
   1. Specify target endpoint
   2. Select RDS instance -> Specify RDS instance created above
   3. Select DBMS engine of RDS instance
   4. Server name should automatically populate
   5. Enter username and password 
   6. Create endpoint
6. DMS -> Database migration tasks -> Create database migration task
   1. Specify Replication instance, source and destination endpoint
   2. Select migration type (demo 'Migrate existing data')
   3. Table mappings ->  Specify 'a4lwordpress' as DB to migrate
   4. Create task (will start immediately if enabled)

## RDS multi-AZ and recovering snapshots

1. Take snapshot of RDS instance
2. When complete go to Databases -> Click DB -> **Modify**
3. Check **Create a standby instance (recommended for production usage)** -> Continue -> Modify DB instance
4. When done, verify by Actions -> Reboot DB instance (with failover)
5. After several mins, check that the instance now shows a different AZ

## Restore DB from snapshot

1. RDS -> Snapshots -> Select snapshot -> Actions -> Restore snapshot
2. Specify DB instance name, VPC and SGs
   1. Try to pick the same instance class (**Include previous generation classes**)
   2. Restore and wait till complete
3. Edit the config file in the application to point to restored RDS instance



# Launch Templates

Create launch template

1. EC2 -> Create launch template
   1. Specify AMI to use, instance type, VPC, SG, IAM instance profile
   2. Enter user-data for bootstrapping if required 
   3. Create launch template
   4. When created, test with 'Launch instance from template'
   5. Network settings: Specify target subnet
   6. Launch instance from template

# Load Balancer

Create an ALB for load balancing. 

1. EC2 -> Create load balancer -> Application load balancer
   1. Specify Internet-facing or Internal
   2. Mappings: Specify AZs and public subnets to situate them in
   3. Specify SG to allow TCP 80 in
   4. Listeners and routing: HTTP 80. Click **Create target group**
      1. Target type: Instances, Protocol HTTP 80, Health checks: / path
      2. Register targets
      3. Create
   5. Refresh and select target group for LB
   6. Create LB
2. Once created, note the DNS name of the ALB

# Auto Scaling Group

1. EC2 -> Auto scaling groups -> Create auto scaling group
   1. Specify Launch template and version to use
   2. Next
   3. Select VPCs and subnets to launch instances in
   4. Next
   5. Select attach to existing load balancer (must do this)
   6. Specify desired, max and min capacity
   7. Next, until Create ASG

2. Click on created ASG -> Automatic scaling tab -> Create dynamic scaling policy

3. Select scaling type
4. Create CloudWatch alarm or use existing one
   1. Select metric -> EC2 -> By auto scaling group -> ASG name (CPU Utilization)
   2. Specify threshold greater than
   3. Specify notification if necessary, Next all the way
   4. Create alarm
5. Select created alarm, Take the action: Add 1 capacity units
6. Create it

# VPC endpoints

## Gateway endpoints

Create gateway endpoint for S3 bucket

1. VPC -> Endpoints -> Create endpoint
2. Look for `com.amazonaws.us-east-1.s3` in list
3. Select VPC and route tables which the instance is using
4. Specify policy (if you need to limit endpoint access to certain buckets)
5. Create endpoint
6. Click route table specified above, verify there's an entry similar to **pl-63a5400a**, leading to VPC EP
7. Enter the instance with Session Mgr, check you can list buckets with `aws s3 ls`

## Interface endpoints

Create interface endpoint for SNS

1. VPC -> Endpoints -> Create endpoint
2. Look for `com.amazonaws.us-east-1.sns` in list
3. Select VPC and subnets (not RTs unlike gateway EPs), just 1 subnet per AZ
4. Enable private DNS for endpoint, select SG, then create endpoint
5. Wait till available

# Egress-only IGW (IPv6)

1. VPC -> Egress-only IGW -> Create
2. Select VPC, then Create
3. Edit route table instance is using for subnet
4. Add `::/0` destination egress-only IGW
5. Test that  `ping -6 ipv6.google.com` works within instance

# Transit Gateway

Create transit gateway (just one will do)

1. VPC -> Transit gateway -> Create transit gateway
2. Leave default options -> Create
3. Wait till TGW is created

Create transit gateway attachments in each VPC you want to join to TGW

1. VPC -> Transit gateway attachment -> Create transit gateway attachment
   1. Specify created TGW above
   2. Attachment type: VPC
   3. Select VPC, and subnet to host attachment in
2. Create, repeat one for every VPC to be connected
3. Wait till all attachments created
4. Check in TGW Route tables that both attachments are associated, and both routes are propagated.

Lastly edit the route tables for each of the subnets the instances in VPCs A and B

1. Route table -> Add route for CIDR range of other VPC specify TGW attachment
2. Do the same for other VPC's route table

Now test you can ping the other VPC's instances

# Single Sign-On

Create a user Sally that has only billing access to AWS accounts in the org. Pre-req: Need to already have an organization created.

1. SSO -> Enable AWS SSO
2. Settings -> Ensure identity source is SSO
   1. Customize User Portal URL to what you want the user to type
   2. Configure MFA as required

Create the permission set or roles the user will be allowed to assume

1. AWS accounts -> Permission sets -> Create permission set
   1. Select existing job function policy (or create custom as required)
   2. Select **Billing** or as needed
   3. Next until Create
2. Once created, click each permission set to configure duration as needed

Create the user which will assume the permission set

1. SSO -> Users -> Add user
   1. Populate the user profile -> Next Groups
   2. Create group 'Billing'
   3. Add Sally to group
   4. Save sign-in instructions by copying
2. Click created group and check that Sally is a member

Assign the Billing group the permission sets for Billing

1. AWS accounts -> Select all the OUs in Organization -> Assign Users -> Groups tab
2. Select the Billing group -> Next Permission sets
3. Select Billing permission set -> Finish
4. When done, go to Dashboard and copy the User portal URL

5. Test by logging in with sally credentials copied
6. Test that you can view each AWS's account  -> Go to management console
7. Dropdown -> My billing dashboard (check you can view without errors)

Note that when adding user already signed in to groups with new permissions, need to logout and back in the see the changes.

# Advanced Site-to-Site VPN

## Create CGW for on-prem routers

1. VPC -> Customer Gateways -> Create customer gateway
   1. Enter public IP of on-prem router and BGP ASN
   2. Create CGW

## Create TGW attachment for on-prem routers to connect

For this step, you need to have a TGW already created

1. VPC -> TGW Attachments -> Create TGW attachment
   1. Transit GW ID: Select ID of TGW
   2. Attachment type: VPN
   3. Customer gateway: Select CGW created above
   4. Enable Acceleration
   5. Create TGW attachment
2. When done, go to Site-to-Site VPN Connections and check they're in pending state
3. Download configuration for each VPN connection

# Enable Session Manager

This works without having to enable NAT Gateway or make a subnet public

## Create VPC endpoints for SSM, EC2

1. VPC -> Endpoints -> Create endpoints
   1. Need these 3 interface endpoints in the subnets where the instances will be
      - com.amazonaws.[region].ec2messages
      - com.amazonaws.[region].ssmmessages
      - com.amazonaws.[region].ssm
   2. Add the private instance SG to the endpoints
   3. In the SG for the endpoints, include an inbound rule which allows TCP connections from all ports to itself and put the instances in the same SG.
2. Create an IAM role for permissions `AmazonSSMManagedInstanceCore`, attach to instance.
3. Make sure AMI supports it
4. Note you don't need IGW for Session Mgr, nor edit any route tables.

## Egress-Only IGW for IPv6

1. VPC -> Egress-only IGW -> Create Egress-only IGW
2. Attach to gateway
3. In route table of subnet of private instance add destination route for `::/0` to be egress-only IGW
4. Also make sure outbound rules allow ICMPv6
5. Test with `ping6 ipv6.google.com`
6. Note that you don't need to attach a regular IGW to the VPC

# CloudFront

## Add CDN to static website using S3

1. CloudFront -> Create CF distribution
   1. Select from **Origin domain** list the S3 bucket hosting the static website
   2. Choose origin path, OAI
   3. Select viewer protocol (HTTP, HTTPS)
   4. Specify other settings (such as alt CNAME)
   5. Default root object = **index.html**
   6. Create distribution
2. Click on distribution after creation, check General tab for distribution domain name and visit site

## Invalidate cached objects at edge

1. CloudFront -> Select distribution ID -> Invalidations tab
2. Specify object path`/*`  to invalidate everything or something more targeted.
3. Create invalidation, wait till complete and verify with CF link

## Add alt CNAME and SSL to CloudFront

1. Request cert from ACM (this requires CNAME record verification on your hosted domain)
2. If on R53, when done, create simple record, select **Alias to CloudFront distribution**, A record type

## Add OAI to CF distribution

Do this if you created the CF distribution without restricting origin access

1. CF distribution -> Origins tab -> Edit origin
2. Choose **Yes use OAI** -> **Create new OAI**
3. Let CF update bucket policy
4. Save changes

Now need to edit S3 bucket policy to remove this

```json
{
            "Sid": "PublicAccess",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cfands3-top10cats-151h000x5aa43/*"
},
```

which allows anyone to access it. Check that the CF distribution added this rule below

```json
        {
            "Sid": "2",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity E107S91V1H02J2"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cfands3-top10cats-151h000x5aa43/*"
        }
```

5. Check that you get HTTP 403 when you try to access S3 bucket URL directly