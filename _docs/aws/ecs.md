---
layout: default
---

## Elastic Container Service (ECS)

ECS Clusters are logical grouping of EC2 instances all running the ECS agent (docker). The ECS agent registers the EC2 instance with the ECS cluster. The EC2 instances run a special AMI specifically for ECS.

**ECS Task Definitions** - JSON metadata that tells ECS how to run a docker container. Task Definitions contain:
  - Image Name
  - Port Binding for container to host
  - Memory and CPU required
  - Environment variables
  - Networking information

Task definitions can be given a Task Role, which is an IAM role that will be used by the containers to give them specific permissions. Used to allow tasks to make API requests to AWS services.

Task definitions also contain a Container definition, which contains information for Docker, such as the image, port mapping, etc.

**ECS Service** - Created at the Cluster level, services define how many tasks should run, how they should be run, and ensures the desired number of tasks are running within the cluster. Services can be linked to Load Balancers if needed.

*Service Scheduling* - There are two servcie scheduler strategies:
  - Replica - Places and maintains the desired number of tasks across the cluster. For instance, five tasks can be ran (5 containers) on two EC2 instances; 3 containers on one, 2 contains on antoher.
  - Daemon - Deploys exactly one task on each active container instance that meets the task placement contraints.

*Service load balacing* - Services can be registered with a load balancer to allow dynamic port mapping for multiple tasks on the same EC2 instance. ECS automatically maps the dynamic ports of the tasks with the load balancer, so no Target Group is needed (although a dummy target group will need to be created to create the load balancer). Load balancers can only be registered with a service during service creation.

**ECS Task Placement Strategy** - An algorithm for deciding how to place tasks on EC2 instances, or for task termination. Can be specified when running a task or creating a new service, and can be updated for existing services.

Strategy types:
  1. binpack - Tasks placed on instances so as to leave the least amount of unused CPI or memory; crams containers onto one instance until its at near full capacity. When terminating tasks, termination is based on which instance will have the most available resources left after task termination.

  ```
    "placementStrategy": [
        {
            "field": "memory",
            "type": "binpack"
        }
    ]
  ```

  2. random - Tasks are placed randomly.

  ```
    "placementStrategy": [
        {
            "type": "random"
        }
    ]
  ```

  3. spread - Tasks are placed evenly based on a specified value such as instanceId or availability-zone. Tasks are terminated in a manner that maintains balance across AZs. Within an AZ, tasks are selected at random.

  ```
    "placementStrategy": [
        {
            "field": "attribute:ecs.availability-zone",
            "type": "spread"
        }
    ]
  ```

Placement strategies can be mixed together. The following strategy distributes tasks evenly across AZs, then bin packs based on memory:
```
    "placementStrategy": [
        {
            "field": "attribute:ecs.availability-zone",
            "type": "spread"
        },
        {
            "field": "memory",
            "type": "binpack"
        }
    ]
```

**ECS Auto Scaling**

*ECS Service Auto Scaling* - Automatically scales tasks based on CloudWatch metrics. The following types of autoscaling are supported:
  * Target Tracking - Scales off a target value for a specific metric.
  * Step Scaling - Scales based on CloudWatch alarms. Scales off a set of scaling adjustments (step adjustments), that vary based on size of the alarm breach.
  * Scheduled Scaling - Scale based on a specific date and time.

When creating a new Service, can switch to Capacity Provider strategy to tie in with a Capacity Provider.

*Capacity Providers* - Ties into an EC2 Auto Scaling group to manage auto-scaling of EC2 instances. Can set a Target Capacity in the Capacity provider to indicate when to scale out or scale in; e.g. Scale out when capacity hits 70%.

**ECS IAM Roles** - Various IAM roles are created to allows ECS clusters and its components (services, tasks, instances) to perform the jobs they need to. 
  * ecsinstanceRole - Gives proper permissions for the ECS Agents running within the EC2 instances.
  * ecsServiceRole - Gives proper permissions to ECS Services mostly for load balancing, and for some EC2 tasks.
  * AWSServiceRoleForECS - Allows ECS to create EC2 network interfaces, connect to load balancers, route 53, etc.
  * ecsTaskExecutionRole - Gives the proper permissions to tasks for them to run properly; e.g. get ECR images and create logs.
  * Custom task roles - Should create custom roles for each ECS task definition to increase security.

#### Elastic Container Registry (ECR)

ECR is an AWS container registry to hold custom docker images for use in ECS clusters.

Access is controlled through IAM. If there are permission errors (e.g., cluster instances cant pull the image), this is likely due to IAM role issues.

In order to push to an ECR registry (for AWS CLIv2), need to pipe the login password into the docker command, like so:
```aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 136707440429.dkr.ecr.us-east-1.amazonaws.com```

#### Fargate

Farget is a serverless compute engine for containers that works with ECS and EKS, removing the need to provision and manage servers. It lets you specify and pay for resources per application and improves security through application isolation.

Benefits of Fargate:
  * Allows to focus on building applications, not infrastructure. Removes operational overhead of scaling, patching, securing and managing servers.
  * Launches and scales the compute to closely match specified resource requirements for the container, allowing for flexible pricing.
  * Individual ECS tasks or EKS pods run on their own dedicated kernel runtime environment and do not share CPU, memory, storage or network resources, ensuring workload isolation for better security.
  * Out-of-the-box observability with other AWS services such as CloudWatch Container Insights.

Setup is similar to basic ECS, but theres no EC2 instances. A load balancer is still required for multiple containers of the same task, and a security group is configured for the Fargate cluster.