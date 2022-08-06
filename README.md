# full-dr-sample-1

Template for full DR automation using cloudformation, lambda, cloudwatch events

## Workflow ideas

Cloudwatch events triggered (Successful snapshot created)-> Snapshot copied to DR region -> Notified user -> Cloudwatch events triggered (Successful snapshot copied) -> Copied snapshots tagged -> Notified user

## Reference

[aws-step-functions-ebs-snapshot-mgmt](https://github.com/aws-samples/aws-step-functions-ebs-snapshot-mgmt)

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
