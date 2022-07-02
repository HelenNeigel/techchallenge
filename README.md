# Tech Challenge

Deploying Docker image of Web Application to AWS ECS cluster with Auto Scaling connected to HA Postgres RDS DB

## Prerequisites

1. Docker Image is ready and available from DockerHub
2. Docker is installed
2. AWS CLI is installed

## Architecture

## Instruction

1. Upload docker image to ECR: create repository, authenticate docker with AWS and push tagged docker image
```console
#download image from DockerHub
docker pull <image_name>
#create repository on ECR (note <repository_uri> and <regitry_uri> from output)
aws ecr create-repository --repository-name images-tech --region us-east-2
#authenticate docker with AWS 
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin <registry_uri>
#tag image for ECR
docker tag <image_name> <repository_uri>/images-tech:latest
#push image to ECR
docker push <repository_uri>/images-tech:latest
```

2. Clone current repo and Run the following command from WD to create CloudFormation stack
```console
aws cloudformation create-stack --stack-name ecs-db-stack --template-body file://ecs-db-stack.yaml --parameters ParameterKey=EcrImageUri,ParameterValue=<ecr_image_URI>  --capabilities CAPABILITY_IAM
```

3. Run command to wait for stack completion
```console
aws cloudformation wait stack-create-complete --stack-name ecs-db-stack
```

4. Get SNS topic from stack description (Output section)
```console
aws cloudformation describe-stacks --stack-name ecs-db-stack
```

5. Trigger Lambda Function which prepare DB with SNS message  
```console
aws sns publish --topic-arn <arn_sns_topic> --message "DB Set Up!"
```

## Improvement Considerations

1. CloudFormation template and instructions can benefit from parametrising more values
2. Instuctions should be run from a tool of choice (e.g Jenkins) on a trigger (e.g DockerHub push)
3. Elastic IP can be introduced, which can be reassigned each time to a new Load Balancer upon successful deployment of a new version of stack to insure no downtime
