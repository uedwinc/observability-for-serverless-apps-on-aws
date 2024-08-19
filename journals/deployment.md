# Deploying a basic serverless application running on AWS Lambda

Let’s deploy a basic serverless application using AWS services, namely Amazon API Gateway, AWS Lambda, and DynamoDB.

1. Create a [CloudFormation YAML file](../template.yaml) to deploy the sample application.

See the architecture for the deployment according to AWS application composer:

![architecture](/images/application-composer-template-yaml.png)

Once the application is deployed in your AWS account, it will only contain the necessary IAM roles for accessing and storing data in CloudWatch and other relevant services. It is not yet instrumented for end-to-end observability. 

2. You can deploy the sample application directly on the CloudFormation console page (Set stack name as `serverless-app`) or by clicking the following link: https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=serverless-app&templateURL=https://insiders-guide-observability-on-aws-book.s3.amazonaws.com/chapter-07/init/template.yaml

The Lambda functions and API Gateway deployed as a part of the CloudFormation are set to capture only the default out-of-the-box metrics. Further deployed Lambda functions are enabled to log to `CloudWatch log groups`.

3. You can find the API Gateway URL (APIurl) of the deployed application in the `Outputs` tab of the CloudFormation stack page. You can insert a few items into the application using Postman or HTTPie to send POST or GET requests.

- To send POST request, use `<API-Gateway-URL>items/` for the Put item operation with sample body as shown below:

![post](/images/POST.png)

- You can examine the items you have posted by navigating to DynamoDB or by using the `Get` option in the HTTPie app.

![ddb-console](/images/ddb-console.png)

- To send GET request, use `<API-Gateway-URL>items/` for the Get all items operation

![get-all](/images/GET-ALL.png)

- Use `<API-Gateway-URL>items/1` to Get item by id operation

![get-1](/images/GET-1.png)

This shows that the serverless application is successfully deployed and you are ready to use the AWS observability tools to instrument the application.

