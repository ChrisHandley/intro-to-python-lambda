---
title: Local Testing
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
While we can deploy our app directly into our testing stage and account on AWS, we can also test our lambda functions locally.

To do this we will need `Docker` installed, and following all the relevant steps for you system (for Windows and WSL, Docker can be installed on the Windows side and communicate to the Linux Subsystem - very handy!)

## Build and Deploy Locally
In the head directory of our Lambda function application, we build the application.

~~~
sam build
~~~
{: .language-bash}

Further options to `sam build` include building the Lambda function application in a Docker like container.

With the application built, we can then deploy it. In this case we are deploying it first locally.

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

We first need to rebuild the app using a modified template.

In the `template.yaml` we add a new event type to our Lambda function

~~~
            Events:
                HelloWorld:
                    Type: Api
                    Properties:
                        
                        Path: /hello
                        Method: get
                  
                
                HelloWorldName:
                    Type: Api
                    Properties:
                        Path: /hello/{name}


                ByeWorldTest:
                    Type: Api
                    Properties:
                        Path: /goodbye
                        Method: get

~~~
{: .language}

We will also add a new endpoint to the lambda function

~~~
@app.get("/goodbye")
@tracer.capture_method
def goodbye():
    tracer.put_annotation(key="User", value="unknown")
    logger.info("Request from unknown received")
    return {"message": "goodbye unknown person!"}
~~~


Note that the new event, `HelloWorldName` and `ByeWorldTest`. The latter is of the same form as the original `HelloWorld` and the event is processed and sent to the endpoint as expected.

`HelloWorldName` differs because it defines a path with a variable in the path `{name}`, and this variable is used in the endpoint function.

~~~
curl http://127.0.0.1:3000/hello/Chris

...

{"message":"hello Chris, you are new here!"}
~~~
{: .language-bash}

### Direct Invoke

It is likely in the template you define a number of different Lambda functions, but when testing you simply want to test a particular Lambda function. This can be done using the following;

~~~
sam local invoke "HelloWorldFunction" -e events/event.json
~~~
{: .language-bash}

Notice that we invoke the function by its logical name given in the template, and that we are passing to it an event.

The event looks as follows;

~~~
{
  "body": "{\"message\": \"hello world\"}",
  "resource": "/hello",
  "path": "/hello",
  "httpMethod": "GET",
  "isBase64Encoded": false,
  "queryStringParameters": {
    "foo": "bar"
  },
  "pathParameters": {
    "proxy": "/path/to/resource"
  },
  "stageVariables": {
    "baz": "qux"
  },
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate, sdch",
    "Accept-Language": "en-US,en;q=0.8",
    "Cache-Control": "max-age=0",
    "CloudFront-Forwarded-Proto": "https",
    "CloudFront-Is-Desktop-Viewer": "true",
    "CloudFront-Is-Mobile-Viewer": "false",
    "CloudFront-Is-SmartTV-Viewer": "false",
    "CloudFront-Is-Tablet-Viewer": "false",
    "CloudFront-Viewer-Country": "US",
    "Host": "1234567890.execute-api.us-east-1.amazonaws.com",
    "Upgrade-Insecure-Requests": "1",
    "User-Agent": "Custom User Agent String",
    "Via": "1.1 08f323deadbeefa7af34d5feb414ce27.cloudfront.net (CloudFront)",
    "X-Amz-Cf-Id": "cDehVQoZnx43VYQb9j2-nvCh-9z396Uhbp027Y2JvkCPNLmGJHqlaA==",
    "X-Forwarded-For": "127.0.0.1, 127.0.0.2",
    "X-Forwarded-Port": "443",
    "X-Forwarded-Proto": "https"
  },
  "requestContext": {
    "accountId": "123456789012",
    "resourceId": "123456",
    "stage": "prod",
    "requestId": "c6af9ac6-7b61-11e6-9a41-93e8deadbeef",
    "requestTime": "09/Apr/2015:12:34:56 +0000",
    "requestTimeEpoch": 1428582896000,
    "identity": {
      "cognitoIdentityPoolId": null,
      "accountId": null,
      "cognitoIdentityId": null,
      "caller": null,
      "accessKey": null,
      "sourceIp": "127.0.0.1",
      "cognitoAuthenticationType": null,
      "cognitoAuthenticationProvider": null,
      "userArn": null,
      "userAgent": "Custom User Agent String",
      "user": null
    },
    "path": "/prod/hello",
    "resourcePath": "/hello",
    "httpMethod": "POST",
    "apiId": "1234567890",
    "protocol": "HTTP/1.1"
  }
}
~~~
{: .language-json}

The event has multiple keys which we will explain. This is specifically for an API Gateway Rest API event. Events from different services have different formats.

`body` contains the information that is passed in the event and is likely where all data is that the Lambda will process e.g. parameters, strings, numbers, locations of resources where the data is held. The AWS Powertools

&&&&

## Sending Information

Our values we pass are in the event.

Update the event handler

~~~
@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST, log_event=True)
@tracer.capture_lambda_handler
def lambda_handler(event, context):
    print(event)
    return app.resolve(event, context)
~~~
{: .language-python}

Then when we build and invoke the function, we can pass some path parameters.

~~~
sam build
sam local start-api
curl http://127.0.0.1:3000/hello/Chris?surname=handley
~~~
{: .lanugage-bash}

The event returned is as follows

~~~
{'body': None, 'headers': {'Accept': '*/*', 'Host': '127.0.0.1:3000', 'User-Agent': 'curl/7.68.0', 'X-Forwarded-Port': '3000', 'X-Forwarded-Proto': 'http'}, 'httpMethod': 'GET', 'isBase64Encoded': False, 'multiValueHeaders': {'Accept': ['*/*'], 'Host': ['127.0.0.1:3000'], 'User-Agent': ['curl/7.68.0'], 'X-Forwarded-Port': ['3000'], 'X-Forwarded-Proto': ['http']}, 'multiValueQueryStringParameters': {'surname': ['Handley']}, 'path': '/hello/Chris', 'pathParameters': {'name': 'Chris'}, 'queryStringParameters': {'surname': 'Handley'}, 'requestContext': {'accountId': '123456789012', 'apiId': '1234567890', 'domainName': '127.0.0.1:3000', 'extendedRequestId': None, 'httpMethod': 'GET', 'identity': {'accountId': None, 'apiKey': None, 'caller': None, 'cognitoAuthenticationProvider': None, 'cognitoAuthenticationType': None, 'cognitoIdentityPoolId': None, 'sourceIp': '127.0.0.1', 'user': None, 'userAgent': 'Custom User Agent String', 'userArn': None}, 'path': '/hello/{name}', 'protocol': 'HTTP/1.1', 'requestId': 'b5e0949b-743b-467c-99b2-ada4ee067c74', 'requestTime': '08/Nov/2022:13:27:15 +0000', 'requestTimeEpoch': 1667914035, 'resourceId': '123456', 'resourcePath': '/hello/{name}', 'stage': 'Staging'}, 'resource': '/hello/{name}', 'stageVariables': None, 'version': '1.0'}
~~~
{: .language-json}

We can see the `pathParameter` is `name` and is used to form the path. The surname appears as the `queryStringParameter`.

Let's extract that in our `hello_name` function.

~~~
@app.get("/hello/<name>")
@tracer.capture_method
def hello_name(name):
    tracer.put_annotation(key="User", value=name)
    logger.info(f"Request from {name} received")
    surname = app.current_event.query_string_parameters.get("surname")
    return {"message": f"hello {name} {surname}, you are new here!"}
~~~
{: .language-python}

Because this is a Get request, the surname we are passing is extracted using the `app.current_event.query_string_parameters.get("surname")` call.

If we are posting information, the process would be the same for query string parameters in the url, but given it is a Post request, we can send a JSON to form the body.

We first make a new event to pass, `event2.json`, in the events directory.

~~~
{
  "body": "{\"surname\": \"handley\"}",
  "resource": "/goodbye",
  "path": "/goodbye",
  "httpMethod": "POST",
  "isBase64Encoded": false,
  "queryStringParameters": {
    "foo": "bar"
  },
  "pathParameters": {
    "proxy": "/path/to/resource"
  },
  "stageVariables": {
    "baz": "qux"
  },
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate, sdch",
    "Accept-Language": "en-US,en;q=0.8",
    "Cache-Control": "max-age=0",
    "CloudFront-Forwarded-Proto": "https",
    "CloudFront-Is-Desktop-Viewer": "true",
    "CloudFront-Is-Mobile-Viewer": "false",
    "CloudFront-Is-SmartTV-Viewer": "false",
    "CloudFront-Is-Tablet-Viewer": "false",
    "CloudFront-Viewer-Country": "US",
    "Host": "1234567890.execute-api.us-east-1.amazonaws.com",
    "Upgrade-Insecure-Requests": "1",
    "User-Agent": "Custom User Agent String",
    "Via": "1.1 08f323deadbeefa7af34d5feb414ce27.cloudfront.net (CloudFront)",
    "X-Amz-Cf-Id": "cDehVQoZnx43VYQb9j2-nvCh-9z396Uhbp027Y2JvkCPNLmGJHqlaA==",
    "X-Forwarded-For": "127.0.0.1, 127.0.0.2",
    "X-Forwarded-Port": "443",
    "X-Forwarded-Proto": "https"
  },
  "requestContext": {
    "accountId": "123456789012",
    "resourceId": "123456",
    "stage": "prod",
    "requestId": "c6af9ac6-7b61-11e6-9a41-93e8deadbeef",
    "requestTime": "09/Apr/2015:12:34:56 +0000",
    "requestTimeEpoch": 1428582896000,
    "identity": {
      "cognitoIdentityPoolId": null,
      "accountId": null,
      "cognitoIdentityId": null,
      "caller": null,
      "accessKey": null,
      "sourceIp": "127.0.0.1",
      "cognitoAuthenticationType": null,
      "cognitoAuthenticationProvider": null,
      "userArn": null,
      "userAgent": "Custom User Agent String",
      "user": null
    },
    "path": "/prod/goodbye",
    "resourcePath": "/goodbye",
    "httpMethod": "POST",
    "apiId": "1234567890",
    "protocol": "HTTP/1.1"
  }
}
~~~
{: .language-json}

Then we update the `goodbye` endpoint.

~~~
@app.post("/goodbye")
@tracer.capture_method
def goodbye():
    tracer.put_annotation(key="User", value="unknown")
    logger.info("Request from unknown received")
    surname = app.current_event.json_body.get("surname")
    return {"message": f"goodbye {surname}"}
~~~
{: .language-python}

`app.current_event.json_body.get("surname")` is how we extract the values from the body of the json event that is sent.


{% include links.md %}
