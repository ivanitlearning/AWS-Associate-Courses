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