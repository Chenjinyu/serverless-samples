## Flask server packaged for AWS Lambda

This example app builds a simple flask application packaged to be run at AWS Lambda. Notice the [`/app/run.sh`](./app/run.sh) shell script which starts gunicorn as a daemon and then calls the [AWS Lambda Runtime Interface Client](https://github.com/aws/aws-lambda-python-runtime-interface-client) (RIC). The RIC enables the handler function to interact with the Lambda Runtime API during execution.

![Application Flow](./appflow.png)

Follow the instructions below to build and run the application.

1. Build your container locally:
```
    docker build -t flask-lambda-container:latest .
```

2. Tag your container for deployment to ECR:
```
    docker tag flask-lambda-container:latest \
    674415394971.dkr.ecr.us-east-2.amazonaws.com/flask-lambda-container:latest
```

3. Login to ECR
```
    aws ecr get-login-password --region us-east-2 \
    | docker login --username AWS \
    --password-stdin 674415394971.dkr.ecr.us-east-2.amazonaws.com
```

4. Create a repository at ECR

```
    aws ecr create-repository \
    --repository-name flask-lambda-container \
    --image-scanning-configuration scanOnPush=true \
    --region us-east-2
```

5. Push the container image to ECR
```
docker push 674415394971.dkr.ecr.us-east-2.amazonaws.com/flask-lambda-container:latest
```

6. Create the Lambda function
   (note: you'll have to [create an execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html) first)

>The Role([lambda-serverless-example-exec-role](https://us-east-1.console.aws.amazon.com/iamv2/home?region=us-east-2#/roles/details/lambda-serverless-example-exec-role?section=permissions))has been created. 
```
    aws lambda --region us-east-2 create-function \
    --function-name flask-lambda-container \
    --package-type Image \
    --role arn:aws:iam::674415394971:role/lambda-serverless-example-exec-role \
    --code ImageUri=674415394971.dkr.ecr.us-east-2.amazonaws.com/flask-lambda-container:latest
```

7. Invoke the Lambda function
```
    aws lambda --region us-east-2 invoke \
    --function-name flask-lambda-container \
    --invocation-type Event \
    --payload '{ "foo": "bar" }' \
    outfile.txt
```

[Move on to the next sample application](../3-flask-dual-deployment/README.md)...
