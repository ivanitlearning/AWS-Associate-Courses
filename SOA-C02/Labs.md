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

1. Create roles allowing Lambda function to access EFS
2. Create access point for Lambda to access EFS
   1. EFS -> Click on file systems -> Access points tab -> Create access point
      * Fill in user ID and group ID
      * Set POSIX permissions for root directory
   2. Create access point
3. Create Lambda function
   1. Edit general config
   2. Edit timeout (max 15 min)
4. Go to Configuration -> VPC tab
   1. Select VPC, subnets, SG
   2. Lambda will create 2 ENI inside selected subnets
   3. Save
5. Go to Configuration -> File systems -> Add file system
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
   2. Select 'Enable for all accounts in organization'
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

