# MXNet package for AWS Lambda

This is a reference application that predicts labels along with their probablities for an image using a pre-trained model with [Apache MXNet](http://mxnet.io) deployed on [AWS Lambda](https://aws.amazon.com/lambda).
It has been adapted from the original function available from AWS Labs (https://github.com/awslabs/mxnet-lambda) so that users can specify, using Lambda environment variables, the model they want to use.

A Serverless Application Model template (SAM) and instructions are provided to automate the creation of an API endpoint. You can leverage this package and its precompiled libraries to build your prediction pipeline on AWS Lambda with MXNet.

Additional models can be found in the [Model Zoo](http://data.mxnet.io/models/).

This repo shows how you can deploy Apache MXNet with AWS Lambda using Serverless Application Model (SAM) to create a serverless application with API Gateway and AWS Lambda

## Components

- MXNet 0.10.1
- Numpy
- PIL

* boto3 is included in the Lambda environment, if you want to try it locally please pip install boto3 in your virtualenv 

## Creating an API endpoint with Serverles Application Model (SAM)

* In the AWS Region you plan to deploy, make sure you have an existing Amazon S3 bucket in which SAM can create the deployment artifacts.

Else create a new bucket using the following AWS CLI command:

```
aws s3 mb s3://<your-bucket-name>
```

Place your function in your bucket:

```
cd mxnet-lambda/src
zip -9r lambda_function.zip  *
aws s3 cp lambda_function.zip <<your bucket>>
```

Before deploying the project to SAM for the first time, you'll need to update some variables in  `lambda_function.py` and `template.yaml`/`swagger.yaml` (found in `sam/` folder).

```
# swagger.yaml
# <<region>> : AWS region set in Pre-Requisites, referenced twice in swagger.yaml
# <<accountId>> : your global AWS account ID (found in MyAccount)
uri: arn:aws:apigateway:<<region>>:lambda:path/2015-03-31/functions/arn:aws:lambda:<<region>>:<<accountId>>:function:${stageVariables.LambdaFunctionName}/invocations

# template.yaml
CodeUri: s3://<bucket-name>/lambda_function.zip # name of S3 bucket created in Pre-Requiisites
DefinitionUri: s3://<bucket>/swagger.yaml # name of S3 bucket created in Pre-Requisites
```
- Execute the AWS Cloudformation SAM commands to create the Serverless application

```
aws cloudformation package \
--template-file template.yaml \
--output-template-file template-out.yaml \
--s3-bucket <your-s3-bucket-name>

aws cloudformation deploy \
--template-file <path-to-file/template-out.yaml \
--stack-name <STACK_NAME> \
--capabilities CAPABILITY_IAM
```

If something goes wrong, delete your stack:

```
aws cloudformation delete-stack --stack-name lambda-mxnet
```

Fix the problem, upload your code, and repeat the package and deploy steps.

- Get the URL of the Serverless application that was created

```
$ host=`aws cloudformation describe-stacks --stack-name mxnet-lambda-v2 | python -c 'import json,sys;obj=json.load(sys.stdin);print obj["Stacks"][0]["Outputs"][0]["OutputValue"];'`
```

- Test with GET request

```
curl $host/predict?url=https://images-na.ssl-images-amazon.com/images/G/01/img15/pet-products/small-tiles/23695_pets_vertical_store_dogs_small_tile_8._CB312176604_.jpg
```

- Test with POST request

```
curl -H "Content-Type: application/json" -X POST $host/predict -d '{"url": "https://images-na.ssl-images-amazon.com/images/G/01/img15/pet-products/small-tiles/23695_pets_vertical_store_dogs_small_tile_8._CB312176604_.jpg"}'
```

- Test the Lambda function directly:

Get the function name:

```
aws lambda list-functions --query Functions[].FunctionName
```

Then use the function name in the command below:

```
aws lambda invoke --invocation-type RequestResponse --function-name <<your function name>> --region us-east-1 --log-type Tail --payload '{"url": "https://images-na.ssl-images-amazon.com/images/G/01/img15/pet-products/small-tiles/23695_pets_vertical_store_dogs_small_tile_8._CB312176604_.jpg"}' output_file
```

## Changing the model

You can change the model by specifying environment variables. This can be done in the [Lambda console](http://docs.aws.amazon.com/lambda/latest/dg/env_variables.html). Along with the output, you can see the model that was used. 


## Notes

All the necessary libraries needed for MXNet have been copied to the src/lib folder. In addition, PIL for Python is also available for your use. OpenCV was available in the previous release, but has been taken out to reduce the size of the code package. Refer to opencv branch for the code. The instructions with the original AWS Labs Github code have additional instructions on how the package was put together.
