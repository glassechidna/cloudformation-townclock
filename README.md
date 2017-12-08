# CloudFormation Town Clock

## Why

Sometimes you have a [Lambda-backed CloudFormation custom resource][lambda-cfn] reference in your template.
CloudFormation only invokes the Lambda function when one of its inputs change. This is _usually_ the desired
behaviour, but sometimes you want the Lambda to be invoked every time the stack is updated. This is usually
achieved by passing a "dummy" parameter to the resource that changes each time: timestamps and incrementing
build numbers are popular choices. This project provides a simpler alternative.

[lambda-cfn]: http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources-lambda.html

## Usage

Firstly, create a new CloudFormationstack in your account based on [`townclock.yml`](/townclock.yml). 

Next, anywhere you have a Lambda-backed custom resource that you want invoked on every stack update, add 
the following:

```yaml
Parameters:
  TownClock:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /TownClock
    Description: Dummy parameter to keep things updating.
Resources:
  CustomResource:
    Type: Custom::YourResource
    Properties:
      ServiceToken: # your lambda arn
      # your normal properties..
      DummyProperty: !Ref TownClock
```

Tada, all done. Every stack update will now trigger a Lambda invocation.

## How

The [`townclock.yml`](/townclock.yml) CloudFormation template will create an SSM parameter called 
"TownClock" (by default), a Lambda function that updates it and a CloudWatch Events rule to execute 
that Lambda. Every time a stack is updated or a changeset is created, CloudFormation checks Parameter
Store to see if the value has changed. If it's been more than a minute since the last update, the 
Lambda will have updated the parameter's value.


