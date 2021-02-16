---
layout: default
---

## IAM (Identity and Access Management)

* Handles all AWS security - Users, Groups, Roles
  * Users - Physical person
  * Groups - Functions (admins, devops), Teams (engineering); contains users
  * Roles - Allows to grant permissions to entities, such as an IAM user in another account, application code, an AWS service such as EC2, etc.

* Policies define what can and cannot be done; written in JSON
* Root account should never be used
* IAM Federation – Used by large enterprises; allows Active Directory integration


**Credentials provider precedence**

Certain environments have specific precendence chains for using access credentials.

*AWS CLI* - The CLI will look for credentials in the following order:
  1. Command line options: ```--region, --output, --profile```
  2. Environment Variables: ```AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_SESSION_TOKEN```
  3. CLI credentials file: ```~/.aws/credentials```
  4. CLI configuration file: ```~/.aws/config```
  5. Container credentials for ECS tasks
  6. Instance profile credentials for EC2 Instance Profiles

*AWS SKD* - The AWS SDK (e.g. Java SDK) will look for credentials in the following oder:
  1. Environment variables
  2. Java system properties: ```aws.accessKeyId, aws.secretKey```
  3. Default credential profiles file: ```~/.aws/credentials```, shared by many SDK
  4. Amazon ECS container credentials
  5. Instance profile credentials used on EC2 instances

**Credentials Best Practices**

  * Never store credentials in code
  * Best practice is for credentials to be inherited from the credentials chain
  * If working within AWS, use IAM Roles
    - EC2 Instance Roles for EC2 Instances
    - ECS Roles for ECS tasks
    - Lambda Roles for Lambda functions
  * If working outside AWS (e.g. in the CLI), use environment variables or named profiles