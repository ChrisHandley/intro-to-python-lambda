---
title: Python Lambda Fundamentals
teaching: 10
exercises: 10
questions:
- "How to resolve requests?"
- "How do we do logging and tracing" 
objectives:
- "Build a simple lambda handler"
- "Build simple endpoints to perform some task"
keypoints:
- "Lambda functions are compact, and with python it all looks like the Flask framework"
---

## Lambda Function and Python

We have already looked at the application structure.

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

When we perform a deployment all the files and folders we see here are used as part of that deployment. Note the `__init__.py` files that denote that the directory can contain python files that can be accessed by the interpretor e.g. modules.

`hello_world` is the directory which our Lambda function is stored within `app.py`.

### Imports

~~~
from aws_lambda_powertools import Logger, Tracer
from aws_lambda_powertools.event_handler import APIGatewayRestResolver
from aws_lambda_powertools.logging import correlation_paths
~~~
{: .language-python}

`aws_lambda_powertools` comes with a number of functions that enable fully fledged error logging and event tracing for our app. These are `Logger` and `Tracer` and these will decorate the app, or provide the log to which we return our errors and logging messages.

Powertools also offers the `APIGatewayRestResolver`, which informs the app how it will interpret the messages it is sent. We can use a HTTP resolver in the case that access to the Lambda function via a HTTP Gateway. We also can use Application Load Balancer via `ALBResolver`, or a Lambda Function URL resolver.

### Initiating the Lambda Function


~~~
logger = Logger(service="APP")
tracer = Tracer(service="APP")
app = APIGatewayRestResolver()
~~~
{: .language-python}

We initialize the logger and tracer, with a service name, in this case "APP". This can be set in the CloudFormation template using a environment varible so that the service name is programmatically defined.

With the event handler we initiate an instance of our app.

### Lambda Handler

~~~
@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST, log_event=True)
@tracer.capture_lambda_handler
def lambda_handler(event, context):
    return app.resolve(event, context)
~~~
{: .language-python}

The logger and tracer decorate the lambda_handler, and through them we can set some settings. `log_event` set to `True` means that the logger is logging all events passed to it, and should only be used for debugging and is controlled in the CloudFormation template using a environment variable. The `correlation_id_path` is set to the ID assocatiated with the Api Gateway request using a built-in ID expression. This is used by the tracer for AWS X-ray.

The lambda handler looks like any other function for python. We define it, and it accepts two inputs - the event, and the context. We see that the handler will perform some task via `app.resolve` and return the result as an output.

The event is a JSON-formmated object and contains the data we will process. The context passes information about how the function is being invoked. Using the Api Gateway Resolver it allows us to look at the context and the event and route it to the correct endpoint - something we would have to code for ourselves otherwise.

### Event Resolution

The routing is handled by the `app.resolver` which understands if we are posting Get or Post requests.


~~~
@app.get("/hello")
@tracer.capture_method
def hello():
    tracer.put_annotation(key="User", value="unknown")
    logger.info("Request from unknown received")
    return {"message": "hello unknown person!"}
~~~
{: .language-python}

Again, our endpoints look like functions (at this point you will be sensing a distinct sense of deja vu as this all look very Flask-like - in fact we can use Flask to build lambda functions!).

The decorator `@app.get` informs the resolver that this is an endpoint, performing a Get request. The endpoint address is denoted in the brackets. The other decorator is to state that we should capture the routing for our tracing.

In this simple case it will return a json formatted response to the requesting application.


~~~
@app.get("/hello/<name>")
@tracer.capture_method
def hello_name(name):
    tracer.put_annotation(key="User", value=name)
    logger.info(f"Request from {name} received")
    return {"message": f"hello {name}, you are new here!"}
~~~
{: .language-python}

In the case of the other endpoint, more business logic is happening. Firstly the endpoint is such that is allows us to construct endpoints with variables, and split those out e.g. `hello/Chris` would place `Chris` in the variable `name` that is subsequently used in the formation of the tracer annotation, the logger message and the returned json result.



{% include links.md %}
