---
title: Cloud Deployment
teaching: 20
exercises: 10
questions:
- "Can I run my lambda function locally?"
- "Can I send information to my lambda function?"
objectives:
- "Perform local testing."
- "Perform Get and Post requests."
keypoints:
- "AWS Powertools make life dead easy for Python to process events."
---

## Setup
To use SAM CLI to deploy to AWS we need to have our AWS CLI configured.

~~~
aws configure
~~~
{: .language-bash}

Using the above command means you will be prompted to give the information to access an AWS account. This includes;

- region
- output
- aws_access_key_id
- aws_secret_access_key

You will also be prompted to assign this information to a `profile` name.

The first two bits of information will be stored in `.aws\config` while the latter parts are stored in `.aws\credentials`.

For our work we are assuming we are using the general account for training that we have stored as `default`

~~~
export AWS_PROFILE=default
~~~
{: .language-python}


## Build and Deploy in the Cloud
In the head directory of our Lambda function application, we build the application.

~~~
sam build
~~~
{: .language-bash}

If we are deploying by hand (like we are now) we can use the guided deploy option

~~~
sam deploy --guided
~~~
{: .language-bash}

This will prompt us for a number of settings. Defulat values are used from the `samconfig.toml` if present.

- Stack Name
- AWS Region


Always as best practice confirm changes before deploying.

To use the Lambda function the IAM roles will need to be created if not already defined. SAM will make these for us. 

Don't disable rollback and allow authorization to not be defined.

The output should be similar to the following.

~~~
Deploying with following values
        ===============================
        Stack name                   : cmh-dojo-test
        Region                       : eu-west-2
        Confirm changeset            : True
        Disable rollback             : False
        Deployment s3 bucket         : aws-sam-cli-managed-default-samclisourcebucket-v0frhvjxchny
        Capabilities                 : ["CAPABILITY_IAM"]
        Signing Profiles             : {}

Initiating deployment
=====================
Uploading to cmh-dojo-test/41197cdc70cabe0377bb98e643f29f62.template  1481 / 1481  (100.00%)

Waiting for changeset to be created..

CloudFormation stack changeset
---------------------------------------------------------------------------------------------------------------------------------------------
Operation                                LogicalResourceId                        ResourceType                             Replacement                            
---------------------------------------------------------------------------------------------------------------------------------------------
+ Add                                    HelloWorldFunctionByeWorldTestPermissi   AWS::Lambda::Permission                  N/A                                    
                                         onProd                                                                                                                   
+ Add                                    HelloWorldFunctionHelloWorldNamePermis   AWS::Lambda::Permission                  N/A                                    
                                         sionProd                                                                                                                 
+ Add                                    HelloWorldFunctionHelloWorldPermission   AWS::Lambda::Permission                  N/A                                    
                                         Prod                                                                                                                     
+ Add                                    HelloWorldFunctionRole                   AWS::IAM::Role                           N/A                                    
+ Add                                    HelloWorldFunction                       AWS::Lambda::Function                    N/A                                    
+ Add                                    ServerlessRestApiDeploymenteffb3069d5    AWS::ApiGateway::Deployment              N/A                                    
+ Add                                    ServerlessRestApiProdStage               AWS::ApiGateway::Stage                   N/A                                    
+ Add                                    ServerlessRestApi                        AWS::ApiGateway::RestApi                 N/A                                    
---------------------------------------------------------------------------------------------------------------------------------------------

Changeset created successfully. arn:aws:cloudformation:eu-west-2:003415847847:changeSet/samcli-deploy1667988620/671eb532-882d-4dee-816d-f6e8508bf0b2


Previewing CloudFormation changeset before deployment
======================================================
Deploy this changeset? [y/N]: y

2022-11-09 10:10:28 - Waiting for stack create/update to complete

CloudFormation events from stack operations
---------------------------------------------------------------------------------------------------------------------------------------------
ResourceStatus                           ResourceType                             LogicalResourceId                        ResourceStatusReason                   
---------------------------------------------------------------------------------------------------------------------------------------------
CREATE_IN_PROGRESS                       AWS::IAM::Role                           HelloWorldFunctionRole                   Resource creation Initiated            
CREATE_IN_PROGRESS                       AWS::IAM::Role                           HelloWorldFunctionRole                   -                                      
CREATE_COMPLETE                          AWS::IAM::Role                           HelloWorldFunctionRole                   -                                      
CREATE_IN_PROGRESS                       AWS::Lambda::Function                    HelloWorldFunction                       -                                      
CREATE_IN_PROGRESS                       AWS::Lambda::Function                    HelloWorldFunction                       Resource creation Initiated            
CREATE_COMPLETE                          AWS::Lambda::Function                    HelloWorldFunction                       -                                      
CREATE_IN_PROGRESS                       AWS::ApiGateway::RestApi                 ServerlessRestApi                        -                                      
CREATE_IN_PROGRESS                       AWS::ApiGateway::RestApi                 ServerlessRestApi                        Resource creation Initiated            
CREATE_COMPLETE                          AWS::ApiGateway::RestApi                 ServerlessRestApi                        -                                      
CREATE_IN_PROGRESS                       AWS::Lambda::Permission                  HelloWorldFunctionHelloWorldNamePermis   -                                      
                                                                                  sionProd                                                                        
CREATE_IN_PROGRESS                       AWS::ApiGateway::Deployment              ServerlessRestApiDeploymenteffb3069d5    -                                      
CREATE_IN_PROGRESS                       AWS::Lambda::Permission                  HelloWorldFunctionHelloWorldPermission   -                                      
                                                                                  Prod                                                                            
CREATE_IN_PROGRESS                       AWS::Lambda::Permission                  HelloWorldFunctionByeWorldTestPermissi   Resource creation Initiated            
                                                                                  onProd                                                                          
CREATE_IN_PROGRESS                       AWS::Lambda::Permission                  HelloWorldFunctionByeWorldTestPermissi   -                                      
                                                                                  onProd                                                                          
CREATE_IN_PROGRESS                       AWS::Lambda::Permission                  HelloWorldFunctionHelloWorldNamePermis   Resource creation Initiated            
                                                                                  sionProd                                                                        
CREATE_IN_PROGRESS                       AWS::Lambda::Permission                  HelloWorldFunctionHelloWorldPermission   Resource creation Initiated            
                                                                                  Prod                                                                            
CREATE_IN_PROGRESS                       AWS::ApiGateway::Deployment              ServerlessRestApiDeploymenteffb3069d5    Resource creation Initiated            
CREATE_COMPLETE                          AWS::ApiGateway::Deployment              ServerlessRestApiDeploymenteffb3069d5    -                                      
CREATE_IN_PROGRESS                       AWS::ApiGateway::Stage                   ServerlessRestApiProdStage               -                                      
CREATE_IN_PROGRESS                       AWS::ApiGateway::Stage                   ServerlessRestApiProdStage               Resource creation Initiated            
CREATE_COMPLETE                          AWS::ApiGateway::Stage                   ServerlessRestApiProdStage               -                                      
CREATE_COMPLETE                          AWS::Lambda::Permission                  HelloWorldFunctionByeWorldTestPermissi   -                                      
                                                                                  onProd                                                                          
CREATE_COMPLETE                          AWS::Lambda::Permission                  HelloWorldFunctionHelloWorldNamePermis   -                                      
                                                                                  sionProd                                                                        
CREATE_COMPLETE                          AWS::Lambda::Permission                  HelloWorldFunctionHelloWorldPermission   -                                      
                                                                                  Prod                                                                            
CREATE_COMPLETE                          AWS::CloudFormation::Stack               cmh-dojo-test                            -                                      
---------------------------------------------------------------------------------------------------------------------------------------------

CloudFormation outputs from deployed stack
---------------------------------------------------------------------------------------------------------------------------------------------
Outputs                                                                                                                                                          
---------------------------------------------------------------------------------------------------------------------------------------------
Key                 HelloWorldFunctionIamRole                                                                                                                    
Description         Implicit IAM Role created for Hello World function                                                                                           
Value               arn:aws:iam::003415847847:role/cmh-dojo-test-HelloWorldFunctionRole-1E5R14DJP457P                                                            

Key                 HelloWorldApi                                                                                                                                
Description         API Gateway endpoint URL for Prod stage for Hello World function                                                                             
Value               https://yqk76gxlnc.execute-api.eu-west-2.amazonaws.com/Prod hello/                                                                           

Key                 HelloWorldFunction                                                                                                                           
Description         Hello World Lambda Function ARN                                                                                                              
Value               arn:aws:lambda:eu-west-2:003415847847:function:EquansHelloWorld                                                                              
---------------------------------------------------------------------------------------------------------------------------------------------
~~~
{: .language-bash}

## Invoke Cloud 

We can now invoke the function using the value returned for the API Gateway i.e. the URL. If we repeat the steps we did in local testing, we should achieve the same results.


~~~
sam local start-api
~~~
{: .language-bash}

When you run this command in a directory that contains your serverless functions and your AWS SAM template, it creates a local HTTP server that hosts all of your functions. It starts a Docker container locally to invoke the function. It reads the CodeUri property of the AWS::Serverless::Function resource to find the path in your file system that contains the Lambda function code.

With the local server running and the Lambda function available we can invoke it.

~~~
curl http://127.0.0.1:3000/hello
~~~
{: .language-bash}

This may differ if you are using Windows, Mac, or using Postman or similar GUI based request application.

~~~
Invoking app.lambda_handler (python3.9)
Skip pulling image and use local one: public.ecr.aws/sam/emulation-python3.9:rapid-1.37.0-x86_64.

Mounting /home/chandley/Equans/equans-power-tools/.aws-sam/build/HelloWorldFunction as /var/task:ro,delegated inside runtime container
START RequestId: f1d7a5f7-9fc6-4847-8ad0-a02343dbad26 Version: $LATEST
{"level":"INFO","location":"decorate:400","message":{"body":null,"headers":{"Accept":"*/*","Host":"127.0.0.1:3000","User-Agent":"curl/7.68.0","X-Forwarded-Port":"3000","X-Forwarded-Proto":"http"},"httpMethod":"GET","isBase64Encoded":false,"multiValueHeaders":{"Accept":["*/*"],"Host":["127.0.0.1:3000"],"User-Agent":["curl/7.68.0"],"X-Forwarded-Port":["3000"],"X-Forwarded-Proto":["http"]},"multiValueQueryStringParameters":null,"path":"/hello","pathParameters":null,"queryStringParameters":null,"requestContext":{"accountId":"123456789012","apiId":"1234567890","domainName":"127.0.0.1:3000","extendedRequestId":null,"httpMethod":"GET","identity":{"accountId":null,"apiKey":null,"caller":null,"cognitoAuthenticationProvider":null,"cognitoAuthenticationType":null,"cognitoIdentityPoolId":null,"sourceIp":"127.0.0.1","user":null,"userAgent":"Custom User Agent String","userArn":null},"path":"/hello","protocol":"HTTP/1.1","requestId":"9ee5f7f2-4dc4-4727-9630-da4c0e80965a","requestTime":"07/Nov/2022:17:14:24 +0000","requestTimeEpoch":1667841264,"resourceId":"123456","resourcePath":"/hello","stage":"Staging"},"resource":"/hello","stageVariables":null,"version":"1.0"},"timestamp":"2022-11-08 11:08:40,458+0000","service":"APP","cold_start":true,"function_name":"HelloWorldFunction","function_memory_size":"128","function_arn":"arn:aws:lambda:us-east-1:012345678912:function:HelloWorldFunction","function_request_id":"f1d7a5f7-9fc6-4847-8ad0-a02343dbad26","correlation_id":"9ee5f7f2-4dc4-4727-9630-da4c0e80965a"}
[WARNING]       2022-11-08T11:08:40.458Z        f1d7a5f7-9fc6-4847-8ad0-a02343dbad26    Subsegment ## lambda_handler discarded due to Lambda worker still initializing
[WARNING]       2022-11-08T11:08:40.458Z        f1d7a5f7-9fc6-4847-8ad0-a02343dbad26    Subsegment ## app.hello discarded due to Lambda worker still initializing
{"level":"INFO","location":"hello:22","message":"Request from unknown received","timestamp":"2022-11-08 11:08:40,458+0000","service":"APP","cold_start":true,"function_name":"HelloWorldFunction","function_memory_size":"128","function_arn":"arn:aws:lambda:us-east-1:012345678912:function:HelloWorldFunction","function_request_id":"f1d7a5f7-9fc6-4847-8ad0-a02343dbad26","correlation_id":"9ee5f7f2-4dc4-4727-9630-da4c0e80965a"}
[WARNING]       2022-11-08T11:08:40.459Z        f1d7a5f7-9fc6-4847-8ad0-a02343dbad26    No subsegment to end.
[WARNING]       2022-11-08T11:08:40.459Z        f1d7a5f7-9fc6-4847-8ad0-a02343dbad26    No subsegment to end.
END RequestId: f1d7a5f7-9fc6-4847-8ad0-a02343dbad26
REPORT RequestId: f1d7a5f7-9fc6-4847-8ad0-a02343dbad26  Init Duration: 0.15 ms  Duration: 452.44 ms     Billed Duration: 453 ms Memory Size: 128 MB     Max Memory Used: 128 MB
2022-11-08 11:08:40 127.0.0.1 - - [08/Nov/2022 11:08:40] "GET /hello HTTP/1.1" 200 -
{"message":"hello unknown person!"}
~~~
{: .language-bash}

There is a lot of output to the command line.

First the lambda handler is invoked, and the Docker image is used that is already loaded locally.

The container is mounted with the lambda app running within it, and the event is passed. This is useful for understanding the structure of events sent to Lambda functions, such as the `message` and `body`, the `httpMethod`, and more. We also have the output of the error logging. Lastly there is the returned output of the function.

Instead we can try invoking the other endpoint.


{% include links.md %}
