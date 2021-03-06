# reinvent2019-ARC314-R Lab-1

This is a sample template for reinvent-2019-ARC314-R-lab-1 - Below is a brief explanation of what we have build for you:

```bash
.
├── README.md                   <-- This instructions file
├── event.json                  <-- API Gateway Proxy Integration event payload
├── unicorn-management-service  <-- Source code for a lambda function
│   ├── __init__.py
│   ├── app.py                  <-- Lambda function code
│   └── requirements.txt        <-- Lambda dependencies
└── template.yaml               <-- SAM Template
```

## Requirements

* AWS CLI already configured with Administrator permission
* [Python 3 installed](https://www.python.org/downloads/)
* [Docker installed](https://www.docker.com/community-edition)

## Setup process

Firstly, we need a `S3 bucket` where we can upload our Lambda functions packaged as ZIP before we deploy anything - If you don't have a S3 bucket to store code artifacts then this is a good time to create one:

```bash
aws s3 mb s3://<BUCKET_NAME>
```

[AWS Lambda requires a flat folder](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html) with the application as well as its dependencies in  deployment package. When you make changes to your source code or dependency manifest,
run the following command to build your project local testing and deployment:

```bash
sam build
```

Next, run the following command to package our Lambda function to S3:

```bash
sam package \
    --output-template-file packaged.yaml \
    --s3-bucket <BUCKET_NAME>
```

Next, the following command will create a Cloudformation Stack and deploy your SAM resources.

```bash
sam deploy \
    --template-file packaged.yaml \
    --stack-name reinvent-2019-ARC314-R-lab-1 \
    --capabilities CAPABILITY_IAM
```

> **See [Serverless Application Model (SAM) HOWTO Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-quick-start.html) for more details in how to get started.**

After deployment is complete you can run the following command to retrieve the API Gateway Endpoint URL:

```bash
aws cloudformation describe-stacks \
    --stack-name reinvent-2019-ARC314-R-lab-1 \
    --query 'Stacks[].Outputs[?OutputKey==`WildRydesApi`]' \
    --output table
``` 


## Test the setup

You can test the setup, by submitting ride completion events to the API Gateway 'submit-ride-completion' resource via an HTTP POST (you can lookup the URL in your CloudFormation output section). The name of the key is WildRydesApi:

```bash
curl -XPOST -i -v -H "Content-Type:application/json" -d @event.json <WildRydesApi>

while true; do curl -XPOST -H "Content-Type:application/json" -d @event.json --silent --write-out "%{http_code}\n" --output /dev/null <WildRydesApi>; sleep 1; done
```

### Local development

**Invoking function locally using a local sample payload**

```bash
sam local invoke SubmitRideCompletionFunction --event event.json --env-vars env-vars.json 
```

**Invoking function locally through local API Gateway**

```bash
sam local start-api --env-vars env-vars.json
```

If the previous command ran successfully you should now be able to hit the following local endpoint to invoke your function `http://localhost:3000/submit-ride-completion`


## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called sam logs. sam logs lets you fetch logs generated by your Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
sam logs -n SubmitRideCompletionFunction --stack-name reinvent-2019-ARC314-R-lab-1 --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).


## Cleanup

In order to delete our Serverless Application recently deployed you can use the following AWS CLI Command:

```bash
aws cloudformation delete-stack --stack-name reinvent-2019-ARC314-R-lab-1
```



## SAM and AWS CLI commands

All commands used throughout this document

```bash
# Generate event.json via generate-event command
sam local generate-event apigateway aws-proxy > event.json

# Invoke function locally with event.json as an input
sam local invoke SubmitRideCompletionFunction --event event.json

# Run API Gateway locally
sam local start-api

# Create S3 bucket
aws s3 mb s3://<BUCKET_NAME>

# Package Lambda function defined locally and upload to S3 as an artifact
sam package \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME

# Deploy SAM template as a CloudFormation stack
sam deploy \
    --template-file packaged.yaml \
    --stack-name reinvent-2019-ARC314-R-lab-1 \
    --capabilities CAPABILITY_IAM

# Describe Output section of CloudFormation stack previously created
aws cloudformation describe-stacks \
    --stack-name reinvent-2019-ARC314-R-lab-1 \
    --query 'Stacks[].Outputs[?OutputKey==`HelloWorldApi`]' \
    --output table

# Tail Lambda function Logs using Logical name defined in SAM Template
sam logs -n SubmitRideCompletionFunction --stack-name reinvent-2019-ARC314-R-lab-1 --tail
```