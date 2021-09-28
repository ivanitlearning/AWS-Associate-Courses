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
4. 