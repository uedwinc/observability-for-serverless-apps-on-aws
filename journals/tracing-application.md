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