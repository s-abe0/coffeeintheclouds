---
layout: default
---

## CloudFormation

Infrastructure as code, written using either YAML or JSON. Tempates are uploaded to S3, no matter what. If imported through CloudFormation directly, its sent to S3 behind the scenes, and CloudFormation references it there.

Not every AWS service is supported, but can be worked around using Lambda Custom Resources

**Resources**
  * Required components that make up the core of the template; EC2, Security Groups, ASGs, Load Balancers, etc.
  * Resources can be declared and can reference each other using ```!Ref```
  * Identifiers take the form of ```AWS::aws-produce-name::data-type-name```
    
  ```
  Resources:
    MyInstance:
        Type: AWS::EC2::Instance
        Properties:
        AvailabilityZone: us-east-1a
        ImageId: ami-123456
        InstanceType: t2.micro
        SecurityGroups:
            - !Ref SSHSecurityGroup
            - !Ref ServerSecurityGroup

    # an elastic IP for our instance
    MyEIP:
        Type: AWS::EC2::EIP
        Properties:
        InstanceId: !Ref MyInstance

    # our EC2 security group
    SSHSecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
        GroupDescription: Enable SSH access via port 22
        SecurityGroupIngress:
        - CidrIp: 0.0.0.0/0
            FromPort: 22
            IpProtocol: tcp
            ToPort: 22
  ```
 
**Parameters**
  * Provide inputs to templates, allowing to reuse generic templates
  * Types of parameters include: String, Number, CommaDelimitedList, List<Type>
  * Parameters have Description, Contraints, ConstrainDescription, Min/MaxLength, Min/MaxValue, AllowedValues, and Defaults options
  * Parameters can be leveraged using the ```!Ref``` intrinsic function
  * AWS offers *Pseudo parameters*, which are enabled by default and contain specific data such as AccountId, Region, StackId, etc.
    * ```AWS::AccountId```
    * ```AWS::Region```

  ```
    Parameters:
        SecurityGroupDescription:
            Description: Security Group Description
            Type: String

    Resources:
        ServerSecurityGroup:
            Type: AWS::EC2::SecurityGroup
            Properties:
            GroupDescription: !Ref SecurityGroupDescription  # Reference the Parameter
            SecurityGroupIngress:
            - IpProtocol: tcp
                FromPort: 80
                ToPort: 80
                CidrIp: 0.0.0.0/0
            - IpProtocol: tcp
                FromPort: 22
                ToPort: 22
                CidrIp: 192.168.1.1/32
  ```

**Mappings**
  * Fixed variables within a template
  * Good to use if a value is already known
  * Can access map variables using the ```Fn::FindInMap``` function; ```!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]```

    ```
    Mappings: 
        RegionMap: 
            us-east-1:
                HVM64: ami-0ff8a91507f77f867
                HVMG2: ami-0a584ac55a7631c0c
            us-west-1:
                HVM64: ami-0bdb828fd58c52235
                HVMG2: ami-066ee5fd4a9ef77f1
            eu-west-1:
                HVM64: ami-047bb4163c506cd98
                HVMG2: ami-0a7c483d527806435
            ap-northeast-1:
                HVM64: ami-06cd52961ce9f0d85
                HVMG2: ami-053cdd503598e4a9d
            ap-southeast-1:
                HVM64: ami-08569b978cc4dfa10
                HVMG2: ami-0be9df32ae9f92309
    Resources: 
        myEC2Instance: 
            Type: "AWS::EC2::Instance"
            Properties: 
                ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", HVM64]
                InstanceType: m1.small
    ```

**Outputs**
  * Declares optional output values that can be imported into other stacks (templates)
  * Must be exported before being used by another stack
  * Useful in scenarios such as creating a network stack, then outputing VPC ID and Subnet ID variables to be used by another stack
  * Best way to perform collaboration across stacks
  * If outputs are being referenced by another stack, the output stack cannot be deleted

Exporting a Security Group
  ```
  Outputs:
    StackSSHSecurityGroup:
      Description: SSH Security Group
      Value: !Ref SSHSecGroup
      Export:
        Name: SSHSecurityGroup
  ```

Importing the security group in another template
  ```
  Resources:
    FirstInstance:
      Type: AWS::EC2::Instance
      Properties:
        AvailabilityZone: us-east-1a
        ImageId: ami-a4c7ebd2
        InstanceType: t2.micro
        SecurityGroups:
          - !ImportValue: SSHSecurityGroup
  ```

**Conditions**
  * Used to control creation of resources or outputs based on a condition
  * Conditions can be anything, but common ones are Environment (dev, test or prod), Region, some parameter value
  * Conditions can reference another condition, param value or a mapping
  * Logical intrinsic functions can be ```Fn::And, Fn::Equals, Fn:If, Fn::Not, Fn::Or```

Creating a condition
  ```
  Resources:
    MountPoint:
      Type: "AWS::EC2::VolumeAttachment"
      Condition: CreateProdResources

  Conditions:
    CreateProdResource: !Equals [ !Ref EnvType, prod ]  # Resolves to true if EnvType param is equals to 'prod'
  ```

The following resolves to true if the EnvType param is NOT equal to 'prod':
  ```
  CreateDevResource: !Not [ !Equals [ !Ref EnvironmentType, prod ]]
  ```

The ```Fn:If``` function returns one value if the condition is true, or another value if the condition is false:
  ```
    SecurityGroups:
      - !If [ CreateNewSecGroup, !Ref NewSecGroup, !Ref ExistingSecGroup ]
  ```

**Intrinsic Functions** 

*Fn::Ref* - Used to reference Parameters and Resources; returns value of param and physical ID of the resource (e.g. EC2 Id)

*Fn::GetAtt* - Used to get the attributes (properties) of a resource

  ```
  Resources:
    EC2Instance:
      Type: "AWS::EC2::Instance"
      Properties:
        ImageId: ami-123456
        InstanceType: t2.micro

    NewVolume:
      Type: "AWS::EC2::Volume"
      Condition: CreateProdResources
      Properties:
        Size: 100
        AvailabilityZone:
          !GetAtt: EC2Instance.AvailabilityZone  # Use !GetAtt to get AZ prop of EC2Instance resource
  ```

*Fn::FindInMap* - Return a value within a Mapping. See **Mappings** section above.

*Fn::ImportValue* - Imports value that were exported from other templates. See **Outputs** section above

*Fn::Join* - Joins a list of values with a delimiter. Works same as *join* in other languages. ```!Join [ ":" , [ a, b, c ]]``` outputs ```"a:b:c"```

*Fn::Sub* - Substitutes variables. Can combine Fn::Sub with references or Pseudo variables. Fn::Sub can return the string with the substituted values.
  ```
    !Sub
      - String
      - Var1Name: Var1Value
        Var2Name: Var2Value
  ```

  *String* is a string with variables, declared using ```${var}```, that are substituted at runtime. Variables can be template parameter names, resource logical IDs, resource attributes or a variable in a key-value map (Var1Name, Var2Name in the below key/value mapping).

  *VarName* - Name of a variable included in the *String* param
  *VarValue* - Value that will be substituted at runtime.

  Example using the key/value mapping and Fn::Ref function:

  ```
    Name: !Sub
      - www.${Domain}
      - { Domain: !Ref RootDomainName }
  ```

  Example without a mapping:

  ```
    !Sub 'arn:aws:ec2:${AWS::Region}:${AWS::AccountId}:vpc/${vpc}'
  ```


**CloudFormation Rollbacks**

If a stack creation fails, by default, everything gets rolled back (everything gets deleted). Look at the log to find out what happened. There is an option to disable rollback and troubleshoot what happened.

If stack update fails, everything gets rolled back to the previous known working state.

**Nested Stacks** - Stacks created as part of other stacks, by using the ```AWS::CloudFormation::Stack``` resource. Common components can be separated out, having dedicated templates. Then use the resource in your template to reference other templates, creating a nested stack.
  * Helpful when components must be re-used

**Cross Stacks** - Cross-stack references allow to use a layered or service-oriented architecture. Instead of including all resources in a single stack, create generic resources in separate stacks and use *Outputs* to cross-reference resources across stacks.
  * Helpful when stacks have different lifecycles or when you need to pass export values to many stacks

**Change Sets** - Allow to preview how proposed changes to a stack may impact running resources. Changes are made only when the change set is executed. Change sets do not indicate whether a stack will successfully update; e.g. if a resource doesn't support updates, or if you have insufficient permissions to modify a resource. 

**StackSets** - Extends the functionality of stacks by enabling to create, update and delete stacks across multiple accounts and regions with a single operation, using an admin account. 
  * Trusted accounts can create, update and delete instances from StackSets.
  * When a stack set is updated, all associated stack instances are updated through all accounts and regions