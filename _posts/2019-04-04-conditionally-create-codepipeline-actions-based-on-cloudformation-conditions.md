---
layout: post
title:  "Conditionally create CodePipeline Actions based on Cloudformation Conditions"
categories: aws devops
---

You can accomplish this by conditionally by inserting the AWS::CodePipeline::Pipeline Resource's Action into the Actions list using the Fn::If Intrinsic Function referencing your Conditions element, returning the Action when the Condition is true and AWS::NoValue (which removes the property, in this case removing the item from the list) when it is not true.

For example if you pass a value that indicates if a particular element, such as Redis cluster Cloudformation should be included in a deployment when the pipeline is ran:

```yaml
Parameters:
  CreateRedis:
    Type: String
    Description: Do you want to create a redis? (yes/no)
    Default: "no"
    AllowedValues: ["yes", "no"]
```

Then setup a conditional based on it:

```yaml
Conditions:
  NeedsRedis: !Equals [ !Ref CreateRedis, "yes" ]
```

This element in the Actions Section of the CodePipeline Resource will toggle based on the state of the condition:

```yaml  
- !If 
- NeedsRedis
- Name: !Sub "${ApplicationName}-redis"
  RunOrder: 2
  InputArtifacts:
    - Name: Build
  ActionTypeId:
    Category: Deploy
    Owner: AWS
    Version: '1'
    Provider: CloudFormation
  Configuration:
    ActionMode: REPLACE_ON_FAILURE
    RoleArn: !Sub "${CodePipelineCloudFormationRole.Arn}"
    Capabilities: CAPABILITY_NAMED_IAM
    StackName: !Sub "${ApplicationName}-redis"
    TemplatePath: Build::cloudformation/redis.cform
    ParameterOverrides: !Sub |
      {
        "ApplicationName": "${ApplicationName}"
      }
- !Ref AWS::NoValue
```
