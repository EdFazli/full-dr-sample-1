# full-dr-sample-1

Template for full DR automation using cloudformation, lambda, cloudwatch events

## Workflow ideas

Cloudwatch events triggered (Successful snapshot created)-> Snapshot copied to DR region -> Notified user -> Cloudwatch events triggered (Successful snapshot copied) -> Copied snapshots tagged -> Notified user

## Reference

[aws-step-functions-ebs-snapshot-mgmt](https://github.com/aws-samples/aws-step-functions-ebs-snapshot-mgmt)

## Configuration Options

There are a few different configuration options for controlling the Snapshot Management architecture.  

1. **Configure Notifications for Failures**: Whenever a failure is detected in the state machine execution (or AWS Lambda functions it invokes) the state machine executes a Lambda function that sends a notification to an [Amazon Simple Notification Service](https://aws.amazon.com/sns/) (SNS) topic. To receive emails when a failure occurs, [add an email subscription](http://docs.aws.amazon.com/sns/latest/dg/SubscribeTopic.html) to the **SnapshotMgmtTopic**.

2. **Configure Volumes to Include**: If you only want certain volumes to be included in the snapshot management workflow, you can specify a Tag key that a volume must have in order for it to be included.  If a Tag key isn't specified, then the
snapshot management will take place for all snapshot creations.  If you would like to specify a Tag key, either:

    - After the CloudFormation stack has completed in your primary region.  Follow
 these steps to modify it:
    - Go to **Services** -> **Lambda**
    - Select the **TagSnapshots** function
    - On the **Code** tab, scroll to the bottom and in the **Environment Variables**
    section, fill in the tagKey environment variable with the value of your tag key
    you want to perform snapshot management for.
    - Edit the PrimaryRegionTemplate.yaml prior to deployment (if you are following the steps in the section that describes **How to customize and run the architecture in your account**).

 You will modify the tagKeyValue Default value in that file.

 ```yaml
 tagKeyValue:
   Description: 'The value for the key tag that you want all volumes to have for the snapshot management to apply.'
   Type: 'String'
   Default: 'none'
 ```

## Runbook

BELOW COMMANDS NEED TO BE PERFORM IN LOCAL PC WITHIN THE FOLDER CONTAINS ALL THE CODE

1. Creates an S3 bucket for staging code in Primary region (ap-southeast-1)
    `aws s3api create-bucket --bucket dr-test-sg-region --region ap-southeast-1 --create-bucket-configuration LocationConstraint=ap-southeast-1`
2. Package the code and upload to S3 (SG Region)
    `aws cloudformation package --template-file PrimaryRegionTemplate.yaml --s3-bucket dr-test-sg-region --output-template-file tempPrimary.yaml --region ap-southeast-1`
3. Deploy the cloudformation stack
    `aws cloudformation deploy --template-file tempPrimary.yaml --stack-name PrimaryRegionSnapshotManagement --capabilities CAPABILITY_IAM --region ap-southeast-1`
4. Creates an S3 bucket for staging code in DR region (ap-southeast-2)
    `aws s3api create-bucket --bucket dr-test-sydney-region --region ap-southeast-2 --create-bucket-configuration LocationConstraint=ap-southeast-2`
5. Package the code and upload to S3 (Sydney Region)
    `aws cloudformation package --template-file DR_RegionTemplate.yaml --s3-bucket dr-test-sydney-region --output-template-file tempDR.yaml --region ap-southeast-2`
6. Deploy the cloudformation stack
    `aws cloudformation deploy --template-file tempDR.yaml --stack-name DRRegionSnapshotManagement --capabilities CAPABILITY_IAM --region ap-southeast-2`
