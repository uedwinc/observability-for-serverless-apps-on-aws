# CloudWatch Lambda Insights

CloudWatch Lambda Insights is a powerful extension for understanding the performance of Lambda functions. It provides valuable insights into a range of issues that can affect the performance of Lambda such as memory leaks, identifying high-cost functions, identifying performance impact caused by new versions of Lambda functions, and also understanding latency drivers in the Lambda functions. CloudWatch Lambda Insights allows you to capture diagnostic information before, during, and after function invocation, requiring no code changes.

1. You can enable CloudWatch Lambda Insights from the AWS console. Navigate to the Lambda function and then click on `Configuration` | `Monitoring and operations tools` | `Edit configuration` and then enable `Enhanced monitoring`.

![enable-lambda-insights](/images/enable-lambda-insights.png)

When you enable enhanced monitoring, the Lambda Insights extension will add to the Lambda function as a layer

![insights-extension](/images/insights-extension.png)

Rather than enabling each from the AWS console, let’s enable CloudWatch Lambda Insights for all the functions in the application deployed using CloudFormation.

2. Edit the CloudFormation template by adding the following YAML code, which adds a new Lambda layer with the Lambda Insights extension with the latest version in the [template-insights-enabled.yaml](../template-insights-enabled.yaml) file:

```yaml
Layers:
  - !Sub "arn:aws:lambda:${AWS::Region}:580247275435:layer:LambdaInsightsExtension:53"

# Make sure to use the latest ARN version
```

3. Update the CloudFormation stack (on the console) with the new [template file](../template-insights-enabled.yaml) to enable the Lambda extension for all three Lambda functions.

Enabling Lambda Insights for a Lambda function provides additional metrics related to execution and also provides logs about the Lambda execution. A dashboard view is available out of the box for visualizing performance monitoring for a single function or a performance monitoring view for multiple functions.

Let’s navigate to the Lambda Insights dashboard from the CloudWatch console and understand the metrics and insights available from the Lambda Insights dashboard.