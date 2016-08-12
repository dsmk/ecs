# How to deploy an application to ECS?

* Initial deployment
* Updating

### ECS-Cli

```
ecs-cli compose --file hello-world.yml service up
```

Nice, but application is not reachable. Let's try it again
```
# create task definition for a docker container
CLUSTER_NAME=ecs-demo-ecscli
ecs-cli compose --file compose.yml --project-name $CLUSTER_NAME --verbose create

# create elb & add a dns CNAME for the elb dns
aws elb create-load-balancer --load-balancer-name "$CLUSTER_NAME" --listeners Protocol="TCP,LoadBalancerPort=8080,InstanceProtocol=TCP,InstancePort=8080" --subnets "$SUBNET_ID" --security-groups "$ECS_SECURITY_GROUP" --scheme internal

# create service with above created task definition & elb
aws ecs create-service --service-name "ecscompose-service-$CLUSTER_NAME" --cluster "$CLUSTER_NAME" --task-definition "ecscompose-$CLUSTER_NAME" --load-balancers "loadBalancerName=$CLUSTER_NAME,containerName=demo-service,containerPort=8080" --desired-count 1 --deployment-configuration "maximumPercent=200,minimumHealthyPercent=50" --role ecsServiceRole
```

* No built-in support for ELB
* `compose` is built on top of docker-compose (which often includes multiple containers) but scope of `compose scale` is always the whole service not a single task.
=> Picture which explains service, task, compose, container, ...


### CloudFormation
* No direct connection between dockerfile, docker-compose and cloudformation (tooling)


## Updating an ECS cluster

* How will containers be re-scheduled?
