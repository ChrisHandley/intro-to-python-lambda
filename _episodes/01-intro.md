---
title: CloudFormation Fundamentals
teaching: 10
exercises: 10
questions:
- "What is the SAM Template"
- "Why do we use it" 
objectives:
- "Build resource definitions"
- "Basics of Infrastructure as Code"
keypoints:
- "A lot is often boilerplate"
---

## Getting Started


~~~
mkdir dojo-python-lambda

cd dojo-python-lambda

sam init
~~~
{: .language-bash}

With sam init select option 1.

Then select the Hello World Example

Select No for the runtime option.

Then select option 9 - Python3.9

Then select option 1 - zip

Then just press enter for the default application name.

## The sam-app structure

Within the `sam-app` directory we have a few files and directories.

sam-app/
├── hello_world
│   ├── __init__.py
|   ├── app.py
│   └── requirements.txt
├── events
|   └── events.json
├── tests
│   ├── __init__.py
│   ├── requirements.txt
│   ├── integration
|   |   ├── __init__.py
|   |   └── test_handler.py
|   └── unit
|       ├── __init__.py
|       └── test_api_gateway.py
├── __init__.py
├── template.yaml
└── README.md

The first thing we will need to do is activate the premade environment.

~~~
source env/bin/activate

pip list
~~~
{: .language-bash}

Using pip list we can see we have a few things preinstalled for us. Notably aws lambda powertools, aws xray sdk, and boto3. These are libraries for enabling debugging and tracing of lambda actions in the cloud, and boto3 is a python library that has apis for many AWS services such as S3, EC2, Glue, and Athena.

## Build and Locally Run Lambda

Core to our Lambda function is that infrastructure is code. And this the `template.yaml`

~~~
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  sam-app

  Sample SAM Template for sam-app

# More info about Globals: https://github.com/awslabs/serverless-application-model/blob/master/docs/globals.rst
Globals:
  Function:
    Timeout: 3

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.9
      Architectures:
        - x86_64
      Events:
        HelloWorld:
          Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
          Properties:
            Path: /hello
            Method: get

Outputs:
  # ServerlessRestApi is an implicit API created out of Events key under Serverless::Function
  # Find out more about other implicit resources you can reference within SAM
  # https://github.com/awslabs/serverless-application-model/blob/master/docs/internals/generated_resources.rst#api
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
  HelloWorldFunction:
    Description: "Hello World Lambda Function ARN"
    Value: !GetAtt HelloWorldFunction.Arn
  HelloWorldFunctionIamRole:
    Description: "Implicit IAM Role created for Hello World function"
    Value: !GetAtt HelloWorldFunctionRole.Arn
~~~
{: .language-yaml}

Let's break this down.

The head of the file establishes the Template Format Version and how the yaml is translated and interpreted by Cloudformation. This is boilerplate and we leave it as is.

The Description is just a string, where we describe what is being built by the template.

We then define some `Globals` parameters. These set common configurations for resource types, while within a resource you can define more specifics. In this case we set the timeout time for the functions, and for the Api we set that tracing is enabled.

`Parameters` is where we can define some parameters that are used within the template. We can define them here and set defaults, as these values can be specified as part of the SAM deploy CLI command e.g. in our example the `DomainName` is has a `Type` and a `Default`. In essence this is like defining environment variables for the CloudFormation template.

`Resources` is where we begin to define the resources that build our service. This consists of functions, S3 buckets, queues, api gateways, and more. For this tutorial we will focus just on two types that allow us to build our Serverless function.

`Outputs` is where we define the names of the resources made. This defines them so they can be referenced elsewhere outside of this stack.

## Lambda Function

White space in our yaml file matters. So note the tab indentations.

Our function is first defined with a name. This is a logical name that we can use in the template when we need to reference the function, as this doesn't need to be the same as the resource name, which must be unique on the AWS platform account. Generally each setting/property/thing is define by the name of it, and a colon.

With the name define we assert the `Type`, which is in this case the `AWS::Serverless::Function`.

This particular type of service is then defined by `Properties`. The first is the `FunctionName` which is a unique name for the function on the AWS account. The `CodeUri` is the location in the local space where the code is deployed to, where the Lambda function is found.

The `Handler` is the function name within the code that interprets the event being passed to it. Note the naming convention. In this case the code being looked for is the file `app` in the directory defined by `CodeUri`. This file in our case is a python file. In that python file the Lambda function is called lambda_handler and so to reference that function we use the file name and function name concatenated by a dot. e.g. `app.lambda_handler`.

`Runtime` defines for CloudFormation what language the code is in and what basics to use for intepreting the code. In this case, python 3.9. `Tracing` is Active so we can trace and track events, i.e. see when this function is invoked. This is critical in very large services where many functions are being invoked, and AWS can track these and procude pathways of events as they trigger lambdas that trigger other functions or resources.

### Events

Lambda Functions are invoked with events, that either come through some Api Gateway, or from some messaging queue, or via some other program running on AWS e.g. Elastic Beanstalk. A function can manage multiple events, and each can be given a logical name, which can be reused in the template.

The event has a `Type`, which in this case is delivered by the Api. The `Properties` of the function include the `Path` that the event is passed down - the Lambda function can have multiple end points that trigger, and which is used is determined by the handler and the event - the handler is acting as a filter of the event types. `Method` defines if the function is being invoked with a Get or Post (or similar) method via HTTP. `RestApiId` is the id of the ARN, in this case a RestApi resource. It is typical, as in this case, to define the Api in the same template.

## Api Gateway

The Gateway is handling calls to our Lambda function by HTTP requests. Again like any resource we give it a logical name we can use elsewhere in the template. The `Type` is `AWS::Serverless::Api`, and it has properties. The `StageName` is an important property so we can define some resources to operate on our development, staging and production stages, so that different end points are available and to prevent issues with CI/CD.

The `Domain` defines the URL of the Api Gateway. The `DomainName` is the url for the domain, where we wil lbe sending our requests. The `CertificateArn` is a Amazon Resource Name (ARN) of an AWS managed certificate this domain name's endpoint. AWS Certificate Manager is the only supported source. `Route53` defines Amazon Route 53 configuration. `HostedZoneId` is the ID of the hosted zone that you want to create records in. Route53 is a AWS service that provides the Domain Name System for our services. In essence we are defining how we go from connecting from a URL and thus a domain name, to get through the Api and send our request to the Lambda function or some other service.

## Refering Resources

We have a few ways we can access resources in our template (and outside of it), and some of these ways make our life easier and the code more compact.

`!Sub ${Parameter}`

This allows a paramter to be passed into a string via substitution.

Alternatively we can reference the resource directly in the template and grab an attribute associated with it.

`!GetAtt LogicalName.AttributeName`

Finally we may just want to reference something by arn, in which cas we provide the full arn name, using the Substition and Get Attribute functions above to insert and replace parts of the arn string to form the true string.

{% include links.md %}
