How to create an ECS cluster
============================

### Manual

1. Open https://eu-west-1.console.aws.amazon.com/ecs/home?region=eu-west-1#/
2. Click "create cluster"
3. Set name of the cluster
TODO: Create ASG
4. Done


### Using the AWS-SDK

```Bash
aws ecs create-cluster --cluster-name ecs-demo-awscli-ecscluster
aws ec2 create-security-group --group-name ecs-demo-awscli-sg --description "ecs demo awscli" --vpc-id vpc-1a2b3c4d

aws autoscaling create-launch-configuration \
    --launch-configuration-name ecs-demo-awscli-launchconfig \
    --key-name pgarbe \
    --image-id ami-c74127b4 \
    --instance-type t2.micro \
    --iam-instance-profile ecsInstanceRole \
    --security-groups sg-1a2b3c4d \
    --user-data file://ecs-demo-awscli-userdata.txt

aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name ecs-demo-awscli-asg \
    --launch-configuration-name ecs-demo-awscli-launchconfig \
    --min-size 1 --max-size 3 \
    --vpc-zone-identifier subnet-1a2b3c4d
```
See http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-optimized_AMI.html for ECS optimized AMIs per region


Cleanup
```
    aws autoscaling delete-auto-scaling-group --auto-scaling-group-name ecs-demo-awscli-asg --force-delete
    aws autoscaling delete-launch-configuration --launch-configuration-name ecs-demo-awscli-launchconfig
    aws ec2 delete-security-group --group-name ecs-demo-awscli-sg
    aws ecs remove-cluster --cluster-name ecs-demo-awscli-ecscluster 
```

* AutoScaling can be configured
* Lot of (dependend) commands to execute


### Using ECS-Cli
```
ecs-cli configure --cluster ecs-demo-ecscli --region eu-west-1
ecs-cli up --keypair pgarbe --capability-iam --size 2 --instance-type t2.micro
```

Cleanup
```
ecs-cli down --force
```

* Only one command
* Manual scaling is possible
* AutoScaling needs to be configured manually
* Can't change instance types later
* Uses CloudFormation under the hood (for cluster creation)

### Using CloudFormation 

```
aws cloudformation create-stack \
    --stack-name ecs-demo-cloudformation \
    --capabilities CAPABILITY_IAM \
    --parameters ParameterKey=KeyName,ParameterValue=pgarbe ParameterKey=SubnetID,ParameterValue=subnet-1a2b3c4d \
    --template-body file://ecs-demo-cloudformation.json
```

Cleanup
```
aws cloudformation remove-stack --stack-name ecs-demo-cloudformation
```

* Configuration as Code
* AutoScaling can be configured
* Like awscli




