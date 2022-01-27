# HerbsAndSpices

These useful small bits of code will help you save time and get the most out of AWS.

Set your `AWS_ACCOUNT_ID` to a bash variable:

```
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity \
--query Account --output text)
```

Get the most recently created CloudWatch log group name:
```
aws logs describe-log-groups --output=yaml \
--query 'reverse(sort_by(logGroups,&creationTime))[:1].{Name:logGroupName}'
```

Tail the logs for the CloudWatch group:
```
aws logs tail <<LOGGROUPNAME>> --follow --since 10s
```

Delete all log groups that match a text pattern and prompt yes/no for confirmation:
```
aws logs describe-log-groups | \
jq ".logGroups[].logGroupName" | grep -i <<pattern>> | \
xargs -p -I % aws logs delete-log-group --log-group-name %
```

Stop all running instances for your current working Region (H/T: Curtis Rissi):
```
aws ec2 stop-instances \
--instance-ids $(aws ec2 describe-instances \
--filters "Name=instance-state-name,Values=running" --query "Reservations[].Instances[].[InstanceId]"
--output text | tr '\n' ' ')
```

Determine the user making CLI calls:
```
aws sts get-caller-identity --query UserId --output text
```

Generate YAML input for your CLI command and use it:
```
aws ec2 create-vpc --generate-cli-skeleton yaml-input > input.yaml
#Edit input.yaml - at a minimum modify CidrBlock, DryRun, ResourceType, and Tags
aws ec2 create-vpc --cli-input-yaml file://input.yaml
```

List the AWS Region names and endpoints in a table format:
```
aws ec2 describe-regions --output table
```

Find interface VPC endpoints for the Region you are currently using:
```
aws ec2 describe-vpc-endpoint-services \
--query ServiceDetails[*].ServiceName
```

Populate data into a DynamoDB table:
```
aws ddb put table_name '[{key1: value1}, {key2: value2}]'
```

Determine the current supported versions for a particular database engine (e.g., aurora-postgresql):
```
aws rds describe-db-engine-versions --engine aurora-postgresql \
--query "DBEngineVersions[].EngineVersion"
```

Delete network interfaces associated with a security group and prompt for each delete (answer yes/no to delete or skip):
```
aws ec2 describe-network-interfaces \
--filters Name=group-id,Values=$SecurityGroup \
--query NetworkInterfaces[*].NetworkInterfaceId \
--output text | tr '\t' '\n' | xargs -p -I % \
aws ec2 delete-network-interface --network-interface-id %
```

Find your default VPC (if you have one) for a Region:
```
aws ec2 describe-vpcs --vpc-ids \
--query 'Vpcs[?IsDefault==`true`]'
```

Enable encryption by default for new EBS volumes in a Region:
```
aws ec2 enable-ebs-encryption-by-default
```

List all AWS Regions:
```
aws ssm get-parameters-by-path \
--path /aws/service/global-infrastructure/regions \
--output text --query Parameters[*].Name | tr "\t" "\n"
```

List all AWS services:
```
aws ssm get-parameters-by-path \
--path /aws/service/global-infrastructure/services \
--output text --query Parameters[*].Name \
| tr "\t" "\n" | awk -F "/" '{ print $6 }'
```

List all services available in a region (e.g., us-east-1):
```
aws ssm get-parameters-by-path \
--path /aws/service/global-infrastructure/regions/us-east-1/services \
--output text --query Parameters[*].Name | tr "\t" "\n" \
| awk -F "/" '{ print $8 }'
```

List all Regions that have a particular service available (e.g., SNS):
```
aws ssm get-parameters-by-path \
--path /aws/service/global-infrastructure/services/sns/regions \
--output text --query Parameters[*].Value | tr "\t" "\n"
```

Create a presigned URL for an object in S3 that expires in a week:
```
aws s3 presign s3://<<BucketName>>/<<FileName>> \
--expires-in 604800
```

Find Availability Zone IDs for a Region that are consistent across accounts:
```
aws ec2 describe-availability-zones --region $AWS_REGION
```

Set the Region by grabbing the value from an EC2 instance’s metadata:
```
export AWS_DEFAULT_REGION=$(curl --silent http://169.254.169.254/latest/dynamic/instance-
identity/document \
| awk -F'"' ' /region/ {print $4}')
```
