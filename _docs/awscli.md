---
layout: default
---

## AWS CLI Basics

To set up the aws CLI on a personal machine, run ```aws configure```, and input the account access key and secret access key and the region of which you want to work in. The access key and secret key can be generated from IAM User account management under Access Keys. If no default Region is set, *us-east-1* will be chosen by default. 

**Note:** NEVER configure an access key on an EC2 instance! This is super insecure; access keys should only be used on personal computers. Instead, create an IAM Role and assign it to the EC2 instance.

**CLI Dry Runs**

Dry runs can be used for testing purposes (checking for permissions, results of a command, etc.). Some commands (not all) contain a ```--dry run``` argument which allows to simulate the command without it actually taking affect.

Example command to run which would create an EC2 instance:

```aws ec2 run-instances --dry-run --image-id ami-0be2609ba883822ec --instance-type t2.micro```

This command will return either an UnauthorizedOperation error if the user does not have permissions to run this command, or a DryRunOperation error if the command would have succeeded.

**CLI STS Decode**

Some CLI calls can return an encoded message in response to an AWS operation error, most likely an authorization error. Not all operations will return an encoded error message however. 

To decode these message, the ```sts decode-authroization-message``` command must be used. Example:

```aws sts decode-authorization-message --encoded-message <value>```

Note that the user must have proper permissions to run the sts command, which can be set in the IAM policy. The Action permission required is the ```sts:DecodeAuthorizationMessage``` write action.

The command returns the decoded message in JSON format.

**CLI Profiles**

If a user has multiple accounts, they can configure multiple *profiles* in their aws cli environment. This can be achieved using the ```aws configure``` command by using the ```--profile <profile name>``` argument, like so:

```aws configure --profile second-account```

Then enter the acces key, secret key and default region. To use this profile in another command operation, use the ```--profile second-account``` argument, and the command will run using the *second-account* account.

**CLI with MFA**

Multi-Factor Authentication can be used with the CLI by using the ```sts get-session-token``` command. Running this command will return a temporary access key, secret key and session token. This information is then used to create a profile. The session token must then be appended in the *credentials* file under the created profile, with the key ```aws_session_token```. 

Example command to get temporary session information:

```aws sts get-session-token --serial-number <arn of registered mfa device> --token-code <mfa code>```

**Signature Version 4**

When sending HTTP requests to AWS, authentication information must be included within the request. This is achieved by using Signature Version 4 (SigV4), a process used to add authentication information to the HTTP request. The process uses an access key and secret key of an AWS users credentials.

If using the AWS CLI or SDK to make requests, the requests are automatically signed by the tools themselves using the configured credentials. If manually creating an HTTP request, the request must be manually signed using SigV4.