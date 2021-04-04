---
layout: default
---

## Elastic Beanstalk

An AWS service for deploying and scaling web applications and services. Elastic Beanstalk is essentially a one stop shop for deploying an entire web application infrastructure stack, as it manages the EC2 instances, Load Balancers, auto scaling groups, health monitoring, and deployments. Web applications can be Java, .NET, PHP, Node.js, Python, Ruby, Go, and Docker, hosted on popular platforms such as Apache, Nginx, Passenger and IIS. All one needs to do is upload a zip file of the application code. The underlying resources (EC2 instances, load balancers, etc.) are fully retained and accessible by the user.

Benefits of Elastic Beanstalk:
  * Fast and simple - Very fast and easy to stand up a web application in minutes. Can use the AWS console, a Git repo, or an IDE such as Eclipse to upload an application and Elastic Beanstalk handles the rest.
  * Developer productivity - Since Elastic Beanstalk handles the infrastructure, more time can be spent on development. The underlying platform running the application is kept up to date with latest patches and updates.
  * Automatic scaling - The application is scaled up and down using adjustable Auto Scaling settings.
  * Full resource control - You are given the freedom to choose the underlying resources, such as EC2 type.

Three architecture models of Beanstalk:
  1. Single Instance - Contains one EC2 instance with an Elastic IP address. Does not have a load balancer or an auto scaling group. Good for development and testing.
  2. Load-balanced, scalable - Used load balancers and auto-scaling groups with EC2 instances. Good for production.
  3. Worker tier ASG only - Uses only auto-scaling groups with no load balancer. This is used in the worker tier, and is great for non-web apps in production.

#### Concepts

*Source Bundle* - The packaged contents of an application, such as a zip file for NodeJS or Python applications, or a WAR file for Java applications.

Elastic Beanstalk manages all the recourses that run an *application* as *envrionments*.

*Application* - A collection of components including envrionments, versions and envrionment configurations. An Application contains multiple environments.

*Application version* - A specific, labeled iteration of deployable code for an application which points to an S3 object. The S3 object contains the deployable code, such as a Java WAR file. 

*Environment* - A collection of AWS resources running an application version. Can deploy multiple environments when multiple applicaiton versions need to be ran; for example, dev, test and prod environments.

*Environment Tier* - Designates the type of application being ran in the environment, and what resources Elastic Beanstalk will provision to support it. *Web Server environment tier* will contain applications such as REST application or web pages. *Worker environment tier* will contain backend applications.

*Environment configuration* - A collection of parameters and settings that define how an environment and its resources behave. If the configuration settins are changed, Elastic Beanstalk automatically applys the change by updating or re-deploying resources.

Elastic Beanstalk allows *cloning* an environment with the exact same configuration, which can be useful for deploying a test version of an application to the production environment. All resources and configurations are preserved (RDS database data is not preserved, however).

*HTTPS in Beanstalk* - To enable HTTPS in a Beanstalk environment, need to load SSL certificate onto the Load Balancer. This can in a few ways:
  * From the EB console, under load balancer configuration
  * From the ebextensions code; e.g. *.ebextensions/securelistener-alb.config*
  * Redirect HTTP to HTTPS by configuring instances to do so, or configure the ALB with a rule. Make sure health checks are not redirected so they continue to return 200 OK

SSL Certificates can be provisioned using AWS Certificate Manager or the CLI, and security group must be configured to allow incoming on port 443.

*Beanstalk Custom Platforms* - An advanced concept which allows to define an environment from scratch. Should be used if an app language is not supported within Beanstalk. Custom Platforms allow to define the OS, additional software and script that beanstalk runs on the platforms. 

Platforms are created by defining an AMI using a Platform.yaml file, and is built using *Packer* software, which is an open source tool used to create AMIs. 

#### Deployment Policies

Elastic Beanstalk can deploy updated code to an environment by uploading a new source bundle. Each deployment is identified by a Deployment ID, starting at 1 and incrementing by one with each deployment.

There are various deployment policies:

  * All at once - The quickest method where Elastic Beanstalk deploys the new application to all the instances. However, downtime may occur as the servers may need to restart.
  * Rolling - Takes longer to deploy, but avoids downtime and minimizes reduced availability. The application is deploy to the environment one batch of instances at a time; i.e. with 6 instances, two instances will be deployed to at a time, for a total of 3 rolling deployments. With this deployment, two verison of the application will be running at the same time. This method is good if you can't accept any periods of lost service. 
  * Rolling with additional batch - Longer deployment time than Rolling, but avoids any reduced availability. During this deployment, Elastic Beanstalk creates an extra batch of instances, then performs a rolling deployment. This method encurs a small cost due to the additional instances, but should be used if the same bandwitdh and availability must be maintained.
  * Immutable - A slower deployment method that deploys to new instances instead of updating existing onces. A second Auto Scaling group is created for the new instances, and traffic is served using both new and old instances until the new instances pass health checks. Incurs more costs, but is great for Prod deployments, as rollbacks are supported incase of a deployment failure.
  * Blue/Green - More of a manual deployment method and not directly supported in Beanstalk. Involes creating a second 'staging' environment (green) and deploying the new version there. This second environment can take a small portion of traffic for testing, using Route 53 weighted policies. Once the new environment is good, URLs can be swapped, and the new environment becomes the main environment.

#### Lifecycle Policies

Elastic Beasntalk can store at most 1000 application versions, and if old versions are not removed, then you can't deploy anymore. Lifecycle policies can be used to manage old versions; to either remove them or store them in S3. Policies can be based on time (remove after n days), or space (keep up to 200 versions).

#### Elastic Beanstalk Extensions

Elastic Beanstalk configuration files can be included in application source code to configure and customize the environment. These files are contained within the ```.ebextensions/``` folder in the root of the source bundle. Files need to end in ```.config``` and must be in either YAML or JSON format. 

eb extensions allows to modify defult settings using ```option_settings```, such as environment variables, or add extra resources such as RDS, ElastiCache, DynamoDB, etc.

Example of adding environment variables for connecting to a database:

    ```
    option_settings:
    aws:elasticbeanstalk:application:environment:
        DB_URL: "jdbc:postgresql://rds-url.com/db"
        DB_USER: admin
    ```

This format may also be seen:
    ```
    option_settings:
    - namespace:  namespace
        option_name:  option name
        value:  option value
    ```

**Quick note:** Elastic Beanstalk relies on *CloudFormation* to provision its resources, and .ebextensions allows to define CloudFormation resources to provision anything, such as ElastiCache, an S3 bucket, etc.


#### Elastic Beanstalk Migrations

Some scenarios may require a Beanstalk migration; creating a new environment with slight changes and removing the old environment. Once such scenario is a Load Balancer migration. The current environment has a Classic LB, and a Application LB is now required. Cloning cannot be used for this, so the following steps must be performed:
  1. Create new environment with same configurations, but with an ALB
  2. Deploy the new application onto the new environment
  3. Shift traffic to the new environment by performing a CNAME swap or Route 53 update

Another scenario is de-coupling an RDS database from a Beanstalk environment (the RDS database will get removed if the Beanstalk environment is removed)
  1. Create a snapshot of the DB
  2. Protect the DB from deletion by enabling deletion protection
  3. Create new Beanstalk environment, without RDS, and point the app to the RDS DB
  4. Perform a CNAME swap or Route 53 update
  5. Terminate the old environment. Since the RDS DB has deletion protection, it will remain.
  6. Since the RDS DB was not removed, the CloudFormation template will display errors. Need to delete the CloudFormate stack.