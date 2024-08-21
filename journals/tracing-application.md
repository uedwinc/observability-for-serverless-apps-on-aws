# End-to-end tracing of the Node.js application

Let’s enable `X-Ray active tracing`. When enabled, active tracing captures detailed information about the flow of a request as it travels through the different components of your application, such as AWS Lambda functions, API Gateway, and DynamoDB.

1. Let’s enable active tracing for both the API Gateway and the Lambda function using the [updated CloudFormation template](../template-tracing-enabled.yaml)

- To enable active tracing for the Lambda function(s), I have added a `Mode:Active` line to the function:

```yaml
<Function-Name>:
  Runtime: nodejs16.x
  Timeout: 100
  Layers:
    - !Sub "arn:aws:lambda:${AWS::Region}:580247275435:layer:LambdaInsightsExtension:53"
  TracingConfig:
    Mode: Active
```

- To enable active tracing for API Gateway, I have added a `TracingEnabled: true` line in the CloudFormation template:

```yaml
Api:
  TracingEnabled: true # <----- ADD FOR API Tracing
```

2. To implement the changes, update your CloudFormation stack using the [updated template](../template-tracing-enabled.yaml)

3. After successfully deploying the updated template, you can confirm that tracing is enabled for your Lambda function(s) by navigating to the Lambda function’s configuration page. Navigate to the Lambda function, click on the `Configuration` tab, and then verify that `Active tracing` is `Enabled`

![lambda-enabled](/images/lambda-enabled.png)

4. To verify tracing is enabled for the API Gateway `Prod` stage, navigate to `API Gateway` | `Stages` | `Prod` and verify that the `X-Ray Tracing` option is marked.

![api-gate-enabled](/images/api-gate-enabled.png)

To test the end-to-end tracing of the application, you can use HTTPie or Postman to invoke the `GetAllItems` function or insert items using the `Put` function. You should be able to see the X-Ray traces showcasing the end-to-end view of the user transaction, as shown below from the CloudWatch service map.

5. To view the X-Ray trace in the service map, navigate to `CloudWatch` | `X-Ray traces` | `Trace map`. Here, you can see the flow of the requests between `Client`, `API Gateway`, and `Lambda`, including additional properties, such as each node’s latency, requests/sec, and 5xx errors, without the need for any code-level instrumentation.

![trace-map](/images/trace-map.png)

6. Navigate to the `Trace` section and click on a particular trace. By going to the `Trace details` section and viewing `Segments Timeline`, you can gain insight into the amount of time spent at each stage and identify whether the major delay is at the invocation of the Lambda function.

![trace-details](/images/trace-details.png)
![segments-timeline](/images/segments-timeline.png)

> The `Trace` view provides a high-level overview of the user journey, but it may not give enough detail on where the user is encountering an issue. To get this information, we can use segments in AWS X-Ray and trace database calls with X-Ray. 
> Leveraging Lambda Powertools can simplify the implementation process and add these details without sacrificing the maintainability or readability of the application code.

## Exploring Lambda Powertools

Lambda Powertools is a suite of utilities and libraries that will help in adopting best practices for tracing, structured logging, and so on in Lambda functions that are built around the AWS SDKs. There are three main core utilities, namely `Logger`, `Metrics`, and `Tracer` to support the three pillars of observability.

Lambda Powertools offers three different ways to instrument your code, offering flexibility and convenience to suit your specific needs: `middy` (middleware approach), the `method decorator` approach and the `manual` approach. We will use the manual approach to generate logs, metrics, and traces for the Lambda function.

Now, let’s extend the observability for the Node.js application using Lambda Powertools.

First, let’s look into current logging in the CloudWatch logs. As we examine the logs, we’ll notice they are lacking in detail and are not structured in any particular format. This presents a challenge when trying to gain insight into the inner workings of the Lambda function, especially when retrieving items from DynamoDB.

![lambda-logs-first](/images/lambda-logs-first.png)

### Deploying a new sample application

1. Create an [updated CloudFormation template](../template-powertools.yaml) with all the necessary changes.

2. Use this CloudFormation template and create a new application named `serverless-app2`. As the application is in Node.js, I have imported the npm libraries of Lambda Powertools and included them in the CloudFormation deployment

> I have included Lambda Powertools only in the `get-all-items.js` Lambda function in this exercise. So, please execute the Postman/HTTPie configuration for inserting the records and retrieving the details. 

> We will examine each modification made to the `GetAllItems` Lambda function step by step in order to expand observability for metrics, logs, and traces. We will see how it enhances our ability to troubleshoot applications and add business context to our observability, resulting in an improved overall experience.

### Lambda Powertools for enhanced logging

The `GetAllItems` Lambda function has been added with Lambda Powertools to capture information using the structure JSON format and retrieve additional information about the context of the Lambda function, such as cold starts, runtime, and so on. 

Let’s look at the additional code added for structure in the Lambda function:

1. The code first imports the `Logger` and `injectLambdaContext` modules from the `@aws-lambda-powertools/logger` library:

```js
//Logging using Lambda powertools with Lambda context support.

//Inclusion of Logger PowerLambda Tools
const { Logger, injectLambdaContext } = require('@aws-lambda-powertools/logger');
```

2. A new `Logger` instance is created where the `serviceName` property is set to `get-all-items`. This is used to identify the source of the logs in the Amazon CloudWatch logs:

```js
//Servicename of the lambda function shown in the CloudWatch Logs
const logger = new Logger({serviceName: 'get-all-items'});
```

3. The `injectLambdaContext` function is then used to log the Lambda context, which contains information about the function’s execution environment and its interaction with the AWS infrastructure. You can see that the added Lambda context provides information about cold starts, function information, and service name. We have also logged output received from `get-all-items` into the CloudWatch log:

```js
//Logging Lambda Context and the output of items as a JSON using logger standard format
  logger.addContext(context);

//Logging Items retrieved as a JSON in CloudWatch Logs.
  logger.info('Items in list:', { items });
```

4. Enhanced Lambda logging with Lambda Powertools showing the full Lambda context along with the number of items retrieved and the details in the CloudWatch logs

![lambda-with-powertools](/images/lambda-with-powertools.png)

### Lambda Powertools for custom metrics

We have further enhanced our application by retrieving and adding custom business metrics using the Lambda Powertools metrics functionality. The Lambda function retrieved the item count metric namely Count of retrieved items as a custom metric and added it into CloudWatch metics using `Embedded Metric Format (EMF)`. 

Let’s understand the changes made in the `get-all-items.js` file:

1. The code first imports the `Metrics`, `MetricUnits`, and `logMetrics` modules from the `@aws-lambda-powertools/metrics` library:

```js
//Inclusion of Metrics from Lambda PowerTools
const { Metrics, MetricUnits, logMetrics } = require('@aws-lambda-powertools/metrics');
```

2. Next, a `new Metrics` instance is created, where the `namespace` property is set to `getitems` and the `serviceName` property is set to `get-all-items`. These properties identify the source of the metrics in the Amazon CloudWatch custom metrics:

```js
//Custom Metric namespace and service name in CloudWatch Custom Metrics
const metrics = new Metrics({ namespace: 'getitems', serviceName: 'get-all-items' });
```

3. The code then adds a custom metric called '`itemcount`' to the Metrics instance, using the `metrics.addMetric` method. The metric is set to the count of items in the items array and has a unit of `MetricUnits.Count`. Finally, the code calls the `metrics.publishStoredMetrics` method to publish the custom metrics to CloudWatch:

```js
// Adding the total retrieved items as a metric in the Custom namespace called "getitems"
  metrics.addMetric('itemcount', MetricUnits.Count, items.Count);
  metrics.publishStoredMetrics();
```

4. You can verify from the CloudWatch logs that the metric count for the total items retrieved is published to the CloudWatch metric custom namespace called `getitems`:

![custom-metric-data](/images/custom-metric-data.png)

5. Navigate to `All metrics` | `Custom namespaces` | `getitems`. The `getitems` namespace is published with the details of the `count of items` retrieved from DynamoDB from the structure JSON output from the Lambda log.

6. If you graph the metric over the period, you can understand how the number of items increased in DynamoDB over some time:

![items-in-dynamodb](/images/items-in-dynamodb.png)

