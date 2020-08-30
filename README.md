# Deploy ECS Fargate Cluster with ecs-cli

Just working on the command line, we create an ECS Fargate cluster and run a container.

[Full description](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-cli-tutorial-fargate.html)

## 10 Steps to see a container running on ECS Fargate

* Prerequisites: set env variables to access AWS account and region
```
AWS_SECRET_ACCESS_KEY=...
AWS_SECRET_ACCESS_KEY=...
AWS_DEFAULT_REGION=eu-west-1
```

* Step 1: Create the Task Execution IAM Role
```
aws iam --region $AWS_DEFAULT_REGION create-role --role-name ecsTaskExecutionRole --assume-role-policy-document file://task-execution-assume-role.json

aws iam --region $AWS_DEFAULT_REGION attach-role-policy --role-name ecsTaskExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

* Step 2: Configure the Amazon ECS CLI
```
ecs-cli configure --cluster tutorial --default-launch-type FARGATE --config-name tutorial --region $AWS_DEFAULT_REGION

ecs-cli configure profile --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY --profile-name tutorial-profile
```

* Step 3: Create a Cluster and Configure the Security Group
```

ecs-cli up --cluster-config tutorial --ecs-profile tutorial-profile

# set env variables to values returned from previous command
VPC_ID=...
SUBNET1_ID=...
SUBNET2_ID=...

aws ec2 describe-security-groups --filters Name=vpc-id,Values=$VPC_ID --region $AWS_DEFAULT_REGION

# set env variables to values returned from previous command
SG_ID=...

aws ec2 authorize-security-group-ingress --group-id $SG_ID  --protocol tcp --port 80 --cidr 0.0.0.0/0 --region $AWS_DEFAULT_REGION
```

* Step 4: Create a Docker compose file (file is provided, change region if you want to) and configure parameters
```
# set subnets and sg in ecs_params.yml to the values we got earlier
```

* Step 5: Deploy the Compose File to a Cluster
```
ecs-cli compose --project-name tutorial service up --create-log-groups --cluster-config tutorial --ecs-profile tutorial-profile
```

* Step 6: View the Running Containers on a Cluster
```
ecs-cli compose --project-name tutorial service ps --cluster-config tutorial --ecs-profile tutorial-profile

# set TASK_ID to the output of previous command
TASK_ID=... (e.g. b88cdee1-bf2e-487c-a03b-aec4be1ac689, first value until '/')
```

* Step 7: View the Container Logs
```
ecs-cli logs --task-id $TASK_ID --follow --cluster-config tutorial --ecs-profile tutorial-profile
```

* Step 8: Scale the Tasks on the Cluster
```
ecs-cli compose --project-name tutorial service scale 2 --cluster-config tutorial --ecs-profile tutorial-profile

ecs-cli compose --project-name tutorial service ps --cluster-config tutorial --ecs-profile tutorial-profile
```

* Step 9: View your Web Application
```
# open your browser and check container is running using ip of any task (should return a congratulations page)
http://...
```

* Step 10: Clean Up
```
ecs-cli compose --project-name tutorial service down --cluster-config tutorial --ecs-profile tutorial-profile

ecs-cli down --force --cluster-config tutorial --ecs-profile tutorial-profile
```
