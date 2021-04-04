---
layout: default
---

## AWS CICD

AWS provides various automation tools for deploying applications. Some of these tools include CodeCommit, CodePipeline, CodeBuild and CodeDeploy. All these tools work together to accomplish Continuous Integration/Continuous Delivery.

*Continuous Integration* - Devs push code often to a repo where a build & test server checks the code and delivers immediate feedback to the developer. This helps to find bugs early, make faster deliveries and more regular deployments.

*Continuous Delivery* - Ensures that deployments happen quickly and often to move away from the older 'release once every month' models. CD includes tools such as CodeDeploy, Jenkins, Spinnaker, etc.

Tech Stack for CICD on AWS:
  * CodeCommit - A code repository service similar to github (actually uses git).
  * CodeBuild - A fully managed build service that compiles source code, runs unit tests and produces artifacts. CodeBuild removes the need to provision and manage build servers (such as Jenkins).
  * CodeDeploy - A deployment service that automates app deployments to EC2 instances, on-premises servers, Lambda functions or ECS. 
  * CodePipeline - A CICD service used to model, visualize and automate app deployment steps. Encompasses CodeCommit, CodeBuild, and CodeDeploy.

#### CodeCommit

Basically the AWS version of Github, CodeCommit is an AWS hosted Git repository service that is fully managed and highly available. 

CloudWatch Events can be set up to send email alerts anytime pull requests are created, a commit happens, etc.

**CodeCommit Security** - Integrated with IAM and files in transit (HTTPS or SSH) and at rest are encrypted using KMS.
  * AWS Users can configure SSH keys in the IAM console to gain access to the repo. Users can also create HTTPS credentials.
  * HTTPS credentials can also be generated through the CLI using Authentication helper.
  * MFA can be enabled for an added layer of security
  * IAM policies are used to manage acceess and authorizations

**Cross Account Access** - CodeCommit allows cross account access. To enable this, AWS STS (with AssumeRole API) and an IAM Role are needed. To allow users from AccountB access to a repo in AccountA, an admin in AccountA must:
  1. Create a policy in AccountA that grants access to the repo
  2. Create a role in AccountA that can be assumed by IAM users and groups in AccountB
  3. Attach the policy to the role

Then, an AccountB admin must create a group with a policy that allows group members to assume the role created in AccountA.

Users in AccountB cannot user SSH keys or Git creds to access repos in another account. Either *git-remote-codecommit* or the credential helper must be used. 

**CodeCommit Notifications** - Can create notifications or triggers in CodeCommit to use SNS (Simple Notification Service), Lambda or CloudWatch Event Rules after certain actions

*Triggers* are commonly configured to send emails when someone pushes to a repo or to notify an external build system. Triggers use either SNS or Lambda functions and are limited to operational events.

Use cases for SNS / Lambda triggers:
  * Deletion of branches
  * Trigger for pushes that happens in master branch
  * Notify external build system
  * Trigger Lambda function to perform codebase analysis (check for secret credentials)

*Notifications* are used to send emails when an action is taken that affects other users, such as when one user comments on a commit. Notifications use CloudWatch Events, and can trigger an SNS topic.

Use cases for CloudWatch Events:
  * Trigger for pull requests
  * A user comments on a commit
  * Event Rules goes into an SNS topic

#### CodeBuild

Features:
  * A fully managed build service that can replace other build tools such as Jenkins, removing the need to manage more instances. 
  * Comes with continuous scaling (no servers to manage)
  * Pay for usage; the time it takes to complete the builds. No need to pay to run EC2 instances hosting Jenkins.
  * Secure integration with KMS for encryption of build artifacts, IAM for permissions, VPC for network security, and CloudTrail for API calls logging.
  * Leverages Docker under to hood, and allows to extend capabilities by using a custom Docker image for builds.
  * Outputs logs to S3 & CloudWatch
  * Can use CloudWatch alarms to detect failed builds and trigger notifications
  * Comes with various supported environments such as Java, Ruby, Python, Nodejs, etc. Can use Docker to extend any environment needed

CodeBuild gets build commands from a special file called ```biuldspec.yml```, located at the root of the source code directory (this can be overridden, however). The buildspec file is a collection of build commands and related settings. It can contain the following:

    ```
    version: 0.2

    env: # Define environment variables such as secure secrets (SSM Parameter store)

    phases: 
        install:
            runtime-versions:
                nodejs: 10
            commands:
                - echo "installing something"
        pre_build:
            commands: 
                - echo "we are in the pre build phase"
        build:
            commands:
                - echo "we are in the build block"
                - echo "we will run some tests"
                - grep -Fq "Congratulations" index.html
        post_build:
            commands:
                - echo "we are in the post build phase"
                
    artifacts: # What to upload to S3 (encrypted with KMS)
    
    cache: # Files to cache (dependencies) to S3 for future build speedup
    ```

**Phases**
  * Install - Install any dependencies
  * Pre-build - Any commands to execute before build
  * Build - Actual build commands
  * Post-build - Finishing touches; e.g. zip output

CodeBuild can be ran locally in case of troubleshooting beyond the logs. For this, the CodeBuild Agent is required.

**CodeBuild in VPC** - By default, CodeBuild containers are launched outside the VPC. Therefore, the build cannot access any resources within the VPC. A CodeBuild project can be given VPC configuration to allow running the containers within the VPC. Config needed is VPC ID, Subnet Ids and Security Group Ids. 

#### CodeDeploy

A deployment service that automates deployments to EC2, on-premise servers, Lambda functions or ECS. Can deploy content such as code, Lambda functions, web & config files, executables, packages, scripts, and multimedia files. Content can be on S3, GitHub, or Bitbucket. CodeDeploy is platform-agnostic and works with any application.

CodeDeploy can minimize downtime by performing in-place rolling updates, taking a specified amount of instances offline at a time. Blue/Green deployments can also be used (only with EC2), where the latest content is deployed on replacement instances. If any errors occur, deployments can be stopped and rolled back.

EC2 instances are grouped in *deployment groups*, using Tags. Deployment groups can contain individually tagged EC2 instances, EC2 instances within an ASG, or both.

Each EC2 instance or on-premise server must be running the CodeDeploy agent which continuously polls CodeDeploy for work to do. When a push to the Github repo or S3 bucket happens, a deployment is triggered.

A special file called ```appspec.yml``` must be within the root of the source directory, which holds instructions for the deployment. It manages each deployment as a series of lifecycle event hooks as defined in the file. The structure of the file is as follows:
  * File section - How to source and copy the files from S3/github to the instances filesystem

    ```
    files:
    - source: source-file-location
        destination: destination-file-location
    ```

  * Hooks - Set of instructions on how to perform the deployment. The order is as follows:
    * ApplicationStop - Occurs before the app revision is download. Can gracefully stop the app or remove installed packages. This hook does not run until the second deployment, when the appspec.yml file has been downloaded.
    * DownloadBundle - CodeDeploy agent copies app revisio files to temporary location. This even is reserved for CodeDeploy agent and cannot be used to run scripts.
    * BeforeInstall - Used for preinstall tasks, such as creating backups or decrypting files
    * AfterInstall - Used for postintall tasks such as configuration or changing file permissions
    * ApplicationStart - Used to restart services stopped during ApplicationStop
    * ValidateService - Used to validate that everything is working properly

Not all steps need to be defined, but order is important. 

    ```
    hooks:
    deployment-lifecycle-event-name:
        - location: script-location
        timeout: timeout-in-seconds
        runas: user-name
    ```

**CodeDeploy primary components**
  * Application - Unique name for the application
  * Comput platform - EC2, On-Premise of Lambda
  * Deployment configuration - Rules for success/failures. For EC2/On-Premise, can specify minimum healthy instances. For Lambda, specify how traffic is routed to updated Lambda functions.
  * Deployment group - Group of tagges instances
  * Deployment type - In place or Blue/Green
  * IAM instance profile - Need to give EC2 permissions to pull from S3 or Github
  * Application Revision - app code and appspec.yml file
  * Service Role - An IAM role for CodeDeploy to perform tasks
  * Target revision - Target deployment app version

**Deployment Configuration**
  * Configs can be one instance at a time, where if one instance fails the deployment stops.
  * Can be half at a time
  * All at once; quick but comes with downtime. Good for dev
  * Can set a custom number of minimum health hosts (e.g. 75%)

*Deployment Config Failures*
  * If a deployment fails, instances stay in 'failed state'
  * New deployments are first deployed to 'failed state' instances
  * To rollback, redeploy old deployment or enable automated rollback

*Deployment Config Targets*
  * Can be set of EC2 instances with tags
  * Can be directly to an ASG
  * Can be a mix of ASG and Tags to allow deployment in segments
  * Customization in scrips with DEPLOYMENT_GROUP_NAME environment variables

#### CodePipeline

CodePipeline allows to model, visualize and automate the steps needed to pull, build and deploy an application. Similar to Jenkins pipelines, but integrates well with AWS services. CodePipeline has *stages* where a specific *action* is performed, such as pulling code, building, deploying, etc.

**Stages** - Made up of a series of serial or parallel actions that perform tasks on the application artifacts. Artifacts created from one stage are stored in an S3 bucket chosen when the pipeline was created, where they are then accessed by the next stage.

** Action Groups go within stages. One stage can have multiple action groups, executed sequentially. One action group can have multiple actions, executed in parallel.

*Source* - The stage where code is pulled from a repo. Can be from CodeCommit, S3, GitHub, ECR or Bitbucket.
*Build* - The stage where the code is built and tested. This can tie into CodeBuild or Jenkins.
*Deploy* - The stage where the build artifact is deployed. This can be CodeDeploy, Elastic Beanstalk, CloudFormation, ECS, etc.

**Troubleshooting**

Pipeline state changes happen in *AWS CloudWatch Events*, which can create SNS notifications. Events can be created for failed pipelines or cancelled stages. 

If a stage failes, the entire pipeline stops. Information can be obtained from the console.

AWS CloudTrail can be used to audit AWS API calls.

If an action cannot be performed, check IAM permissions. IAM Service Role may not have proper permissions.

#### CodeStar

An integrated solution for creating, managing and working with software projects. A CodeStar project creates and integrates AWS services such as CodeCommit, GitHub, CodeBuild, CodeDeploy, etc. into a project toolchain. It also manages permissions required for project users (team members). 

CodeStar features:
  * Start new software projects quickly using templates for web applications, services, etc.
  * Manage project access
  * Visualize, operate and collaborate in one place with a project dashboard
  * Iterate quickly